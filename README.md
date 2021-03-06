# Porting NathanReeves esp-idf to PlatformIO 

实在没办在VScode + Pio 上正常使用[NathanReeves 魔改版的esp-idf](https://github.com/NathanReeves/esp-idf), 故将NathanReeves 的代码移植到Pio 的framework-espidf.

初始工程基于以下版本:
* PlatformIO framework-espidf v3.3.0
* ESP-IDF v4.2.1

# 使用

Pio先装好framework-espidf v3.3.0，在"~/.platformio/packages/framework-espidf"下会有Pio版的IDF，直接把framework-espidf 文件夹整个删除，通过git 工具拉取此库替代原来的framework-espidf 文件夹，之后就按正常的framework-espidf v3.3.0 使用，<br/>
千万不要更新framework-espidf！<br/>
千万不要更新framework-espidf！<br/>
千万不要更新framework-espidf！<br/>
或者在platformio.ini 中固定版本号：<br/>
    platform = espressif32@3.3.0 <br/>
这样即使更新了也可以保留v3.3.0版本的platform，其它工程可以使用新版本的platform。<br/>

# 问题

* <s>2021-07-17
新开的项目目前尚未可用，IDF 自4.0后更改了很多宏的命名，个人没办法尽数修改及适配，望各路大神相助！</s><br/>
2021-08-01
宏命名基本解决:</br>
其中有几个宏在新版的IDF 中改动太大甚至已经不在menuconfig 里了, 我放到了Firmware 的Pio 配置文件里定义，参照[我的Firmware 工程](https://github.com/MagicInstall/JoyShot)中[platformio.ini](https://github.com/MagicInstall/JoyShot/blob/master/platformio.ini)里的几个预编译宏。</br>
* <s>2021-08-11
在某些游戏中，手柄会被Switch 置于后台（不知道这个状态怎么表达...），此时手柄会发生ESP_BT_GAP_MODE_CHG_EVT 事件，状态码由ESP_BT_PM_MD_SNIFF
转为ESP_BT_PM_MD_ACTIVE，个人估计是V12 通知手柄切换到某种省电模式的过程，但此时立即就断连了！</br>
参考了一些Issues，瞎猜貌似是Switch 改变了通信间隔，而IDF没有相应的对策导致？<br/>
望各路大神相助！</s><br/>
2021-08-25
之前脑残了，pro 手柄在Splatoon 2下，非P1 是会被强制断开的，而JC 只连接一边，作为P2~4 是不会断连的...<br/>
顺便改了NathanReeves 版的api 原本没有的配对过的手柄自动重连功能，自己加上去了，由于水平有限，未能在BTC 层实现，绕过了BTC 直接调用了BTA，望各路大神指教如何做成通过BTC 层实现！
* 早期我个人只能尽力做出joycon相关的部分...</br>

# 参考
[https://github.com/NathanReeves/esp-idf](https://github.com/NathanReeves/esp-idf)<br/>
[https://github.com/dekuNukem/Nintendo_Switch_Reverse_Engineering](https://github.com/dekuNukem/Nintendo_Switch_Reverse_Engineering)<br/>

# 
# -------------- 以下是原IDF 的Readme --------------

# Developing With ESP-IDF

## Setting Up ESP-IDF

See setup guides for detailed instructions to set up the ESP-IDF:

| Chip | Getting Started Guides for ESP-IDF |
|:----:|:----|
| <img src="docs/_static/chip-esp32.svg" height="85" alt="ESP32"> |  <ul><li>[stable](https://docs.espressif.com/projects/esp-idf/en/stable/get-started/) version</li><li>[latest (master branch)](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/) version</li></ul> |
| <img src="docs/_static/chip-esp32-s2.svg" height="100" alt="ESP32-S2"> | <ul><li>[latest (master branch)](https://docs.espressif.com/projects/esp-idf/en/latest/esp32s2/get-started/) version</li></ul> |

**Note:** Each ESP-IDF release has its own documentation. Please see Section [Versions](https://docs.espressif.com/projects/esp-idf/en/latest/esp32/versions.html) how to find documentation and how to checkout specific release of ESP-IDF.

### Non-GitHub forks

ESP-IDF uses relative locations as its submodules URLs ([.gitmodules](.gitmodules)). So they link to GitHub.
If ESP-IDF is forked to a Git repository which is not on GitHub, you will need to run the script
[tools/set-submodules-to-github.sh](tools/set-submodules-to-github.sh) after git clone.
The script sets absolute URLs for all submodules, allowing `git submodule update --init --recursive` to complete.
If cloning ESP-IDF from GitHub, this step is not needed.

## Finding a Project

As well as the [esp-idf-template](https://github.com/espressif/esp-idf-template) project mentioned in Getting Started, ESP-IDF comes with some example projects in the [examples](examples) directory.

Once you've found the project you want to work with, change to its directory and you can configure and build it.

To start your own project based on an example, copy the example project directory outside of the ESP-IDF directory.

# Quick Reference

See the Getting Started guide links above for a detailed setup guide. This is a quick reference for common commands when working with ESP-IDF projects:

## Setup Build Environment

(See the Getting Started guide listed above for a full list of required steps with more details.)

* Install host build dependencies mentioned in the Getting Started guide.
* Run the install script to set up the build environment. The options include `install.bat` or `install.ps1` for Windows, and `install.sh` or `install.fish` for Unix shells.
* Run the export script on Windows (`export.bat`) or source it on Unix (`source export.sh`) in every shell environment before using ESP-IDF.

## Configuring the Project

* `idf.py set-target <chip_name>` sets the target of the project to `<chip_name>`. Run `idf.py set-target` without any arguments to see a list of supported targets.
* `idf.py menuconfig` opens a text-based configuration menu where you can configure the project.

## Compiling the Project

`idf.py build`

... will compile app, bootloader and generate a partition table based on the config.

## Flashing the Project

When the build finishes, it will print a command line to use esptool.py to flash the chip. However you can also do this automatically by running:

`idf.py -p PORT flash`

Replace PORT with the name of your serial port (like `COM3` on Windows, `/dev/ttyUSB0` on Linux, or `/dev/cu.usbserial-X` on MacOS. If the `-p` option is left out, `idf.py flash` will try to flash the first available serial port.

This will flash the entire project (app, bootloader and partition table) to a new chip. The settings for serial port flashing can be configured with `idf.py menuconfig`.

You don't need to run `idf.py build` before running `idf.py flash`, `idf.py flash` will automatically rebuild anything which needs it.

## Viewing Serial Output

The `idf.py monitor` target uses the [idf_monitor tool](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/idf-monitor.html) to display serial output from ESP32 or ESP32-S Series SoCs. idf_monitor also has a range of features to decode crash output and interact with the device. [Check the documentation page for details](https://docs.espressif.com/projects/esp-idf/en/latest/get-started/idf-monitor.html).

Exit the monitor by typing Ctrl-].

To build, flash and monitor output in one pass, you can run:

`idf.py flash monitor`

## Compiling & Flashing Only the App

After the initial flash, you may just want to build and flash just your app, not the bootloader and partition table:

* `idf.py app` - build just the app.
* `idf.py app-flash` - flash just the app.

`idf.py app-flash` will automatically rebuild the app if any source files have changed.

(In normal development there's no downside to reflashing the bootloader and partition table each time, if they haven't changed.)

## Erasing Flash

The `idf.py flash` target does not erase the entire flash contents. However it is sometimes useful to set the device back to a totally erased state, particularly when making partition table changes or OTA app updates. To erase the entire flash, run `idf.py erase_flash`.

This can be combined with other targets, ie `idf.py -p PORT erase_flash flash` will erase everything and then re-flash the new app, bootloader and partition table.

# Resources

* Documentation for the latest version: https://docs.espressif.com/projects/esp-idf/. This documentation is built from the [docs directory](docs) of this repository.

* The [esp32.com forum](https://esp32.com/) is a place to ask questions and find community resources.

* [Check the Issues section on github](https://github.com/espressif/esp-idf/issues) if you find a bug or have a feature request. Please check existing Issues before opening a new one.

* If you're interested in contributing to ESP-IDF, please check the [Contributions Guide](https://docs.espressif.com/projects/esp-idf/en/latest/contribute/index.html).


