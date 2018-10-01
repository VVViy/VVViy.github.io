---
layout:     post
title:      Device Tree Survey and Summary
subtitle:   LTM
date:       2018-10-02
author:     Max
header-img: img/post-gray-background.jpg
catalog: true
tags:
    - Devicetree
    - Embedded
---

### I. Preface
本文主要介绍嵌入式系统`Bootloader`中的设备树`(Device Tree)`，援引`Wikipedia`对其定义如下， 简单归纳，`DT`文件是由一系列`Node`构成的树形结构，其中每个`Node`都对应硬件系统中的一个`device`，如`CPU`, `I2C controller`. 目前，`Device Tree (DT)`尚无行业标准，仍在发展中，内容上还不算庞杂，短时间内可掌握，其对参与嵌入式开源项目，特别是RISC-V还是有一定价值的.

本文不会简单综合文档资源进行内容罗列，而是简明概括文档中值得注意的点，所以，在内容完整性和连贯性上可能欠佳，如果要了解全部语法和参数定义，可参考文末的文档列表.

> In computing, a device tree (also written devicetree) is a data structure describing the physical hardware components of a particular
> computer so that the operating system's kernel can use and manage those components, including the CPU or CPUs, the memory, the
> buses and the peripherals.
>
> The device tree was derived from SPARC-based workstations and servers via the Open Firmware project. The current Devicetree
> specification is targeted at smaller systems, but is still used with some server-class systems (for instance, those described by the
> Power Architecture Platform Reference including some Apple Macintoshes).

### II. How to work
以`Xilinx`的`zynq`系统为例，简单了解`DT`在系统启动阶段如何工作，`UG1156`中介绍了`Linux Boot Process on the Target Platform`，如Fig-1. 简单的说，`DT`对目标系统中的物理器件进行静态描述，`kernel`通过查找`DT（.dtb）`完成设备检测和驱动配置.  这种设备描述与设备配置过程的解耦，可以在不改变`kernel`的情况下，直接应用于不同的硬件系统(DT).

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%231.jpg?raw=true" height='500' width="251" />

Fig-1

</div>

### III. File classification and relationship
`DT`通常由一系列描述文件构成，而非单一文件，不同文件描述了目标硬件系统的不同组成部分，相关文件类型包括以下三种：
* `.dts`---`"device tree souce"`, `.dts, .dtsi`都是对目标系统硬件的描述，区别在于`.dts`是板级描述文件；
* `.dtsi`---`"device tree source include"`, 是`SoC`级或`Module`级 (板上硬件模块，非RTL module)描述文件，如Fig-2~Fig-3；
* `.dtb`---`"device tree blob"`，`DTB`文件只是`DT compiler(DTC)`对`.dts`源文件的编译结果，是能够被`kernel`识别读取的压缩格式.

三种源文件的关联与处理流程如Fig-4所示，从图中可以看到，`DTC`无法直接识别`.dtsi`描述文件，需要`/include/`到`.dts`中.  需要注意的是, 源文件间的`/include/`顺序会影响`Node`属性或其本身的参数覆盖，即如果要对overlay.dtsi中的`node 1`的属性值进行“重载”或“添加”，那么相应的编码应该添加到`/include/ overlay.dtsi`的描述文件中.

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%232.jpg?raw=true" />

Fig-2 [1]

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%234.jpg?raw=true" />

Fig-3 [2]

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%233.jpg?raw=true" height="500" width="176" />

Fig-4

</div>

### IV. Basic elements within a dts(i)
1.Tree structure.  前述，每个`.dts(i)`都描述了目标系统的部分硬件模块，且文件内部组织形式皆为树形结构，如Fig-5所示，即每个文件内部都有一个或多个`root node ("/")`，其他每个`Node`都描述了一个device或module.  每个`Node`的基本结构如Code-1代码段所示，由三部分构成，`名称+描述域控制("{ };")+属性/子节点定义`，需要说明的是:

