---
layout:     post
title:      Device Tree Survey and Summary
subtitle:   LTM
date:       2018-09-30
author:     Max
header-img: img/post-gray-background.jpg
catalog: true
tags:
    - Devicetree
---

### 0. Preface
本文主要介绍嵌入式系统`Bootloader`中的设备树`(Device Tree)`，援引`Wikipedia`对其定义如下， 简单归纳，`DT`文件是由一系列`Node`构成的树形结构，其中每个`Node`都对应硬件系统中的一个`device`，如`CPU`, `I2C controller`. 目前，`Device Tree (DT)`尚无行业标准，仍在发展中，内容上还不算庞杂，短时间内可掌握，其对参与嵌入式开源项目，特别是RISC-V还是有一定价值的.

本文不会简单综合文档资源进行内容罗列，而是简明概括文档中值得注意的点，所以，在内容完整性和连贯性上可能欠佳，如果要了解全部语法和参数定义，可参考文末的文档列表.

> In computing, a device tree (also written devicetree) is a data structure describing the physical hardware components of a particular
> computer so that the operating system's kernel can use and manage those components, including the CPU or CPUs, the memory, the
> buses and the peripherals.
>
> The device tree was derived from SPARC-based workstations and servers via the Open Firmware project. The current Devicetree
> specification is targeted at smaller systems, but is still used with some server-class systems (for instance, those described by the
> Power Architecture Platform Reference including some Apple Macintoshes).

### 1. How to work
以`Xilinx`的`zynq`系统为例，简单了解`DT`在系统启动阶段如何工作，`UG1156`中介绍了`Linux Boot Process on the Target Platform`，如Fig-1. 简单的说，`DT`对目标系统中的物理器件进行静态描述，`kernel`通过查找`DT（.dtb）`完成设备检测和驱动配置.  这种设备描述与设备配置过程的解耦，可以在不改变`kernel`的情况下，直接应用于不同的硬件系统(DT).

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%231.jpg?raw=true" height='500' width="251" />

Fig-1

</div>

### 2. File classification and relationship
`DT`通常由一系列描述文件构成，而非单一文件，不同文件描述了目标硬件系统的不同组成部分，相关的文件包括`.dts ("device tree souce")`, `.dtsi (device tree source include)`, `.dtb ("device tree blob")`三类.  其中，`.dts, .dtsi`都是对目标系统硬件的描述，区别在于前者是板级描述文件，后者则是`SoC`级或`Module级 (板上硬件模块，非RTL module)`描述，如Fig-2~Fig-3. 而`DTB`文件只是`DT compiler(DTC)`对`.dts`源文件的编译结果，是能够被`kernel`识别读取的压缩格式.

三种源文件的关联与处理流程如Fig-4所示，从图中可以看到，`DTC`无法直接识别`.dtsi`描述文件，需要`/include/`到`.dts`中.  需要注意的是, 源文件间的`/include/`顺序会影响`Node`属性或其本身的参数覆盖，即如果要对overlay.dtsi中的`node 1`的属性值进行“重载”或“添加”，那么相应的编码应该添加到`/include/ overlay.dtsi`的描述文件中.

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%232.jpg?raw=true" />

Fig-2 [1]

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%234.jpg?raw=true" />

Fig-3 [2]

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%233.jpg?raw=true" height="500" width="176" />

Fig-4

</div>

### 3. Basic elements within a dts(i)
1.Tree structure.  前述，每个`.dts(i)`都描述了目标系统的部分硬件模块，且文件内部组织形式皆为树形结构，如Fig-5所示，即每个文件内部都有一个或多个`root node ("/")`，其他每个`Node`都描述了一个device或module.  每个`Node`的基本结构如Fig-6所示，由三部分构成，名称+描述域控制("{ };")+属性，需要说明的是

* 与normal node相比，root node没有“label_name: node_name@unit_address”，而是使用“/”作为节点名称；
* 所有normal node必须在root node的控制域内做“初次声明”，如Fig-6后的实例，有一个例外是在文件中对已声明节点做“属性重载或添加”或“节点重载”时，需要将重载描述写在根节点控制域范围以外. 但作者不确定是否所有vendor都如此，毕竟没有统一标准，至少作 者看见的案例均遵照此规则，如xilinx要求在重载描述的文件中，新添加的("初次声明")节点在根节点控制域内描述，重载描述则在根节点控制域以外添加；（本文档是long-term maintenance，会持续添加新东西.）
* 一个文件中的根节点数目并不是唯一的，可以同时存在多个根节点，即一个.dts(i)内部可以是树或森林；从Fig-4可以看到，所有的.dts(i)最后会组成一个“top”文件，所以，不同文件中即使存在多个根节点最后都会融合在一起.  实际上，所谓的“描述控制域”是作者自定义的说法，其主要意义在于将“brother nodes”和“parent-child nodes”圈定，但不同根节点控制域之间的关系，作者尚不知晓，若有同道了解其中原理还请告知；

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%235.jpg?raw=true" />

Fig-5 [3]

```
//normal node definition
label:node_name@unit_addrss{
    all of properties declaration;
};
```

</div>

``` //example
/ {
    compatible = "acme,coyotes-revenge";
    #address-cells = <1>;
    #size-cells = <1>;
    interrupt-parent = <&intc>;

    cpus {
        #address-cells = <1>;
        #size-cells = <0>;

        cpu@0 {
            compatible = "arm,cortex-a9";
            reg = <0>;
        };

        cpu@1 {
            compatible = "arm,cortex-a9";
            reg = <1>;
        };

    ......

};
```

2.Node structure.  

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%237.jpg?raw=true" />

Fig-7

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%238.jpg?raw=true" />

Fig-8

</div>

### 4. Some useful tips

#### Reference
[1] T. Petazzoni. Device Tree for Dummies. [Online]: https://events.static.linuxfound.org/sites/events/files/slides/petazzoni-device-tree-dummies.pdf

[2] Toradex AG. Device Tree Customization. [Online]: https://developer.toradex.com/device-tree-customization

[3] Devicetree.org. Devicetree Specification (rev0.2). [Online]: https://www.devicetree.org/specifications/

[4] Power.org. Embedded Power Architecture™ Platform Requirements(ePAPR).[Online]: https://elinux.org/images/c/cf/Power_ePAPR_APPROVED_v1.1.pdf

[5] eLinux.org. Device Tree Usage. [Online]: https://elinux.org/Device_Tree_Usage

[6] NXP. Introduction to Device Trees. [Online]: https://www.nxp.com/docs/en/application-note/AN5125.pdf

[7] Raspberry PI Foudation. DEVICE TREES, OVERLAYS, AND PARAMETERS. [Online]: https://www.raspberrypi.org/documentation/configuration/device-tree.md
