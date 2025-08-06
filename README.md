# RM2025-Wireless-JLink
Wireless JLink-OB based on Allwinner V3S
基于全志科技 V3S 的无线 JLink-OB 烧录器. **完美支持 Ozone, Keil, STM32CubeIDE 等开发软件.**

![alt text](./Images/image.jpg)

实现原理: 在通过 RTL8723 联网的 V3S 上运行 Linux 并开启 JLinkRemoteServer. 处于同一局域网的电脑可通过输入无线烧录器的 IP 地址进行连接.
![alt text](./Images/image-1.png)

**主要参数**
|参数||
|:--|:--|
|尺寸|41x39x5mm|
|JLink版本|JLink-OB|
||4线 SWD接口|

**硬件选型**
|参数||
|:--|:--|
|JLink MCU|STM32F103CB|
|嵌入式 Linux Soc|全志 V3S|
|网卡型号|RTL8723|
