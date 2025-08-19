# V3S 简介与固件编译教程

[TOC]

# 引文
本教程第一部分是嵌入式 Linux 综述, 简要对比介绍主流开发板的方案, 并简述嵌入式 Linux 的开发逻辑. 第二部详细展开 **全志 V3S** 的开发流程与技术细节. 

---

**嵌入式 Linux 主流方案介绍** 时下, 树莓派, 香橙派, 荔枝派, 香蕉派等开发板市场呈现出 **百花齐放** 的繁荣景象. 其中, 基于树莓派 Raspberry Pi, 瑞芯微 Rockchip, 全志 Allwinner, 晶晨 Amlogic 等厂商 SoC 的开发板最为丰富. 此外, 主流开发板也有 ESP32, K210 等. 主要差异对比如下:

**Linux 开发板**
| 厂商 | 代表 SoC  | 性能定位 | 典型开发板 | 特点 |
|--|--|--|--|--|
|树莓派 Raspberry Pi| 博通 BCM27xx | 中高端 (1.5-2GHz) | 树莓派 4/5 | 生态优秀, 社区活跃 |
|瑞芯微 Rockchip| RK3588, RK3506 | 高端 (~2.4GHz) | Radxa ROCK 5B | 高性能视频编解码, NPU |
|全志 Allwinner| H616/D1/F1C200s/V3S | 中低端 (1.5-1.8GHz)| 香橙派, 荔枝派 | 生态较好, 社区适配完善 |
|晶晨 Amlogic| A311, S905 | 中端 (~2.0GHz) | Khadas VIM3 | 高性能 NPU, 视频处理. 生态一般. |

**RTOS 开发板**
| 厂商 | 代表 SoC  | 性能定位 | 典型开发板 | 特点 |
|--|--|--|--|--|
| 乐鑫 Espressif | ESP32 | 边缘无线通信 (240MHz) | 安信可 ESP 模块 | Wi-Fi/蓝牙双模, 高性价比 |
| 嘉楠科技 | K210 | 边缘AI (400MHz) | CamMV, MaixCAM | KPU神经网络加速, 边缘视觉 |

---

其中, 全志系列 soc 生产年份较早, 开源社区适配完善. 且全志推出了部分高性能且硬件友好的甜品款 SoC, 例如 F1C200s, V3s, V831, V851s, 其内部集成了 DDR 内存, 无须外部高速布线; 不采用 BGA 封装, 焊接友好, 4 层甚至 2 层板即可布下. 我们的最终目标是运行 JLinkRemoteServer, 并尽可能选择便宜低端的 SoC. F1C200s 没有 FPU 无法运行 JLinkRemoteServer, V831 已停产价格高. V3s 在某二手平台上价格为 10元/片 拆机带板. 因此笔者选择了 V3s. 


**全志 V3s** 封装内内有一个 ARM Cortex-A7 CPU(32 位) 以及 64MB DDR 内存. 特色是包含 CSI 相机接口, 与 RGB 屏幕接口. 设计之初主要应用于相机类产品, 如行车记录仪, 摄像头, 卡拉 OK 中控等. 沿用此思路, V3S 可成为飞镖视觉制导/无线调参方案之一, 笔者进行过初步的研发, 成果请见 [V3S 在飞镖上的应用分享](DartCV.md). 

软件开发方面, V3S 基于 ARM Cortex 架构, 因此其开发方式与 STM32 有诸多相似之处. 本教程也将与 STM32 开发流程进行对比解析, 以方便了解 STM32 嵌入式开发技术的同学快速入门. 软件适配上主要有全志官方 Camdriod SDK, 主线 Linux, 第三方 XBoot 等裸机框架等丰富选择. 下面是对其简要的介绍.

|软件框架|简介|
|--|--|
|全志官方 Camdriod SDK|内核版本 3.4, 官方主推, 摄像头驱动适配好, 但开发资料少|
|主线 Linux|资料丰富但复杂度高|
|XBoot 等裸机框架|与 STM32 开发高度相似, 但社区支持较差, 无法直接运行 JLinkRemoteServer|

