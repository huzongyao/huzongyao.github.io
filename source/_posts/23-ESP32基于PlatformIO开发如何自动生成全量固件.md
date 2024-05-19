---
title: ESP32基于PlatformIO开发如何自动生成全量固件
date: 2024-05-20 10:14:34
tags:
  - esp32
  - Arduino
  - PlatformIO

categories:
  - 技术博客
  - PlatformIO学习
---

ESP32 基于Arduino开发时, 我们修改一些代码就可以自动生成全量固件，全量固件只有一个文件，指定起始地址为0x0即可更新，非常简单。
<!--more-->
由于ESP32的flash一般采用分区管理，因此生成的固件通常涉及多个分区，分区的具体大小和起始地址由编译时使用的分区表决定。
多分区使得固件更新时可以个性化更新某个或者某几个分区。但是对于全量更新的用户，又会带来一些麻烦，因为每个分区都需要各自指定起始地址和固件路径。
PlatformIO编译时默认只会生成分区固件，然而只需要加一些配置，也可以自动合并成全量固件。
社区探讨可以参考这个：
https://github.com/platformio/platform-espressif32/issues/1078

# 分区固件与全量固件
分区固件烧写配置示例：
```shell
0x1000 bootloader.bin
0x8000 partitions.bin
0xe000 boot_app0.bin
0x10000 firmware.bin
```
全量固件烧写配置示例：
```shell
0x0 merged-flash.bin
```

# ESPTool合并分区固件
ESPTool专门提供了固件合并功能，文档中有详细介绍:
* https://docs.espressif.com/projects/esptool/en/latest/esp32/esptool/basic-commands.html

当然，PlatformIO在安装时也自带了ESPTool，ESPTool提供了ESP芯片相关的很多软件服务，我们需要使用的是merge_bin这个功能。
官方的说法：merge_bin 命令将多个二进制文件（任何类型）合并为一个文件，该文件可以在稍后被烧录到设备上。根据所选择的输出格式，
输入文件之间的任何空隙都会被填充。也就说只需要使用以下命令，就可完成合并操作。
```shell
esptool.py --chip ESP32 merge_bin -o merged-flash.bin --flash_mode dio --flash_size 4MB 0x1000 bootloader.bin 0x8000 partition-table.bin 0x10000 app.bin
```
这将会创建一个名为 merged-flash.bin 的文件，其中包含其他三个文件的内容。稍后，可以使用 
esptool.py write_flash 0x0 merged-flash.bin 命令将这个文件写入到闪存中。

# PlatformIO 结合ESPTool合并固件
上述操作我们已经可以实现固件合并，因此我们只需要在每次编译完成后，手动执行一次合并命令即可。
然而，我们还想要PlatformIO自动来执行这个合并命令，毕竟命令参数很多。这就需要设计PlatformIO Actions功能

## PlatformIO Actions在定制编译流程
* 参考文档：https://docs.platformio.org/en/latest/scripting/actions.html

按照官方说法，添加action需要2步
### 1. 修改platformio.ini，添加extra_scripts
``` ini
[env:pre_and_post_hooks]
extra_scripts = post:extra_script.py
```

### 2. 编写我们的extra_script.py，使用python语言
```python
env.AddPreAction("buildprog", callback...)
env.AddPostAction("buildprog", callback...)
```

### 3. Post Action与ESPTool结合，实现自动合并
上述extra_script中，Pre & Post Actions说白了就是指编译前和编译后，我们可以让编译器干一些事情。
因此把ESPTool合并ROM操作， 放到Post Actions当中即可。我用的代码如下：
``` python
Import('env')
import os

OUTPUT_DIR = "$BUILD_DIR{}".format(os.path.sep)
APP_BIN = "$BUILD_DIR/${PROGNAME}.bin"


def copy_merge_bins(source, target, env):
    firmware_src = str(target[0])
    flash_images = env.Flatten(env.get("FLASH_EXTRA_IMAGES", [])) + ["$ESP32_APP_OFFSET", APP_BIN]
    name = firmware_src.split(os.path.sep)[2]
    flash_size = env.GetProjectOption("board_upload.flash_size")
    board = env.BoardConfig()
    f_flash = board.get("build.f_flash", "40000000L")
    flash_freq = '40m'
    if (f_flash == '80000000L'):
        flash_freq = '80m'
    mcu = board.get("build.mcu", "esp32")
    firmware_dst = "{}{}_{}_{}_0x0.bin".format(OUTPUT_DIR, mcu, name, flash_size)
    if os.path.isfile(firmware_dst):
        os.remove(firmware_dst)
    cmd = " ".join(
        ["$PYTHONEXE", "$OBJCOPY", '--chip', mcu, 'merge_bin', '--output', firmware_dst, '--flash_mode', 'dio',
         '--flash_size', flash_size, '--flash_freq', flash_freq] + flash_images)
    env.Execute(cmd)


env.AddPostAction("$BUILD_DIR/${PROGNAME}.bin", [copy_merge_bins])

```

附全量固件烧写命令：
```shell
esptool.py write_flash 0x0 merged-flash.bin
```

当然，也可以用图形化工具FlashDownloadTools，下载地址:
https://www.espressif.com/en/support/download/other-tools

总结：使用PlatformIO的extra_scripts配置，我们可以在编译前后添加用户自定义的行为。
我们利用这特性可以做一些编译后处理，不仅限于固件合并，固件压缩，重命名等操作也很常见，看需求吧。