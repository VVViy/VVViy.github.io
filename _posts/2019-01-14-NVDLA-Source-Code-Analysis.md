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

断断续续的看了两个多月的源码，有了一些收获，在这里与小伙伴一起分享. 对于源码的学习，作者并未以全部代码分析搞懂为目标，那样代价太高也没必要，而是分成两个学习阶段，第一阶段，分析整个加速核的逻辑划分和组织架构，研究背后的微架构设计逻辑；分析CNN算法分解与硬件映射，研究功能分布与模块划分的背后目的；第二个阶段，根据实际应用需求，在第一阶段的基础上快速定位功能关联逻辑，进行定制化修改或分析感兴趣模块的实现逻辑.

### II. Top Partitions Relationship 

相比Full/Large版本，Small版本阉割掉了一些功能，如Rubik，Winograd，BDMA模块等，具体可查看small spec，顶层包括以下5部分:

* Partition_o：管理与外部MCU及存储器的交互接口，包括

  - CSB：通过APB3.0协议与MCU交互读写NVDLA内部各功能子核的配置寄存器;
  
  - CFGROM：内部维护了硬件版本号和该版本号下各功能子核的有效配置信息，软件侧驱动文件通过读取该ROM信息确定待配置硬件版本 (Small/Full/Large) 及不同版本下各子核的可配置参数范围;
  
  - MCIF：NVDLA侧访问外部DDR的AXI控制接口（small版本无SRAM接口），即所有内部需要访问DDR的子核都通过该接口与DDR Controller进行交互;
  
  - PDP与CDP功能子核，实现Pooling和Channel方向的LNR，参考[Unit Description](http://nvdla.org/hw/v1/ias/unit_description.html#planar-data-processor);
  
  - GLB：管理输出到MCU的中断控制面板，即所有内部子核都通过该接口输出中断信号.
  
* Partition_c：管理卷积核的CDMA, CBUF, CSC, 各部分的功能实现参考[Unit Description](http://nvdla.org/hw/v1/ias/unit_description.html#convolution-dma).

* Partition_m：管理卷积核的MAC，之所以分为CMAC_a和CMAC_b两部分，按照官方说法是为了后端PR的优化.

* Partition_a：管理卷积核的累加逻辑.

* Partition_p：管理生成特征图前的激活函数计算逻辑.

各Partition之间关联性及与MCU间的通信可参见作者画的[Visio流程图](https://github.com/VVViy/VVViy.github.io/tree/master/flowchart)，简单总结NVDLA的工作流程：MCU--->CSB--->CFGROM，读取硬件版本和配置信息; MCU--->CSB--->SUB-CORES, 按照CNN计算逻辑配置各功能子核RF；SUB-CORES--->CSB--->MCU，各子核完成本次Load后，中断MCU并进行下一轮计算，Fused mode由MCU协调各子核之间的Hazard.

### III. Several Key Blocks
这部分介绍一些有价值的模块，以及一些与官网Full版本描述文档有出入的逻辑，逐步添加.

#### 1. HW Configuration
NVDLA内部各功能子核都通过`ping-pong buffer`进行参数设置，包括卷积核(不包括其中的CBUF和CMAC)，SDP，PDP，CDP.

* Ping-pong buffer

  [框图](http://nvdla.org/hw/v1/hwarch.html#ping-pong-synchronization-mechanism)及配置逻辑如Fig-1~Fig-2所示，其中`Single Register Group`充当了Controller的角色（\*\_single_reg modules），其他两个`Register Group`是实际的RF（\*\_dual_reg modules），入口由MCU通过CSB进行参数配置，出口则与功能子核内部数据通路连接进行各Module内部寄存器参数配置（reg2dp\_\* signals），数据通路相关模块同时会将配置后的状态信息反馈给`Register Group/Single Register Group`（dp2reg\_\* signals），这些反馈信息会被打包反馈给MCU.
  
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%2312.jpg?raw=true">

Fig-1. Ping-pong block diagram
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%231.jpg?raw=true" />

Fig-2. Ping-pong configuration flow
    
</div>    

* Interrupt

NVDLA的中断控制由`GLB`模块负责管理，其中`NV_NVDLA_GLB_csb, NV_NVDLA_GLB_CSB_reg`负责接收MCU中断配置信息，并将这些包含`mask, trigger, set, clear`等信息转发给中断控制模块`NV_NVDLA_GLB_ic`，控制模块一方面根据MCU设置的`*_mask signals`和功能模块输入的中断请求信号`*_done_statuts(i)`生成**1 bit**中断信号，由NVDLA中断接口Trigger MCU，但MCU只知道NVDLA中断，却不知道是内部哪个Block发出的，所以控制模块还要把各功能模块的`done status`信息回传给负责与CSB交互的模块，由交互模块将中断相关的`mask, set, status`信息打包回传MCU，这里回传的`set`信号是硬件侧的设置状态信号，始终处于拉低状态.

#### 2. DMA Techniques
NVDLA内部卷积，激活函数，池化，局部规则化等功能模块都有DMA需求，这里我们研究一下NVDLA DMA接口的设计考量. 

1）`Read DMA`由数据请求和数据响应双通道构成，如Fig-3，其中，请求通道向MCIF路由控制器发送读DDR地址和存储块尺寸信息，一旦MCIF响应当前请求，会反馈`Ready`信号，之后，读取DDR并准备好数据后，通过响应通道发送数据信息给DMA. 请求通道与响应通道面板由`FIFO and SKID Buffer`两部分逻辑构成. 参考`vmod/nocif/NV_NVDLA_DMAIF_rdreq, vmod/nocif/NV_NVDLA_DMAIF_rdrsp`

2）`Write DMA`则只有写请求单通道，功能模块准备好写DDR数据后，会直接将数据和握手信号发送至MCIF路由控制器，MCIF响应后会反馈`Ready`信号，写请求面板同样由`FIFO and SKID Buffer`两部分构成. 参考`vmod/nocif/NV_NVDLA_DMAIF_wr`

<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%234.jpg?raw=true" />

Fig-3. NVDLA RMDA

</div>

* Using FIFO to relieve upstream optimization

  在DMA中使用FIFO具有缓解相邻模块吞吐量不一致和跨时钟多bit信号同步的作用，实际上，还有很多设计上的优势. 如Fig-4所示，Sender向Receiver发送数据，Receiver接收到数据后，反馈Ready信号，这形成了一个`Long Loop`回路，这种设计很难跑高频时钟，因为走线延迟可能导致一个时钟内Ready信号无法到达Sender端，造成延迟传播，使上游逻辑出错，而且对于回路，很难插入流水或使用Retiming对上游组合逻辑进行优化.   
  
  在下游Receiver前插入一个FIFO，将`Long Loop`打断，把回路压缩到FIFO与Receiver间，如Fig-5所示，这一方面解除了上游电路对下游电路控制信号的依赖，可以对上游电路应用流水线，Retiming等优化技术，另一方面，下游短回路时时序更易收敛，且提高了下游数据请求的连续性. 
  
  然而，所有基于Buffer的通信，都要为`Flow Control`提供`Backpressure`，即缓存即将达到写满状态或下游逻辑出现需要暂停缓存写入的中断请求，Buffer的控制通路就需要通知Sender停止发送数据. 这时，会带来两个问题，Buffer反馈的控制信号Ready依然存在走线超时的风险，其次，如果对控制信号进行Pipeline，会降低DMA的性能，因为一旦数据传输被Stall，重新开始传输会有固定时钟延迟. NVDLA对Control Path的处理则通过SKID Buffer技术实现.
  
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%232.jpg?raw=true" />

Fig-4. Long loop data transfer


<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%233.jpg?raw=true" />

Fig-5. Short loop data transfer

</div>
 
* Adding a skid buffer to aid flow control  

  `nvdla/hw/vmod/plugins`下的`pipe.pm`是SKID Buffer生成器，里面简单描述了SKID Buffer在NVDLA中的作用，即在Ready信号上加了一级Pipeline，从而"Make ro/ri timing clean". 一般情况下，基于握手协议的数据传输会使用如Fig-6所示的`Basic Pipeline`结构，对于Ready信号，因为缺少Reg的中继，所以很难在高频下时序收敛. 那么最直接的想法是为Ready信号加一级Pipeline，出于Pipeline Balance，同时要在Data和Valid路径上插入一级流水，如Fig-7. 然而，Ready信号时序问题虽得到了处理，但基于这种逻辑的DMA传输效率会降低，因为每次Stall后都要牺牲一个时钟恢复重传，即
  
```  
clock     j-1(stall)       j      ...       k(recover)       k+1               k+2
-------------------------------------------------------------------------------------------------------------
signal

RVR       1 for jth        1 for jth        1 for jth        1 for (j+1)th     1 for (j+1)th <----waste 1 clk

RVL       1 for (j+1)th    1 for (j+1)th    1 for (j+1)th    1 for (j+1)th     1 for (j+2)th

R0        0                0                1                1                 1

RR0       1                0                0                1                 1
```
Note: RVR表示Fig-7右侧Valid信号reg，RVL表示左侧Valid信号，R0表示m_ready, RR0表示Ready信号reg.
  
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%2313.jpg?raw=true">

Fig-6. Basic pipe

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%2314.jpg?raw=true">

Fig-7. Basic pipe with 1 pipeline

</div>

下面来研究一下SKID Buffer，看它是否能同时解决控制信号时序和DMA传输效率问题，这里我们仍以`RDMA req`信道为例，根据`NV_NVDLA_DMAIF_rdreq`可以很容易描绘出SKID Buffer原理图，如Fig-8所示，可以看到"pipeline 1"同样是一个"Basic pipe"，但是上面使用了一级"Bypass queue"构成了一级SKID Buffer，时序问题显然由插入的一级Pipeline解决了(注意：SKID Buffer可以叠加使用)，下图简单分析了传输效率问题，即使用旁路解决数据延迟. 

```
clock     j-1(stall)       j      ...       k(recover)       k+1               k+2
-------------------------------------------------------------------------------------------------------------
signal

RV1       1 for jth        1 for jth        1 for jth        1 for (j+1)th     1 for (j+2)th  <----Bypass works!

RV0       1 for (j+1)th    1 for (j+1)th    1 for (j+1)th    1 for (j+1)th     1 for (j+2)th

Ri        0                0                1                1                 1

RR0       1                0                0                1                 1

Ro        1                0                0                1                 1            
```

<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%235.jpg?raw=true" />

Fig-8. SKID bufer

</div>

总之，SKID Buffer是解决基于Buffer传输中Control Path时序和效率问题的有效方式，可以应用于我们自己的DMA设计中. 实际上，还有一种同类解决方法，也是Intel-Altera推荐的一种方式，如Fig-9所示，其是一种利用FIFO Almost Full状态信号配合多级流水的控制结构. 众知，FIFO的状态包括Full/Almost Full/Half Full，Empty/Almost Empty/Half Empty，其中，Almost Full通过设置一阈值，FIFO达到该值后发送"将满"状态信号，该值越低，Control Path可插入流水线的级数越多. 假设Fig-9中FIFO的Almost Full阈值为8，那么当Sender接收到反馈的控制信号时，实际上已经发送了4个数据了，所以Sender"知道"它再发4个数据FIFO就满了，所以这是一种基于先验知识的控制方式. 但对于NVDLA，个人认为使用SKID更适合，因为MCIF需要与MCU交互，而且要占用系统总线，发生非FIFO写满的Stall可能性更大，显然利用FIFO状态信号的方式，传输效率就变低了.

这里没有介绍MCIF对内面板的设计，MCIF对外为AXI协议，对内则为[Arbiter](http://nvdla.org/hw/v1/ias/unit_description.html#sramif)，其从各功能内核DMA中选择服务对象. 这部分逻辑内容太多，在`nvdla/hw/vmod/nvdla/nocif`下除了DMA的三个接口文件，其他都是MCIF ingress/egress逻辑，所以这里就不做逻辑分析了. 但建议在研究时，可以找本NOC相关书配合着看(大牛请忽略)，实际上，完整的SoC数据通信包括两条路径:Datapath(数据通路——数据相关处理路径)和Control Path(控制通路——Valid，Ready等控制信号处理相关路径)，针对这两条通信路径，又分别由Router，Channel，Buffer三部分Device完成传输，上面介绍的SKID Buffer就是一种Channel Microarchitecture，而MCIF实际上是一个Router，路由器中Buffer和Switch是服务于Datapath的Part，而Arbiter/Allocator是服务于Control Path，MCIF对外只有一个AXI接口，所以所有的DMA需要Arbiter选择. Router/Channel/Buffer/Arbiter/Switch等Device都有不同的应用分类和Micro-architecture，所以，结合NOC背景资料，研究NVDLA数据传输特征/功能特征下MCIF的结构设计，应该会有Design Reuse的收获. （推荐两本NOC书[1~2]）

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%236.jpg?raw=true" />

Fig-9. Control flow with FIFO status signals

</div>

#### 3. Convolution Solver Logic（DC Mode）
卷积运算是CNN的主要运算逻辑，本节对官方文档的`Atomic, Stripe, Block, Channel Operations`中的基础硬件逻辑作个简要分析. Fig-10是官方文档对[Convolution](http://nvdla.org/hw/v1/ias/unit_description.html#convolution-dma)运算逻辑单元的描述，其中`CDMA`读入图像/特征图数据和卷积核权重数据，`CBUF, CMAC`没有`Ping-pong buffer`配置寄存器，主要通过`CSC`模块调度，`CBUF`缓存读入数据，并由`CMAC`进行乘累加，攒够一个`Stripe operation`结果后转发到`CACC`进行输出累加.

<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%237.jpg?raw=true" />

Fig-10. Convolution pipeline

</div>

* Convolution workflow
  
  `Atomic, Stripe, Block, Channel Operations`这些操作就是Fig-10中Convolution Pipeline要实现的逻辑，二者关系如下
  
  - CMAC内部只有**16个MAC Cell，64 Elements/MAC Cell，1 Elements=16 bit/2\*8 bit**，所以，
  - NVDLA要求卷积核在运算前要先分组，每组最多只有16 kernels (for int16/fp16)，或32 kernels (for int8)，每个kernel都是一个3D Cube，每个3D Cube又可进一步划分为一系列1\*1\*C的Small Element Cube，C表示Channel Element宽度，如CNN第一层是RGB 3 Channel，那么C = 3\*Element Width/Channel;
  - 由于64 Elements/MAC Cell的限制，所以，每个kernel要划分为一系列1\*1\*64 Elements的Small Cube，那么对于int16/fp16，每个Small Cube=1\*1\*64\*2 bytes = 128 bytes，对于int8，1 Small Cube=1\*1\*64\*1 byte = 64 bytes (但int8，每组32 kernels，即计算量一致，吞吐量int8比int16/fp16高一倍)，Data 3D Cube与权重处理一致. 所以，
  - [Atomic Operation](http://nvdla.org/hw/v1/ias/unit_description.html#atomic-operation)：原子运算就是1个Data Small Element Cube与16个kernel中相同Index的Weight Small Element Cube作乘法，在Fig-11中1个int16/fp16 MAC运算就是一个Data Small Element Cube与1个Weight Small element Cube运算，是1个Atomic Operation的"1/16 op"，对于CNN运算，因为全部kenel要对同一或几幅图像运算求特征图，所以`Data shared by all kernels`；
  - [Stripe Operation](http://nvdla.org/hw/v1/ias/unit_description.html#stripe-operation)：在Fig-12中1个Data 3D Cube中若干Small Elemment Cube与16个kernel中相同Index的MAC运算为1 Stripe运算，但该运算不是并行完成的，而是逐个Data Small Elemment Cube并行与16个Weight Small Element Cube计算；
  - [Block Operation](http://nvdla.org/hw/v1/ias/unit_description.html#block-operation)：在Fig-12中若干个Data Small Element Cube与若干个Weight Small Elememnt Cube构成了一个Block Operation，该操作是可并行计算的，结合官网介绍，1th Data Small Element Cube既是Stripe Op0的第二个元素也是Stripe Op1的首元素，但Weight Small Element Cube的Index不同，可以并行计算；
  - [Channel Operation](http://nvdla.org/hw/v1/ias/unit_description.html#channel-operation)：一般一个kernel Cube可能不会包含64个Wight Small Element Cube，所以多个Block Op是可能并行计算的；
  - 最大并行度：按照上面分解，int16/fp16的最大并行度为16\*64 Element，按照官网介绍1 Atomic Op/clk，最大吞吐量1024\*16 bit/clk，int8则为其2倍.
  
<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%2311.jpg?raw=true" />

Fig-11. One small element cube vs one kernel

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%2310.jpg?raw=true" />

Fig-12. Open 3D cube package

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%239.jpg?raw=true" height="500" width="530" />

Fig-13. One 3D data cube vs one group kernel

</div>

* CDMA workflow

  CDMA Block不作分析，放在这里是因为官网[Unit Description](http://nvdla.org/hw/v1/ias/unit_description.html#convolution-dma)是对Full/Large版本中CDMA的描述，同时，未对Small版本各Unit作描述，CDMA的正确逻辑如Fig-14所示. 
  
<div align="center">
    
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%236-%238.jpg?raw=true" />

Fig-14. nv_small CDMA

</div>

**====Tips====**

    后续应该会添加定点数与浮点数融合datapath，MAC使用效率，内核同步逻辑和Winograd分析对比.
    
#### Referece

[1] W.J. Dally and B. Towles. Principles and Practices of Interconection Networks. Morgan Kaufmann, 2004.

[2] Natalie Enright Jerger, Tushar Krishna, and Li-Shiuan Peh. On Chip Networks, Second Edition, Morgan & Claypool, 2017.
    
