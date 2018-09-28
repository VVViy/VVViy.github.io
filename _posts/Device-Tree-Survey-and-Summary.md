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
### 0. Preface
本文主要介绍嵌入式系统`Bootloader`中的设备树`(Device Tree)`，援引`Wikipedia`对其定义如下， 简单归纳，`DT`文件是由一系列`Node`构成的树形结构，其中每个`Node`都对应硬件系统中的一个`device`，如`CPU`, `I2C controller`. 目前，`Device Tree (DT)`尚无行业标准，仍在发展中，内容上相对还不庞杂.

本文不会简单综合文档资源进行内容罗列，而是简明概括文档中值得注意的点进行介绍，所以，如果要了解具体的语法和定义，可参考文末的文档列表.

> In computing, a device tree (also written devicetree) is a data structure describing the physical hardware components of a particular
> computer so that the operating system's kernel can use and manage those components, including the CPU or CPUs, the memory, the
> buses and the peripherals.
>
> The device tree was derived from SPARC-based workstations and servers via the Open Firmware project. The current Devicetree
> specification is targeted at smaller systems, but is still used with some server-class systems (for instance, those described by the
> Power Architecture Platform Reference including some Apple Macintoshes).

### 1. How to work
以`Xilinx`的`zynq`系统为例，简单了解`DT`在系统启动阶段如何工作，`UG1156`中介绍了`Linux Boot Process on the Target Platfoem`，如Fig-1. 简单的说，`DT`对目标系统中的物理器件进行静态描述，`kernel`通过查找`DT（.dtb）`完成设备检测和驱动配置.  这种设备描述与设备配置过程的解耦，可以在不改变`kernel`的情况下，直接应用于不同的硬件系统(DT).

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%231.jpg?raw=true" height='500' width="251" />

Fig-1

</div>

### 2. File classification and relationship
`DT`相关的文件包括`.dts ("device tree souce")`, `.dtsi (device tree source include)`, `.dtb ("device tree blob")`三类，其中，`.dts, .dtsi`都是对目标系统硬件的描述，区别在于前者是板级描述文件，后者则是`SoC`级或`Module (板上硬件模块，非RTL module)`描述，如Fig-2，Fig-3. 而`DTB`文件只是`DT compiler`对`.dts`源文件的编译结果，是能够被`kernel`识别读取的压缩格式. 

三种源文件的关联与处理流程如Fig-4所示，从图中可以看到，`DTC`无法直接识别`.dtsi`描述文件，需要`/include/`到`.dts`中.  需要注意的是, 源文件间的`/include/`顺序会影响`Node`属性或其本身的参数覆盖，即如果要对overlay.dtsi中的`node 1`的属性值进行“重载”或“添加”，那么相应的编码应该添加到`/include/ overlay.dtsi`的描述文件中. 

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%232.jpg?raw=true" />

Fig-2 [1]

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%234.jpg?raw=true" />

Fig-3 [2]

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%233.jpg?raw=true" height="500" width="176" />

Fig-4

</div>

### 3. Basic elements within a dtsi

### 4. Some useful tips

#### Reference
[1] T. Petazzoni. Device Tree for Dummies. [Online]: https://events.static.linuxfound.org/sites/events/files/slides/petazzoni-device-tree-dummies.pdf

[2] Toradex AG. Device Tree Customization. [Online]: https://developer.toradex.com/device-tree-customization

[3]

[4]

[5]
