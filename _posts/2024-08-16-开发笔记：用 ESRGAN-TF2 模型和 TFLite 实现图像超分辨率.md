---
title: 用 ESRGAN-TF2 模型和 TFLite 实现图像超分辨率
author: ram
date: 2024-08-16 10:00:00 +0800
categories: [开发, 教程]
tags: [图像超分辨率]
---

# 开发笔记：用 ESRGAN-TF2 模型和 TFLite 实现图像超分辨率

在使用 [ESRGAN-TF2 模型](https://www.kaggle.com/models/kaggle/esrgan-tf2/tfLite) 和 `tflite_flutter` 库进行安卓移动端开发时，我遇到了模型输入图片尺寸固定为 `50x50` 的问题。
当我输入非"50\*50"尺寸的图片，都不能正确超分。由于我查找的文章都是关于 esrgan 本身完整模型支持动态尺寸输入，我就尝试修改代码试图使得其在选择图片后获取其尺寸，尝试使其能够动态输入，后面发现不行，后面发现 tflite 为了性能考虑，将 tf 模型转化为 lite 之后，输入尺寸就固定了，无法直接支持动态尺寸的输入。为了让模型能够处理不同大小的图片，我尝试了两个方案：

1. **更换支持动态输入尺寸的模型**：最终发现不可行。
2. **调整图像尺寸并进行分割**：对图像进行补全和分割，使其适应固定尺寸，然后再进行模型推理和拼合。

以下是最终实现的关键代码。

## 加载模型

首先是加载模型的代码，根据设备平台使用不同的硬件加速：

```dart
Future<void> loadModel() async {
  final options = InterpreterOptions();

  // 如果是 Android 设备，添加 XNNPack 和 GPU 委托来加速
  if (Platform.isAndroid) {
    options.addDelegate(XNNPackDelegate());
    options.addDelegate(GpuDelegateV2());
  }
  // 如果是 iOS 设备，使用 Metal 委托加速
  else if (Platform.isIOS) {
    options.addDelegate(GpuDelegate());
  }

  // 从 assets 加载 TFLite 模型
  interpreter = await Interpreter.fromAsset(modelPath, options: options);

  // 获取输入和输出张量的形状
  inputTensor = interpreter.getInputTensors().first;
  outputTensor = interpreter.getOutputTensors().first;

  log('模型加载成功');
}
```

## 处理选择的图片

当用户选择图片时，我们需要处理图片的尺寸，确保它符合模型的输入要求：

```dart
Future<void> processImage() async {
  if (imagePath != null) {
    // 读取图片文件的字节数据
    final imageData = await rootBundle.load(imagePath!);
    // 使用 image 库解码图片
    final originImage = img.decodeImage(imageData.buffer.asUint8List())!;

    // 默认使用原图，但如果需要，会对图片进行补全
    img.Image paddedImage = originImage;
    int inputSize = 50; // 模型要求的输入尺寸

    // 如果图片尺寸不是 50 的倍数，我们就要进行补全
    if (originImage.width % inputSize != 0 || originImage.height % inputSize != 0) {
      paddedImage = padImage(originImage);
    }

    // 把补全后的图片分割成多个 50x50 的小块
    List<img.Image> partImageGroup = splitImage(paddedImage);

    // 对每个小块进行模型推理
    List<img.Image> partImageProcessedGroup = [];
    for (var imageMatrix in partImageGroup) {
      img.Image partImageProcessed = await runInference(imageMatrix);
      partImageProcessedGroup.add(partImageProcessed);
    }

    // 把所有处理过的小块拼合成最终的图像
    img.Image combinedImage = combineImages(partImageProcessedGroup, paddedImage.width * 4, paddedImage.height * 4);

    // 弹窗展示最终的合成图像
    showDialog(
      context: context,
      builder: (context) {
        return AlertDialog(
          title: Text('合成后的图像'),
          content: SingleChildScrollView(
            scrollDirection: Axis.horizontal,
            child: Image.memory(img.encodeJpg(combinedImage)),
          ),
          actions: [
            TextButton(
              onPressed: () {
                Navigator.pop(context);
              },
              child: Text('关闭'),
            ),
          ],
        );
      },
    );
  }
}
```

## 补全图片

为了确保图片的尺寸可以被正确分割，我们对图片进行补全，使其尺寸变为 50 的倍数：

```dart
img.Image padImage(img.Image originImage) {
  // 计算需要补全的宽度和高度，使其变为 50 的倍数
  int paddedWidth = (originImage.width + 49) ~/ 50 * 50;
  int paddedHeight = (originImage.height + 49) ~/ 50 * 50;

  // 创建一个用白色填充的空白图像
  img.Image paddedImage = img.Image(width: paddedWidth, height: paddedHeight);
  img.fill(paddedImage, color: img.ColorRgb8(255, 255, 255));

  // 把原始图像合成到空白图像上
  img.compositeImage(
    paddedImage,
    originImage,
    dstX: 0,
    dstY: 0,
    dstW: originImage.width,
    dstH: originImage.height,
  );

  return paddedImage; // 返回补全后的图像
}
```

## 分割图片

将补全后的图像分割为固定尺寸的图块，方便进行模型推理：

```dart
List<img.Image> splitImage(img.Image paddedImage) {
  List<img.Image> blockImageGroup = [];

  // 逐行逐列地分割图像
  for (int i = 0; i < paddedImage.width; i += 50) {
    for (int j = 0; j < paddedImage.height; j += 50) {
      img.Image blockImage = img.copyCrop(paddedImage, x: i, y: j, width: 50, height: 50);
      blockImageGroup.add(blockImage);
    }
  }
  return blockImageGroup; // 返回分割后的图像列表
}
```

## 拼合图像

在模型推理完成后，我们需要将处理后的图像块拼合回原来的大图。下面是具体的代码和说明：

```dart
img.Image combineImages(List<img.Image> partImageProcessedGroup, int combinedWidth, int combinedHeight) {
  // 创建一个空白图像用于拼合，大小为补全后图像的 4 倍
  img.Image combinedImage = img.Image(width: combinedWidth, height: combinedHeight);
  img.fill(combinedImage, color: img.ColorRgb8(0, 255, 255)); // 使用浅绿色填充背景

  // 将处理后的每个图像块拼合到最终的大图上
  for (int i = 0; i < combinedImage.width; i += 200) {
    for (int j = 0; j < combinedImage.height; j += 200) {
      // 计算要拼合的图块在处理过的图像组中的索引
      int index = (i ~/ 200) * (combinedImage.height ~/ 200) + j ~/ 200;
      img.compositeImage(
        combinedImage,
        partImageProcessedGroup[index],
        dstX: i,
        dstY: j,
        dstW: 200, // 每个图像块的宽度
        dstH: 200, // 每个图像块的高度
        srcX: 0,
        srcY: 0,
        srcW: 200,
        srcH: 200,
      );
    }
  }

  return combinedImage; // 返回拼合后的图像
}
```

这个拼合步骤确保了最终的输出图像恢复到原始的完整图像，并经过超分辨率处理。

通过上述方法，我成功让模型处理不同尺寸的图像，并在移动设备上实现了高效的图像超分辨率处理。这种方法确保了输入图像能够被正确分割并适应模型的固定输入尺寸，最终生成高分辨率的图像。