> **名词解释**
> 主线 Linux: 主要指主线内核 https://github.com/torvalds/linux, 是不同发行版修改的源头, 是 Linux 内核的主干与官方发布.
> 裸机: 没有操作系统, 直接运行程序.

笔者选择在 V3S 上运行 **主线 Linux**, 主要是出于学习目的与信仰.

# V3S
**参考资料列表**
- [荔枝派官方资料](https://wiki.sipeed.com/soft/Lichee/zh/Zero-Doc/System_Development/uboot_build.html)
- [CSDN RTL8723 Wifi 模块配置教程](https://blog.csdn.net/qq_28877125/article/details/125092933)

# 软件框架
固件主要包含三个部分, UBoot, Linux Kernel, Buildroot. 
- **UBoot** 为 bootloader, 其功能是初始化 ddr, 初始化时钟等. 同时, UBoot 会从引导介质, 如 TF 卡, SPI Flash, NAND Flash 等处搜索可启动的介质, 并从中加载 Kernal 并运行. 同时也会向内核传递一些启动参数. 详见后文 Uboot 编译部分.
- **Kernel** 即内核, 本质是一个巨大的程序. 负责管理计算机硬件资源，并为应用程序提供接口.
- **Buildroot** 用来构建根文件系统. 即构建我们在根目录下看到的 /var, /opt 等文件夹. 

你可以跟着下文一步步进行配置编译与烧录.

相关链接:
- [荔枝派 UBoot](https://github.com/Lichee-Pi/u-boot/tree/v3s-current)
- [主线 Linux 内核](https://github.com/torvalds/linux) 
- [主线 Buildroot](https://buildroot.org/downloads/buildroot-2019.08.tar.gz)

# 环境配置
本教程基于 Ubuntu 22.04(x86_64) 平台. 笔者实测 WSL, 虚拟机也可正常编译但速度奇慢. 在 13900 平台上全量 rebuild 需要约 20 分钟.

安装编译所需的基本依赖
```bash
sudo apt install git wget make gcc flex bison libssl-dev bc kmod
```

安装 **交叉编译工具链**. 编译平台架构是 x86_64, 与目标架构是 ARM 不同, 因此需要使用交叉编译器. 这一点与 STM32 开发同理, STM32 常见编译器有 Keil 的 ARM Compiler, gcc 系列的 arm-none-eabi- 工具链.  对于此处应用, 我们选择 **arm-linux-gnueabihf** 工具链. 工具链名称中 arm 代表目标架构是 arm, linux 代表可编译在 linux 操作系统上运行的二进制. 此编译器也可编译无操作系统的裸机程序. gnueabihf 全称为 The GNU C compiler for armhf architecture. armhf 中的 hf 意为 hardware floating point, 即硬件 浮点处理器(FPU). V3S 内部有 FPU, 所以我们应当选择 arm-linux-gnueabihf 而非 arm-linux-gnueabi(无法生成 FPU 相关指令)
> 具体命名细节可参考 [CSDN 交叉编译器的命名规则及详细解释](https://blog.csdn.net/LEON1741/article/details/81537529)

> **名词解释**
> 编译器 Compiler: 狭义的编译器只负责编译过程. 
> 工具链 ToolChain: 包括编译器, 链接器 Linker, 调试器 Debugger 等开发全流程所需的工具合集.

我们选择 6.3.1-2017.05 版本的 arm-linux-gnueabihf-gcc. 即 gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf.tar.xz

工具链网址 https://releases.linaro.org/components/toolchain/binaries/6.3-2017.05/arm-linux-gnueabihf/
使用如下命令下载安装工具链:
```bash
wget https://releases.linaro.org/components/toolchain/binaries/6.3-2017.05/arm-linux-gnueabihf/gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf.tar.xz
tar xvf gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf.tar.xz
sudo mv gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf /opt/
sudo vi /etc/bash.bashrc
# 添加到末尾: PATH="$PATH:/opt/gcc-linaro-6.3.1-2017.05-x86_64_arm-linux-gnueabihf/bin"
source /etc/bash.bashrc
arm-linux-gnueabihf-gcc -v
sudo apt-get install device-tree-compiler
```

我们选择将工具链放在 /opt/ 下, 也可放在任何你想要的位置. 注意对应修改 PATH 变量.


# 固件编译
建议新建一个文件夹, 如 ~/V3S 用于存储相关的项目文件夹.

## UBoot
V3s 相关启动代码(ddr 内存, 时钟等初始化)似乎并没有合并到主线 U-Boot. 因此, 我们选择 [荔枝派维护的 U-Boot](https://github.com/Lichee-Pi/u-boot/tree/v3s-current)

使用如下命令 clone 到本地
```bash
git clone https://github.com/Lichee-Pi/u-boot.git -b v3s-current
```

**make menuconfig 简介**
采用 Makefile 构建的大型软件通常使用 menuconfig 可视化调整参数. 如下命令可以进入 menuconfig 界面, 注意需要保持命令行窗口足够大以显示 GUI 界面:
``` bash
cd u-boot
make ARCH=arm menuconfig
```

什么是 menuconfig: 是一个基于 **文本界面** 的配置工具, 终端的字符由文本绘制而成. make 通过读取目录下 **Kconfig** 文件, 并根据 Kconfig 的描述内容生成一个图形化参数配置界面. 当保存退出之后, 配置文件会保存在目录下的 **.config** 文件中.
- 使用上下左右方向键可移动光标, 回车进入, ESC 返回上一级菜单或退出.
- 对于 checkbox 类型配置, 使用空格键切换启用功能或禁用功能:
- [ ]/< >： 该功能未被启用。
- [\*]/<\*>： 该功能将被编译进内核。
- [M]： 该功能将被编译为独立的模块。

---

荔枝派维护的 U-Boot 主要往 U-Boot 添加了驱动代码与默认 config. 我们使用如下命令使用荔枝派给出的默认 config:

```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- LicheePi_Zero_defconfig
```

在荔枝派默认 config 基础上, 我们要做如下的修改:



最后, 我们编译 U-Boot:

```bash
make ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- -j8
```

> -j8 可自由选择编译的最大线程数. 指定为 -j 为不限制.

### Boot arguments
上文提到, U-Boot 也负责将内核加载到内存, 负责向内核传递启动参数. 具体地说, 我们需要告诉 U-Boot 将内核文件, 设备树在哪里, 需要内核加载到哪个内存地址上. 告诉内核将启动信息显示在哪里(如串口或者屏幕上), 
> **名词解释**
> 设备树: 描述一款 CPU 外设配置与板级硬件配置的信息. 内核中的硬件驱动会读取设备树文件. 可以暂时略过, 后文会详细讨论.

可以在 U-Boot 的 menuconfig 里设置这些启动参数, 但需要修改这些参数则需重新编译烧录 U-Boot 较为麻烦.

我们在这里不修改 boot-args 和 boot-cmd， 我们选择生成一个 boot.scr 文件， 然后直接用文件管理器复制到制定分区， 而不作为 uboot 的一部分编译到 u-boot 里。 这两个参数是环境变量， 作用是告诉 uboot 在什么地址加载内核和文件树。U-Boot 会在你的第一启动分区（fat32/exfat）中寻找boot.scr文件，作为启动参数选项。在这个文件中，可以定义一些给内核传递的参数，以及文件加载及启动的命令选项
```bash
setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait panic=10 earlyprintk rw
load mmc 0:1 0x41000000 zImage
load mmc 0:1 0x41800000 sun8i-v3s-licheepi-zero.dtb
bootz 0x41000000 - 0x41800000
```

### Trouble Shooting
make menuconfig 时可能会报错 fatal error: curses.h: No such file or directory，则需要安装支持环境:

```bash
sudo apt-get install libncurses5-dev libncursesw5-dev
```

## TF 卡分区

## Linux Kernel

## Buildroot