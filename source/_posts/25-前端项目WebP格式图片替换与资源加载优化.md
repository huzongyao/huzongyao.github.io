---
title: 前端项目WebP格式图片替换与资源加载优化
date: 2024-09-10 10:11:24
tags:
  - 前端
  - vue
categories:
  - 技术博客
  - 前端学习
---

WebP是一种同时提供了有损压缩与无损压缩（可逆压缩）的图片文件格式，派生自影像编码格式VP8，被认为是WebM多媒体格式的姊妹项目，
是由Google在购买On2 Technologies后发展出来，以BSD授权条款发布。
<!--more-->
WebP最初在2010年发布，目标是减少文件大小，但达到和JPEG格式相同的图片质量，希望能够减少图片档在网络上的发送时间。2011年11月8日，Google开始让WebP支持无损压缩和透明色（alpha通道）的功能，而在2012年8月16日的参考实做libwebp
0.2.0中正式支持。根据Google较早的测试，WebP的无损压缩比网络上找到的PNG档少了45%的文件大小，即使这些PNG档在使用pngcrush和PNGOUT处理过，WebP还是可以减少28%的文件大小。

# 将项目中的图片替换为WebP格式
使用命令：
```shell
for file in *.png; do
  cwebp -q 80 "$file" -o "${file%.png}.webp"
done
```
gif动图也可以用ffmpeg进行转换，输出文件可以循环动画：
```shell
ffmpeg -i input.gif -loop 0 -c:v libwebp -an output.webp
```

# 其他相关资料
webp浏览器支持情况：https://caniuse.com/webp
