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

断断续续的看了两个多月的源码，有了一些收获，在这里与小伙伴一起分享. 对于源码的学习，作者并未以全部代码分析搞懂为目标，那样代价太高也没必要，而是分成两个学习阶段，第一阶段，分析整个加速核的逻辑划分和组织结构，研究背后的微架构设计逻辑；分析CNN算法分解与硬件映射，研究功能分布与模块划分的背后目的；第二个阶段，根据实际应用需求，在第一阶段的基础上快速定位功能关联逻辑，进行定制化修改或分析感兴趣模块的实现逻辑.

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
这部分介绍一些有价值的模块，以及一些与官网Full版本描述文档有出入逻辑，随着分析的加深，会逐步添加分析结果.

#### 1. HW Configuration
NVDLA内部各功能子核都通过`ping-pong buffer`进行参数设置，包括卷积核(不包括其中的CBUF和CMAC)，SDP，PDP，CDP.

* Ping-pong buffer

  [框图](http://nvdla.org/hw/v1/hwarch.html#ping-pong-synchronization-mechanism)及配置逻辑如Fig-1~Fig-2所示，其中`Single Register Group`充当了Controller的角色（\*\_single_reg modules），其他两个`Register Group`是实际的RF（\*\_dual_reg modules），入口由MCU通过CSB进行参数配置，出口则与功能子核内部数据通路连接进行各module内部寄存器参数配置（reg2dp\_\* signals），数据通路相关模块同时会将配置后的状态信息反馈给`Register Group/Single Register Group`（dp2reg\_\* signals），这些反馈信息会被打包反馈给MCU.
  
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%2312.jpg?raw=true">

Fig-1. Ping-pong block diagram
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%231.jpg?raw=true" />

Fig-2. Ping-pong configuration flow
    
</div>    

* Interrupt

NVDLA的中断控制由`GLB`模块负责管理，其中`NV_NVDLA_GLB_csb, NV_NVDLA_GLB_CSB_reg`负责接收MCU中断配置信息，并将这些包含`mask, trigger, set, clear`等信息转发给中断控制模块`NV_NVDLA_GLB_ic`，控制模块一方面根据MCU设置的`\*\_mask signals`和功能模块输入的中断请求信号`\*\_done_statuts(i)`生成**1 bit**中断信号，由NVDLA中断线发出MCU，但MCU只知道NVDLA中断，但不知道是内部哪个Block发出的，所以控制模块还要把各功能模块的`done status`信息回传给负责与CSB交互的模块，由交互模块将中断相关的`mask, set, status`信息打包回传MCU，这里回传的set信号是硬件侧的设置状态信号，始终处于拉低状态.

#### 2. DMA Techniques
NVDLA内部卷积，激活函数，池化，局部规则化等功能模块都有DMA需求，这里我们研究一下NVDLA DMA接口的设计考量. 

1）`Read DMA`由数据请求和数据响应双通道构成，如Fig-3，其中，请求通道向MCIF路由控制器发送读DDR地址和存储块尺寸信息，一旦MCIF响应当前请求，会反馈`Ready`信号，之后，读取DDR并准备好数据后，通过响应通道发送数据信息给DMA. 请求通道与响应通道面板由`FIFO and SKID Buffer`两部分逻辑构成. 参考`vmod/nocif/NV_NVDLA_DMAIF_rdreq, vmod/nocif/NV_NVDLA_DMAIF_rdrsp`

2）`Write DMA`则只有写请求单通道，功能模块准备好写DDR数据后，会直接将写数据请求和握手信号发送至MCIF路由控制器，MCIF响应后会反馈`Ready`信号，写请求通道`FIFO and SKID buffer`两部分构成，写请求面板同样由`FIFO and SKID Buffer`两部分构成. 参考`vmod/nocif/NV_NVDLA_DMAIF_wr`

<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%234.jpg?raw=true" />

Fig-3. NVDLA RMDA

</div>

* Using FIFO to relieve upstream optimization

  在DMA中使用FIFO具有有效缓解相邻模块间的吞吐量不一致和跨时钟的多bit信号同步，实际上，还有很多设计上的优势. 如Fig-4所示，Sender向Receiver发送数据，Receiver接收到数据后，反馈Ready信号，这形成了一个`Long Loop`回路，这种设计很难跑高频时钟，因为走线延迟可能导致一个时钟内Ready信号无法到达Sender端，造成延迟传播，使上游逻辑出错，而且对于回路，很难插入流水或使用retiming对上游组合逻辑进行优化.   
  
  这时，可以在下游Receiver前插入一个FIFO，将`Long Loop`打断，将回路压缩到FIFO与Receiver间，如Fig-5所示，这一方面释放了上游电路对下游电路控制信号的依赖，可以使用流水线，retiming等技术对上游逻辑进行优化，另一方面，下游短回路能够跑高频，且提高了下游数据请求的连续性. 
  
  然而，所有基于Buffer的通信，都要为`Flow Control`提供`Backpressure`，即缓存即将达到写满状态或下游逻辑出现需要暂停缓存写入的中断请求，Buffer的控制通路就需要通知Sender停止发送数据. 这时，会带来两个问题，Buffer反馈的控制信号Ready依然存在走线超时的风险，其次，如果对控制信号进行Pipeline，会降低DMA的性能，因为一旦数据传输被Stall，重新开始传输会有固定时钟延迟. NVDLA对于Control path的处理是使用SKID Buffer技术.
  
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%232.jpg?raw=true" />

Fig-4. Long loop data transfer


<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%233.jpg?raw=true" />

Fig-5. Short loop data transfer

</div>
 
* Adding a skid buffer to aid flow control  
  
  
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%235.jpg?raw=true" />

Fig-6. SKID Bufer


<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%236.jpg?raw=true" />

Fig-7. Control flow with FIFO status signals

</div>

#### 3. Convolution Solver Logic（DC Mode）


<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%237.jpg?raw=true" />

Fig-8. Convolution pipeline

</div>

* Convolution workflow


<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%239.jpg?raw=true" />

Fig-9. 

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%2310.jpg?raw=true" />

Fig-10.

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%2311.jpg?raw=true" />

Fig-11.

</div>

* CDMA workflow

<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%238.jpg?raw=true" />

Fig-12. nv_small CDMA

</div>

**====Tips====**

    后续应该会添加定点数与浮点数融合datapath，MAC使用效率，内核同步逻辑和Winograd分析对比.
