---
title: 在 Windows 上安装 TensorFlow GPU 避免踩坑 
author: ram
date: 2024-09-03 1:51:00 +0800
categories: [开发, 教程]
tags: [TensorFlow,Docker,Conda,WsL,CUDA]
---

# 在 Windows 上安装 TensorFlow GPU 避免踩坑 

## 1.前提

### 1.1 本文只考虑 Windows 系统上安装 TensorFlow GPU 的问题。CPU 版本的 TensorFlow 可以直接通过 pip 安装，参考官方文档：
https://www.tensorflow.org/install?hl=en
但 GPU 版本需要配置 CUDA 和 cuDNN 环境或者使用 Docker 容器。

### 1.2 为了获取最新的安装信息，建议查看英文教程，方法是将网址后缀 `zh-cn` 改为 `en`，例如：
https://www.tensorflow.org/install/source_windows?hl=zh-cn
改为
https://www.tensorflow.org/install/source_windows?hl=en

## 2.问题背景

在安装 TensorFlow GPU 时，发现中文版的官方安装文档较久没有更新，尤其是关于 Windows 系统上的 TensorFlow GPU 版本支持问题。中文版文档的版本列表仅更新到 2.7.0，而实际情况是 2.10.0 是最后一个支持 Windows 的版本。英文版文档有明确提示：**`tensorflow-gpu-2.10.0` 是最后一个支持 Windows 的版本**。更高版本的 TensorFlow 在 Windows 原生上已经不支持 GPU 加速。


GPU support on native-Windows is only available for 2.10 or earlier versions. Starting in TF 2.11, CUDA build is not supported for Windows. For using TensorFlow GPU on Windows, you will need to build/install TensorFlow in WSL2 or use tensorflow-cpu with TensorFlow-DirectML-Plugin.




## 3.安装方法

### 3.1 （不建议）使用 Conda 安装 TensorFlow GPU 2.10.0

参考Conda官方文档
https://docs.anaconda.com/working-with-conda/applications/tensorflow/

```bash
conda create -n tf-gpu tensorflow-gpu
conda activate tf-gpu
```

但需要注意 Conda 安装存在版本兼容问题。
这个错误消息是由于使用了过时的 NumPy API 引起的。具体来说，np.object 这一属性在 NumPy 1.20 版本中已经被弃用，并且在后续版本中被完全移除。现在应该使用内置的 object 关键字来代替 np.object。
简单地降级 TensorFlow 并不能总是解决问题，因为部分库依赖高版本的NumPy。

AttributeError: module 'numpy' has no attribute 'object'.


### 3.2 手动安装 TensorFlow GPU

参考 TensorFlow 官方文档
https://www.tensorflow.org/install?hl=en
https://www.tensorflow.org/install/pip?hl=en

手动安装可以避免 Conda 的一些兼容性问题，但需要逐步配置 CUDA 和 cuDNN 环境


#### 3.2.1 下载并安装支持的 CUDA 版本（如 CUDA 11.2）和相应的 cuDNN。
#### 3.2.2 设置环境变量，例如 `CUDA_HOME` 和 `PATH`，确保这些工具可用。
3. 使用 pip 安装 TensorFlow 2.10.0：

    ```bash
    pip install tensorflow-gpu==2.10.0
    ```

4. 验证安装是否成功：

    ```python
    import tensorflow as tf
    print(tf.__version__)
    ```

### 3.3 （推荐） 使用 Docker 安装 TensorFlow GPU

参考 TensorFlow 官方文档
https://www.tensorflow.org/install/docker?hl=en
与docker仓库
https://hub.docker.com/r/tensorflow/tensorflow

使用 Docker 可以避免在 Windows 上直接配置环境的兼容性问题：

##### 3.1 安装 Docker Desktop 并启动它。
   
对于 Windows 11 用户，可以通过 Windows Subsystem for Linux (WSL2) 安装最新版本的 TensorFlow GPU，这种方法可以利用最新的 TensorFlow 版本，并且避免 Windows 原生 GPU 支持问题。
#### 3.2 拉取官方的 TensorFlow GPU Docker 镜像：

    ```bash
    docker pull tensorflow/tensorflow:latest-gpu
    ```

#### 3.3 创建运行Docker容器（省略）

## 总结

- **查看英文版文档**：获取最新的 TensorFlow 安装信息，确保使用最新的教程和版本信息，请查看英文版文档。
- **手动安装与 Docker 是有效的替代方法**：手动配置可以定制化安装环境，而 Docker 可以快速、简单地部署 GPU 支持的 TensorFlow。