---
title: MTK平台安卓ROM修改和刷机相关笔记
date: 2023-10-19 11:35:22
tags:
  - android

categories:
  - 技术博客
  - Android学习
---

MTK相关操作和修改。
<!--more-->
记录MTK刷机及修改所用一些工具和技巧。

## Vbmeta关闭avb2.0校验
vbmeta分区中一般记录了boot, system, vendor镜像校验信息，校验会导致修改过的boot等镜像刷入后无法启动，
所以一般需要关闭这个校验。

1. 命令方式(重新刷如vbmeta):
```shell
fastboot --disable-verity --disable-verification flash vbmeta vbmeta.img
```

2. 手动修改ROM：使用二进制编辑工具，如HexEditor修改(VsCode也有HexEdit插件，非常方便)，具体修改位置：
* 第123字节：0x00 -> 0x03
* 修改完成之后刷入vbmeta分区即可，所产生的作用和第一种方式是一样的

## MTK工具软件MTKClient使用
MTKClient有一个带界面的工具，首次连接需要在开机时按下音量键，连接成功后可以做很多ROM操作
1. ROM分区操作（导出、刷入、清空）
2. flash操作（对整块flash导出、刷入、清空）
3. 解除BL锁，回锁

## MTK刷机SPFlashTool
SPFlashTool是官方刷机软件，有些功能与MTKClient类似，https://spflashtool.com/
可以执行固件刷入，导出，flash导出，内存测试等功能

## MTK开机画面修改工具LogoBuilder
使用这个工具可以修改设备开机第一屏，可以执行解压和重打包任务，解压后我们可以对相关的png图片进行修改，
完成后再打包成bin文件即可

## MTK_WWR分区切分及刷机固件制作工具
1. 分区切割：我们可以用MTKClient或者SPFlashTool导出整块flash数据，再使用WWR来嗅探分区表并切割这些分区，
   切割后生成的分区文件和分区表文件可以用于SPFlashTool刷机，也可以使用fastboot刷机
2. 我们使用SPFlashTool来导出分区时通常需要一个分区表txt文件，该工具可以根据芯片型号生成该文件。

