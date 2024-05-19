---
title: 杜丫丫学习机Y1Max刷机ROOT笔记
date: 2024-05-24 14:12:14
tags:
  - android

categories:
  - 技术博客
  - Android学习
---

杜丫丫Y1Max刷机按照普通的MTK刷机流程来就行
<!--more-->

# 线刷救砖包

1. 熟悉刷机方法的话，直接下载固件刷机吧，固件地址：https://www.123pan.com/s/Ea1uVv-oUem3.html提取码:u61V
2. 固件包内容：
    * 买机器备份的固件，用于救砖
    * 已ROOT固件，删除了无用APP
    * Magisk Boot分区镜像

# 刷机流程

### MTK线刷相关

使用下面MTK刷机工具和方法，基本可以把固件线刷到机器上。

1. MTK线刷工具SPFlashTool：https://spflashtool.com/
2. SPFlashTool另一个地址：https://androidmtk.com/smart-phone-flash-tool
3. MTK刷机驱动下载地址：https://androidmtk.com/download-mtk-driver-auto-installer
4. MTK线刷教程：https://androidmtk.com/flash-stock-rom-using-smart-phone-flash-tool

### Fastboot刷机

从安卓模式进入fastboot模式：

```shell
adb reboot bootloader
```

fastboot模式刷入boot

```shell
fastboot flash boot boot.img
```

fastboot模式刷入logo

```shell
fastboot flash logo logo.bin
```

fastboot清空用户数据

```shell
fastboot erase userdata
```
