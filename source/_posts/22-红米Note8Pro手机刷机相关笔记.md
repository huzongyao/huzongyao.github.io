---
title: 红米Note8Pro手机刷机相关笔记
date: 2024-02-29 08:19:50
tags:
  - android
categories:
  - 技术博客
  - Android学习
---

红米Note8Pro手机刷机相关笔记
<!--more-->
该手机是MTK芯片

## 系统下载和烧写

1. 官方下载地址：https://xiaomirom.com/series/begonia/
2. 下载选项：我下载的时候可以选择国行和海外版，国行没有GP，想要GP需要下载海外版。
3. ROM类型：分为fastboot版和recovery版，分别是线刷和卡刷。

## TWRP刷写：

1. 安卓11的下载地址：https://www.pling.com/p/1556853/

## ROOT工具：

1. 获取ROOT工具Magisk：https://github.com/topjohnwu/Magisk

## 反向周边工具：

1. 抓包工具reqable: https://reqable.com/en-US/download/
2. 抓包代理XSocksDroid：https://github.com/zhuhai-and/XSocksDroid
3. Hook代理Frida：https://github.com/frida/frida
4. Frida启动器：https://github.com/metmit/FridaManager
5. Jadx:https://github.com/skylot/jadx

## ROOT常用操作

1. 卸载系统应用：
    ```shell
    pm uninstall --user 0 com.android.chrome
    ```

2. 查看应用路径：
    ```shell
    pm path com.android.chrome
    ```