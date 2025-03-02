---
title: 解决 Flutter 项目运行中的 Java 版本问题
author: ram
date: 2025-03-03 00:05:00 +0800
categories: [开发, 教程]
tags: [Java, JDK, 环境配置]
---
# 解决 Flutter 项目运行中的 Java 版本问题

在Flutter项目开发中，遇到 `Unsupported class file major version 65` 错误，通常是由于 **Java版本与Gradle版本不兼容** 所致。此错误提示当前使用的Java版本过高，而项目所用的Gradle版本不支持该版本。

### 解决方案

**1. 降级Java版本**

Gradle对Java版本有特定的兼容性要求。目前，Gradle尚未完全支持Java 21（对应的class file major version 65）。因此，建议将Java版本降级至Java 17或Java 11。

**步骤：**

- **下载并安装Java 17：**

  - **Windows：** 从[Adoptium](https://adoptium.net/)下载适用于Windows的Java 17安装包并安装。

  - **macOS/Linux：** 可以使用包管理工具安装，例如在macOS上使用Homebrew：

    ```bash
    brew install openjdk@17
    ```

- **设置`JAVA_HOME`环境变量：**

  - **Windows：**

    1. 打开“系统属性”，进入“环境变量”设置。

    2. 在“系统变量”中，点击“新建”，名称为`JAVA_HOME`，值为Java 17的安装路径，例如：`C:\Program Files\Java\jdk-17`.

    3. 编辑`Path`变量，添加`%JAVA_HOME%\bin`。

  - **macOS/Linux：**

    在终端中添加以下内容到`~/.bash_profile`或`~/.zshrc`：

    ```bash
    export JAVA_HOME=$(/usr/libexec/java_home -v 17)
    export PATH=$JAVA_HOME/bin:$PATH
    ```

    然后，运行`source ~/.bash_profile`或`source ~/.zshrc`使配置生效。

**2. 使用`flutter config`指定JDK路径**

Flutter提供了`flutter config --jdk-dir`命令来指定所使用的JDK路径，以避免使用Android Studio内置的Java版本。

**步骤：**

1. 确定Java 17的安装路径。例如：

   - **Windows：** `C:\Program Files\Java\jdk-17`

   - **macOS/Linux：** `/Library/Java/JavaVirtualMachines/jdk-17.jdk/Contents/Home`

2. 在终端或命令提示符中运行：

   ```bash
   flutter config --jdk-dir [JDK 17 路径]
   ```

   将`[JDK 17 路径]`替换为实际的JDK安装路径。

3. 验证配置是否生效：

   ```bash
   flutter doctor -v
   ```

   检查输出中的“Java binary at”是否指向所指定的Java 17路径。

**3. 升级Gradle版本**

如果希望继续使用较新的Java版本，需要确保Gradle版本与之兼容。

**步骤：**

1. **修改`gradle-wrapper.properties`文件：**

   找到`android/gradle/wrapper/gradle-wrapper.properties`文件，修改`distributionUrl`为兼容Java版本的Gradle版本。例如：

   ```properties
   distributionUrl=https\://services.gradle.org/distributions/gradle-8.0-bin.zip
   ```

   请根据实际需要选择合适的Gradle版本。

2. **修改`build.gradle`文件：**

   在`android/build.gradle`中，确保使用与Gradle版本匹配的Android Gradle插件版本。例如：

   ```groovy
   dependencies {
       classpath 'com.android.tools.build:gradle:8.0.0'
   }
   ```

3. **清理并重建项目：**

   ```bash
   flutter clean
   flutter pub get
   flutter build apk
   ```

**注意：** 升级Gradle和Android Gradle插件版本可能会引入其他兼容性问题，请在升级前备份项目并仔细测试。

### 总结

遇到`Unsupported class file major version 65`错误时，首先考虑降级Java版本至Gradle支持的范围（如Java 17）。如果需要使用更高版本的Java，则需相应地升级Gradle和Android Gradle插件版本。使用`flutter config --jdk-dir`命令可以明确指定Flutter使用的JDK路径，避免因环境变量设置不当导致的问题。
