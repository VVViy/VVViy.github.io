---
layout:     post
title:      Device Tree Survey and Summary
subtitle:   LTM
date:       2018-09-13
author:     Max
header-img: img/post-gray-background.jpg
catalog: true
tags:
    - Devicetree
---

### Preface
本文主要介绍嵌入式系统`Bootloader`中的设备树，援引`Wikipedia`对其定义如下，目前，`Device Tree (DT)`尚无行业标准，还在发展中，内容相对不复杂. 本文不会简单综合文档资源进行内容罗列，而是简明概括文档中值得注意的点进行介绍，所以，如果要了解具体的语法和定义，可参考文末的文档列表.

> In computing, a device tree (also written devicetree) is a data structure describing the physical hardware components of a particular 
> computer so that the operating system's kernel can use and manage those components, including the CPU or CPUs, the memory, the buses 
> and the peripherals.
> The device tree was derived from SPARC-based workstations and servers via the Open Firmware project. The current Devicetree 
> specification is targeted at smaller systems, but is 
> still used with some server-class systems (for instance, those described by the Power Architecture Platform Reference including some 
> Apple Macintoshes).

### How to work
以`Xilinx`的`zynq`系统为例，简单了解`DT`在系统启动阶段如何工作，`UG1156`中介绍了`Linux Boot Process on the Target Platfoem`，如Fig-1. 简单的说，`DT`对目标系统中的物理器件进行静态描述，`kernel`通过查找`DT`完成设备检测和驱动配置.  这种设备描述与设备配置过程的解耦，可以在不改变`kernel`的情况下，直接应用于不同的硬件系统(DT).

![]()
Fig-1

### File classification and relationship
`DT`相关的文件包括`.dts`, `.dtsi`, `.dtb`三类，`.dts, .dtsi`都是对目标系统硬件的描述，区别在于描述级别不同，前者是板级描述文件，后者则是`SoC`级描述，如Fig-2，而`DTB`文件只是`DT compiler`的编译结果，是能够被`kernel`读取的格式. 源文件的处理流程如Fig-3所示，  

![]()
Fig-2

![]()
Fig-3 

### Basic elements within a dtsi

### Some useful tips
