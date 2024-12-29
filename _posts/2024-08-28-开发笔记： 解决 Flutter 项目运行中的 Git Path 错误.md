---
title: 解决 Flutter 项目运行中的 Git Path 错误
author: ram
date: 2024-08-28 19：28：00 +0800
categories: [开发, 教程]
tags: [Flutter, Git, Android Studio]
---

# 解决 Flutter 项目运行中的 Git Path 错误

## 问题描述

在运行 Android Studio 中的 Flutter 项目时，我遇到了以下错误：

```
Flutter device daemon #1 exited (exit code 1)
```

## 问题分析

起初，我尝试按照提示解决这个问题，怀疑是 Windows 系统中设置了最大文件句柄数的限制问题，并且提示给出了超链接搜索：`increase maximum file handles Windows 11.0`。这个搜索结果并没有给我带来太多帮助，因此我决定查看更多的错误信息。

我尝试在命令行中运行了 `flutter doctor`，结果如下：

```shell
PS C:\> flutter
Error: Unable to find git in your PATH.
```

我检查了我的环境变量，发现 Git 已经被正确添加到了 PATH 中。我尝试重新启动了 Android Studio，但问题依然存在。

## 解决方案

通过查阅 [GitHub 上相关问题的讨论](https://github.com/flutter/flutter/issues/137308)，我找到了一个解决方案。可以使用以下命令将 `D:\Code\flutter` 目录添加到 Git 的安全目录列表中：

```shell
PS C:\> git config --global --add safe.directory D:\Code\flutter
```

执行以上命令后，问题成功解决。


