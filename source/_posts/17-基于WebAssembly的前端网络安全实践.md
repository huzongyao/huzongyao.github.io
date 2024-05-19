---
title: 基于WebAssembly的前端网络安全实践
date: 2023-05-12 13:41:21
tags:
  - 前端
  - WebAssembly
categories:
  - 技术博客
  - 前端学习
---


WebAssembly（缩写为Wasm）是一种用于基于堆栈的虚拟机的二进制指令格式。Wasm 被设计为编程语言的可移植编译目标，支持在 Web 上部署客户端和服务器应用程序。
<!--more-->

## 使用场景
WebAssembly是一种运行在现代网络浏览器中的新型代码，并且提供新的性能特性和效果。
它设计的目的不是为了手写代码而是为诸如 C、C++和Rust等低级源语言提供一个高效的编译目标。
* 拥有更好的计算性能
* 受益于现代C++的模板，常量表达式，宏定义等特性
* 提高安全性，比js更加难以实施逆向，可以实施编译时加密技术
* 受益于已存在的大量开源C/C++开源库，可以移植使用

## 实现功能
* 基于C++14常量表达式和模板的字符串编译时加密
* 在类unix系统上编译openssl，静态链接到工程中，可用于方便的实现各种加密手段
* 基于openssl，实现sha1, sha256, aes256, base64算法demo
* 基于nlohmann::json，实现方便使用的json操作demo

## WebAssembly快速上手玩法
### 1. 下载跨平台编译SDK：
    * 参考文档：https://emscripten.org/docs/getting_started/downloads.html
    * 下载地址：https://github.com/emscripten-core/emsdk

### 2. 初始化SDK
按照文档初始化SDK

```shell
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh
```

### 3. 编译C/C++代码：
* 编译第一个程序很简单，类似于用gcc/clang编译一个程序：
```c++
//main.cpp
#include <stdio.h>

int main() {
    printf("你好，世界！\n");
    return 0;
}
```

* 编译很简单
```shell
emcc main.cpp
```

* 编译可以直接生成html文件，方便调试
```shell
emcc main.cpp -o main.html
```

* 编辑代码可以采用Clion/VsCode等

* 复杂程序编译也需要按需求加入更多源文件和编译选项
```shell
cd ./cpp
emcc -o ../src/wasm/emjob.mjs -std=c++17 -lembind -I./libs/openssl main.cpp libs/openssl/libcrypto.a
```

* 或者直接执行make.bat
* 其他编译选项可以参考cpp/make.bat
* emcc编译选项可以参考 https://emscripten.org/docs/tools_reference/emcc.html
* 基本原理：emcc编译源文件产生对应的js文件及wasm文件，以供后续在前端代码中调用

### 4. 在Vue中使用wasm
* 上一步编译成功会生成src/wasm/emjob.mjs，在vue项目中可以方便的引用wasm模块：
```javascript
import Module from "@/wasm/emjob"
Module().then(module => {
    app.config.globalProperties.$wasm = module
    app.mount('#app')
})
```

实例中wasm异步加载放在了整个vue加载之前，如果影响前端代码执行，也可以改到适合的地方调用即可

### 5. wasm桥接
* 最常用的js调c++，就很简单，例如：
```c++
void fun1(){}
void fun2(){}

EMSCRIPTEN_BINDINGS(my_module) {
    function("fun1", &fun1);
    function("fun2", &fun2);
}
```

* 更多可以参考文档：https://emscripten.org/docs/porting/index.html

### 6. 后续步骤
* 标准vue前端项目运行或打包即可

### 7. 项目实践中的实际问题
#### 域名验证防止wasm被冒用

用wasm固然可以隐藏一些运算逻辑，使运算逻辑难以被逆向成功，只少比js逻辑保密性好一些吧，但是如果客人果真调试我们的网站，
发现关键逻辑是wasm写的，大概率会想到直接冒用我们的wasm，也不用管算法如何，能用就行。

对于这种情况，我们最简单的防御方法就是做一下域名验证吧，比如一个签名算法，我们可以做个判断，只有当前域名为localhost，
我们才返回正确的签名，否则就返回一个错误的值。客人发现运算结果是错的，不一定第一时间想到是域名问题，也许就知难而退了。
当然这个方法也不能完全保证安全，如果客人知道了这个判断，他们伪造个hostname就可以了

具体校验函数实现如下：
``` c++
bool inline isHostValid() {
    emscripten::val location = emscripten::val::global(_c("location"));
    auto host = location[_c("hostname")].as<string>();
    return find(VHOST.begin(), VHOST.end(), host) != VHOST.end();
}
```
检验函数原理就是我们通过全局变量 location.hostname 获取到当前域名，跟合法域名列表 VHOST 进行比对，结果会返回true/false
在业务逻辑开始之前调用该函数判断一下，就知道域名是否合法了。

#### 编译OpenSSL

OpenSSL是开源的加解密库，我们先编译成静态链接库，以方便使用。
交叉编译机器需要linux或者mac（配置方便一些），可以先下载并解压OpenSSL，使用3.x版本或者1.x版本均可。
配置编译选项：
```
emconfigure ./Configure gcc
```

开始编译：
```
make -j32
```
过一会儿生成文件libcrypto.a，以及include下的头文件就是我们需要的

#### Emscripten结合到CLion
在Clion中开发C、C++很舒服，但是默认没有支持Wasm。我们想要使用emcmake来进行源代码关联，并编译程序，其实也很简单。
需要修改一个设置：settings > Build, Execution, Deployment > CMake，创建一个新的配置
```shell
-DCMAKE_TOOLCHAIN_FILE=<path_to_emsdk>/emscripten//cmake/Modules/Platform/Emscripten.cmake
```
这样就完美解决了代码的关联提示和编译问题，但局限是只能适用于CMake的编译。

