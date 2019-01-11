---
layout:     post
title:      NVDLA HW Source Code Analysis
subtitle:   nv_small
date:       2018-01-11
author:     Max
header-img: img/post-gray-background.jpg
catalog: true
tags:
    - NVDLA
    - nv_small
    - Code_Analysis
---

### I. Preface

断断续续的看了两个月的源码，有了一些收获，在这里与小伙伴一起分享. 对于源码的学习，作者并未以全部代码分析搞懂为目标，那样代价太高也没必要，而是分成两个学习阶段，第一阶段，分析整个加速核的逻辑划分和组织结构，研究背后的微架构设计逻辑；分析CNN算法分解与硬件映射，研究功能分布与模块划分的背后目的；第二个阶段，根据实际应用需求，在第一阶段的基础上快速定位功能关联逻辑，进行定制化修改或分析感兴趣模块的实现逻辑.

本文将介绍一些个人认为有价值，可应用于我们自己工程中的逻辑，这部分内容将逐步添加.

### II. Top Partitions Relationship 

相比Full/Large版本，Small版本阉割掉了一些功能，如Rubik，Winograd，BDMA模块等，具体可查看small spec，顶层包括以下5部分:

* partition_o：管理与外部MCU及存储器的交互接口，包括

  - csb：通过APB3.0协议与MCU交互读写NVDLA内部各功能子核的配置寄存器（状态）;
  
  - cfgrom：内部维护了硬件版本号和该版本号下各功能子核的有效配置信息，软件侧驱动文件通过读取该ROM信息确定待配置硬件版本 (Small/Full/Large) 及不同版本下各子核的可配置参数范围;
  
  - mcif：NVDLA侧访问外部DDR的AXI控制接口（small版本无SRAM接口），即所有内部需要访问DDR的子核都通过该接口进行交互;
  
  - pdp与cdp功能子核，实现Channel方向的LNR和pooling，参考[Unit Description](http://nvdla.org/hw/v1/ias/unit_description.html#planar-data-processor);
  
  - glb：管理输出到MCU的中断控制接口，即所有内部子核都通过该接口输出.
  
* partition_c：管理卷积核的CDMA, CBUF, CSC, 各部分的功能实现参考[Unit Description](http://nvdla.org/hw/v1/ias/unit_description.html#convolution-dma).

* partition_m：管理卷积核的MAC，之所以分为cmac_a和cmac_b两部分，按照官方说法是为了后端PR的优化.

* partition_a：管理卷积核的累加逻辑.

* partition_p：管理生成特征图前的非线性函数计算逻辑.

各Partition之间关联性及与MCU间的通信可参见作者画的[Visio流程图](https://github.com/VVViy/VVViy.github.io/tree/master/flowchart)，简单总结NVDLA的工作流程：MCU--->CSB--->CFGROM，读取硬件版本和配置信息; MCU--->CSB--->SUB-CORES, 按照CNN计算逻辑配置各功能子核RF；SUB-CORES--->CSB--->MCU，各子核完成本次Load后，中断MCU并进行下一轮计算，Fused mode由MCU协调各子核之间的Hazard.

### III. Several Key Blocks

#### 1. HW Configuration

* Ping-pong buffer

* Interrupt

#### 2. DMA Techniques

* Using FIFO to relieve upstream optimization

* Adding a skid buffer to aid flow control  

#### 3. Convolution Solver Logic（DC Mode）

* Convolution workflow

* CDMA workflow

**====Tips====**

    后续应该会添加定点数与浮点数融合datapath，MAC使用效率，内核同步逻辑和Winograd分析对比.