* 与`normal node`相比，`root node`没有`label_name: node_name@unit_address`，而是使用“/”作为节点名称, 同时，除了根节点，每个`normal node`都有一个父节点；
* 所有`normal node`必须在`root node`的控制域内做“初次声明”，如Code-2代码段，有一个例外是在文件中对已声明节点做“属性重载或添加”或“节点重载”时，需要将重载描述写在根节点控制域范围以外. 但作者不确定是否所有manufacturer都如此，毕竟没有统一标准，至少作者看见的案例均遵照此规则，如`xilinx`要求在重载描述的文件中，新添加的("初次声明")节点在根节点控制域内描述，重载描述则在根节点控制域以外添加（本文档是`long-term maintenance`，会持续添加新东西.）;
* 一个文件中的根节点数目并不是唯一的，可以同时存在多个根节点，即一个.dts(i)内部可以是树或森林；从Fig-4可以看到，所有的.dts(i)最后会组成一个“top”文件，所以，不同文件中即使存在多个根节点最后都会融合在一起.  实际上，所谓的“描述控制域”是作者自定义的说法，其主要意义在于将“brother nodes”和“parent-child nodes”圈定，但不同根节点控制域之间的关系，作者尚不知晓，若有同道了解其中原理还请告知.

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%235.jpg?raw=true" />

Fig-5 [3]

</div>

```
//Code-1：normal node definition
[label:] node_name [@unit_addrss] {
                                    [properties definition;]
                                    [child nodes]
};
```

```
//Code-2：DT example
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
     ......

};
```

2.Node structure.  Fig-7~Fig-8描述了`node`结构中的组成元素，需要说明的是:
* `unit-address`: 对于访问`DT`的程序而言，`unit-address`是访问`node`的主要入口地址，其值需与`node`内部的`reg=<address，length>`属性中的`address`匹配，若某`node`无`reg`属性，那么节点名称需去掉`@unit-address`部分；
* `cell`: 节点内部大多数非字符串属性值，以`cell` 为单位，32-bit，如在32-bit系统中某节点属性`reg=<0x0 0x10>`，表示该节点对应的`device`占用一块起始地址为`0x00000000`，大小为16 bit的存储空间，而若在64-bit系统中描述该节点，则要应改写为`reg=<0x0 0x0 0x0 0x10>`，即分别使用两个`cell`表示起始地址和空间大小；
* `label`: 节点前的`label`是可选的，另外，`label`不仅可以放在节点名称前，还可以放在节点内部的属性前，即`label-name: property=property-value`;
* `property value`: 属性值类型随属性的不同而不同，可参考文档，需要说明的是，有些属性的值可以为"空"，即只保留属性名，如`ranges；`，而"空"所代表的意义依赖于其所依附的"属性"，请参考文档定义；
* `phandle`: 类似于windows编程里的“指针句柄”，只不过在那里指向窗口控件，而这里是指向一个`node`，有两种基本用法，即

    - 直接在`node`内部为`phandle`属性赋值，且该值不能与其他节点的`phandle`属性值重复，即在整个`DT`中唯一标识该`node`，以便其他节点能够引用该节点，如Code-3；
    - 无需在`node`内部为`phandle`赋值，而是直接在其他节点中引用该`node`的`label`，如Code-4.

  实际上，一般在`DT`的节点中很少显示声明`phandle`属性，除了显示使用`label`，`DTC`会自动在节点中插入`phandle`.
  
```
//Code-3: phandle example 1
referenced-node {
                    phandle = <1>;
                    ......
};

another-device-node {
                        interrupt-parent = <1>; //reference the above "referenced-node"
                        ......
};
```

```
//Code-4: phandle example 2
label: referenced-node {
                          ......
};

another-device-node {
                        interrupt-parent = <&label>; //reference the above "referenced-node"
                        ......
};
```

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%237.jpg?raw=true" />

Fig-7 [1]

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%233-%238.jpg?raw=true" />

Fig-8 [2]

</div>

### V. Some useful tips
本节会介绍一些在自定义或修改`DT`时，需要注意的一些内容，对比查证后，会逐步添加.

