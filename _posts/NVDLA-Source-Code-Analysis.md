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
    - HW
    - Code_analysis
---

### I. Preface

断断续续的看了两个月的源码，有了一些收获，在这里与小伙伴一起分享. 对于源码的学习，作者并未以全部代码分析搞懂为目标，那样代价太高也没必要，而是分成两个学习阶段，第一阶段，分析整个加速核的逻辑划分和组织结构，研究背后的微架构设计逻辑；分析CNN算法分解与硬件映射，研究功能分布与模块划分的背后目的；第二个阶段，根据实际应用需求，在第一阶段的基础上快速定位功能关联逻辑，进行定制化修改或分析感兴趣模块的实现逻辑.

本文将介绍一些个人认为有价值，可应用于我们个人工程中的逻辑，会逐步添加新的分析内容.

### II. Top Partitions Relationship 

相比full版本，small版本阉割掉了一些功能，像rubik，winograd等，顶层包括以下5部分:

* partition_o

* partition_c

* partition_m

* partition_a

* partition_p

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

    后续应该还会添加定点数与浮点数融合datapath，MAC使用效率，内核同步逻辑和Winograd分析对比.
