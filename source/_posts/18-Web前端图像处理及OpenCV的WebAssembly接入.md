---
title: Web前端图像处理及OpenCV的WebAssembly接入
date: 2023-05-18 10:32:29
tags:
  - 前端
  - WebAssembly
categories:
  - 技术博客
  - 前端学习
---

OpenCV是一个基于Apache2.0许可（开源）发行的跨平台计算机视觉和机器学习软件库，可以运行在Linux、Windows、Android和Mac OS操作系统上。
<!--more-->
它轻量级而且高效——由一系列 C 函数和少量 C++ 类构成，同时提供了Python、Ruby、MATLAB等语言的接口，实现了图像处理和计算机视觉方面的很多通用算法。
有了WebAssembly的应用，OpenCV同时也可以在Web端使用，增强了兼容性和可用性。

# 安装WebAssembly编译环境
* 参考文档：https://emscripten.org/docs/getting_started/downloads.html
* 下载地址：https://github.com/emscripten-core/emsdk

# 编译OpenCV-JS
* 下载OpenCV源代码：https://opencv.org/releases/
* 执行源代码目录下编译脚本：/platforms/js/build_js.py
* 使用编译系统最好为Linux或者MacOS
* 按照提示信息编译即可，需要指定一些参数如：opencv目录，输出目录
* 编译完成后会在输出目录下生成目标文件
* 官方编译参考：https://docs.opencv.org/4.7.0/d4/da1/tutorial_js_setup.html

# OpenCV-JS使用
* 直接使用js接口，可以参考：https://docs.opencv.org/4.7.0/d5/d10/tutorial_js_root.html

# OpenCV静态链接库使用
* 当通用js库不满足需求，需要定制CV算法的时候可以采用
* 实际生产中引入整个OpenCV-JS库也会导致页面请求过大，也需要在C++层做定制修改
* 静态链接库需要使用.a文件作为源文件，在OpenCV-JS编译输出目录下/lib目录下有静态链接库
* 一般需要使用的是libopencv_core.a，libopencv_imgproc.a
* 使用静态链接库可以做到生成的wasm文件大小随需求动态变化，按需索取

# JS与C++图像数据交换方式
* JS层获取图片并传到C++层运算，运算完成后再去读取C++内存的方式交互
* 概括来讲就是利用WebAssembly.HEAPU8使得JS端可以共享C++的堆内存空间
```js
this.$wasm.HEAPU8.set(src.data, heap)
```

* JS层调用C++内存处理方法
```js
let ctx = canvas.getContext('2d')
let srcData = ctx.getImageData(0, 0, canvas.width, canvas.height)
let data = srcData.data
let length = data.length
let heap = this.$wasm['mallocNative'](length)
this.$wasm.HEAPU8.set(src.data, heap)
this.$wasm['processImage'](it.op, src, heap)
let arr = new Uint8ClampedArray(this.$wasm.HEAPU8.buffer, heap, length)
let dstData = new ImageData(arr, canvas.width, canvas.height)
ctx.putImageData(dst, 0, 0)
it.src = canvas.toDataURL('image/jpeg')
this.$wasm['freeNative'](heap)
```

* C++可以使用内存空间创建Mat即可
``` c++
auto *p = (uint8_t *) buf;
int width = img["width"].as<int>();
int height = img["height"].as<int>();
auto raw = Mat(height, width, CV_8UC4, p);
```

* 后续图片处理方式和正常C++调用一样的，例如：
```c++
Mat gray;
cvtColor(raw, gray, COLOR_RGBA2GRAY);
cvtColor(gray, raw, COLOR_GRAY2BGRA);
```

* C++端对HEAPU8修改后，JS端可以直接获取，这也是js和wasm共享内存的一种方式