1.Overwriting node's properties

节点属性重载应该是使用最多的操作之一，因为不管是自动还是手动生成的`DT`文件，在后续系统使用过程中，我们可能需要对其进行重置，如修改某些硬件模块的可用状态`status = "disabled"`--->`status = "okay"`，即`from "everything on" to "everything off unless requested by the DTB"`原则. 但很奇怪，大多文档中并没交代相关语法，且不同manufacturer的复写方式不同，

* Xilinx与NXP版本[4] (已验证)：直接引用已声明节点的标签，并在控制域内对指定属性重新赋值，如

```
//Code-5: Xilinx/NXP overwriting and appending properties
//child.dtsi
existed-node-label: existed-node {
                                    compatible = "nxp, imx6";
                                    ......
};

//parent.dtsi
@existed-node-label{
                       compatible = "nxp, imx7", "nxp, imx6";
                       new_property = new_property_value;
}
```

* Raspberry PI版本[5] (未验证)：树莓派不是使用`&+label-name`，而是`/+node-name`对已存在节点属性进行覆盖和追加，如Code-6，需要注意的是，例子中虽然将原始节点定义和重载节点定义写在了相同文件中，但遵从的原则都是`later values override earlier ones`.

```
//Code-6：Raspberry PI overwriting and appending properties
/dts-v1/;
/include/ "common.dtsi";

/ {
    ......
    
    node2 {
        an-empty-property;
        a-cell-property = <1 2 3 4>; /* each number (cell) is a uint32 */
        child-node1 {
                        my-cousin = <&cousin>;
        };
    };
};

/node2 {
            another-property-for-node2;  //appending
            a-cell-property = <6 6 6 6>; //overwirting
};
```

2.Overwriting node

与节点内属性重载类似的，还有`node`重载，但作者个人觉得这个操作没什么必要，因为节点重载的目的无非是对已有节点属性的"增、删、改"，而这些都不难完成. 不知节点重载的意义是什么，作者也只在`NXP`[4]文档中看到了相关内容，留此以作完整性记录.

```
//Code-7: Overwriting node
&iomuxc {
    vf610-colibri {
        //pinctrl_uart2 is overwritten node.
        pinctrl_uart2: uart2grp {     
            fsl,pins = <
                VF610_PAD_PTD0__UART2_TX   0x21a2
                VF610_PAD_PTD1__UART2_RX   0x21a1
            >;
        };
    ...
    };
};
```

3....

**====说明====**

    1). 由于设备树尚未标准化，所以文中内容并不具备普适性，仅作为参考，各位看官还是要依据手上硬件找对应设备商的文档； 
    
    2). 文中内容来源杂乱，作者难免有疏漏，如果文中内容描述有误或不恰当，请路过的小伙伴指正；
    
    3). 本文将长期维护，会不断添加新内容，并对原文作必要修改，也欢迎有推荐内容的小伙伴，mail作者.

#### Reference
[1] T. Petazzoni. Device Tree for Dummies. [Online]: https://events.static.linuxfound.org/sites/events/files/slides/petazzoni-device-tree-dummies.pdf

[2] Toradex AG. Device Tree Customization. [Online]: https://developer.toradex.com/device-tree-customization

[3] Devicetree.org. Devicetree Specification (rev0.2). [Online]: https://www.devicetree.org/specifications/

[4] NXP. Introduction to Device Trees. [Online]: https://www.nxp.com/docs/en/application-note/AN5125.pdf

[5] Raspberry PI Foudation. DEVICE TREES, OVERLAYS, AND PARAMETERS. [Online]: https://www.raspberrypi.org/documentation/configuration/device-tree.md

[6] Power.org. Embedded Power Architecture™ Platform Requirements(ePAPR).[Online]: https://elinux.org/images/c/cf/Power_ePAPR_APPROVED_v1.1.pdf

[7] eLinux.org. Device Tree Usage. [Online]: https://elinux.org/Device_Tree_Usage
