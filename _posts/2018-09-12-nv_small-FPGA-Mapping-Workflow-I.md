
---
layout:     post
title:      nv_small FPGA Mapping Workflow
subtitle:   Part I-Vivado project
date:       2018-09-12
author:     Max
header-img: img/post-gray-background.jpg
catalog: true
tags:
    - NVDLA
    - nv_small
    - FPGA
---

### Preface
  本文将简要介绍映射`nvdla nv_small`版本到FPGA过程中可能遇到的‘坑’，旨在给做相同工作的小伙伴一个参考，不作指导性建议. 
  
  本文会尽可能完整地描述从官方源码构建`vivado IP`工程到最终`BD`工程网表生成的全部步骤以及每个步骤中容易出错的地方。但需要**==说明==**的是，该映射工作是本人公司项目的一部分，所以，受规则所限，文中不会直接展示官方开源内容以外的设计内容，望谅解.
  
 
### Development environment setup
* Win10: build vivado project for HW
* Vmware Ubuntu 16.04: host for petalinux project
* Vivado: 2017.4
* Petalinux: 2017.4, Linux kernel v4.9.

  (不同版本`petalinux`的`linux kernel`版本不同，从而`DMA API`不同，导致对`nvdla/sw/kmd`驱动源码的修改方式有些区别)
  
* Board: Xilinx zcu102 rev1.0

### Step 1: Build Tree and vmod (Linux)
官方HW工程是通过在`branch/spec/defs/`下定义不同的`spec`对同一源码构建不同的行为模型架构，所以，源码内部有很多的`c++`和`perl`相关的条件编译。因此，在搭建`vivado`工程前，要先按照官方[NVDLA Environment Setup Guide](http://nvdla.org/hw/v2/environment_setup_guide.html)将`nv_small/vmod/nvdla`编译为纯RTL source code.

**====Tips====** 

    1). build tree所需的工具中，cpp，gcc，g++，perl，python采用linux系统默认，java需要额外安装，若只是
        build RTL，其他tool可直接enter.
        
    2). 在build RTL过程还需要perl module的支持，可以直接使用类似pip的perl CPAN安装错误信息中指定的module，
        如

```perl
$ perl -MCPAN -e shell

...

CPAN > install YAML

...

CPAN > exit
```

成功`[TMAKE]:DONE`后，在目录下会生成一个`outdir`目录，目录下便是搭建`vivado`所需的全部`RTL code`.


### Step 2: Create RTL Project (Win10)
1.添加源文件，可按照`nv_small/spec/defs/nv_small.spec`描述来指定`vmod/nvdla/`目录下待添加的内核源文件————small版本不包含`bdma`，`retiming`和`rubik`内核(文件夹)以及其他文件夹中的部分`.v`————实际中，可先添加`top`文件下的`modules`. 之后，`vivado`显示的hierarchy中缺少什么文件，加之即可;

2.添加`RAM`文件时，要选择`outdir/nv_small/vmod/rams/fpga/small_rams/`文件夹下的`RAM`源文件;

**====Tips====** 

    1). Xilinx RAM资源的使用有4种基本方式————flexibility依次递减————i)源码推断, ii)XPMs, iii)直接例化
        8k/16k RAM primitives, iv)例化RAM IP.
        
    2). 待全部缺失文件补全后，可能会发现在NV_nvdla hierarchy以外，还存在一些modules，这个是因为一个文
        件中定义了多个modules，可以将未用到的modules comment掉.
        
3.关闭`clock gating`，原设计对`RAM`存储op使用了大量`clock gating`以降低功耗，但与`processor，ASIC`不同，FPGA的时钟树是设计好的，`clock buf`资源有限，若不关闭`gating`，可能产生很大`skew`(之前因为部分`gating`未关闭，测试一直不过). 使用到的`clock gating`开关宏包括以下4个，有4种设置方式：i）在vivado的`Project Settings`->`General`->`Verilog options`->`Defines`中添加宏名称和值；ii）定义`.vh`头文件，define宏，右键该头文件选择`Global Include`；iii）定义`.vh`，之后在vivado的`Project Settings`->`General`->`Verilog options`->`Verilog Include Files Search Paths`中选择头文件的路径；iv）前三种方式都不需要将头文件在各源文件中`include`，最后一种是笨方法，即将头文件`include`到所有相关的`.v`源文件中，大家自行选择.

- VLIB_BYPASS_POWER_CG
- NV_FPGA_FIFOGEN
- FIFOGEN_MASTER_CLK_GATING_DISABLED
- FPGA

**====Tips====** 

    1). 实际上，也可以不关闭gating信号逻辑，可使用syth选项支持相关逻辑的clock gating，clk buf会增加, 作者测试了
        部分信号, 待测试全部信号的资源消耗和逻辑稳定性.
        
    2). nvdla_pwrbus_ram_*_pd相关逻辑，可以转化使用BRAM的sleep达到相同效果，待测试.

4.[optional] 之后可在`NV_nvdla`里添加generated时钟，进行综合~~及布局布线~~查看资源开销、功耗和时钟频率等信息.

**====Tips====**

        之前的描述可能对一些小伙伴产生了误导，导致几个人都mail我问综合之后IO资源超标，无法进行PR的问题. 
        这里需要解释一下，整个nv_small加速核与外部有两个通信接口，AXI总线接口用于与DDR控制器交互读写数据，
        APB接口与MCU交互配置nvdla内部各子核的寄存器，前者的接口数量很大，后者接口只有少量几个，如果直
        接综合或PR，相当于将这些接口连接到了FPGA的引脚上，所以IO的资源会超标，这里有两种基本解决方式，
        i）使用内置处理器的MPSoC FPGA芯片，这样处于PL部分的nvdla接口将直接连接到PS部分的MCU上，并由
        MCU与DDR控制器通信读写片外DDR；ii）选择IO引脚数量大的FPGA芯片，通过AXI chip-to-chip与MCU通信.

### Step 3: Packaging NVDLA IP (Win10)
1.添加wrapper，如果在`NV_nvdla`里例化了`generated clock`，请删除，另外，为了在`block design`中连接`PS`的`AXI master`和`AXI slave`接口，需要在当前工程结构下，增加一个NV_nvdla_wrapper module封装`NV_nvdla`和`NV_NVDLA_apb2csb` modules;

**====Tips====** 
        
        也可以将axi apb bridge IP同时封装在wrapper中，这样在BD工程中，PS->nvdla IP方向通信就变成了
        axi master->axi slave接口互连映射，而非本文采用的axi master->axi apb bridge->apb slave互连模式. 
        
2.补全`AXI`与`APB`信号，`NVDLA`使用`AXI`和`APB`协议与`PS`通信，但源码中缺失了部分`AXI`协议信号，需按照`AMBA AXI and ACE Protocol Spec`和`AMBA 3 APB Protocol spec`补全缺失的`AXI, APB`信号，即

```
//append axi signal to NV_nvdla_wrapper signal list
output  [2 : 0]   M_AXI_AWSIZE,
output  [1 : 0]   M_AXI_AWBURST,
output            M_AXI_AWLOCK,
output  [3 : 0]   M_AXI_AWCACHE,
output  [2 : 0]   M_AXI_AWPROT,
output  [3 : 0]   M_AXI_AWQOS,
output            M_AXI_AWUSER,
output            M_AXI_WUSER,
input   [1 : 0]   M_AXI_BRESP,
input             M_AXI_BUSER,
output  [2 : 0]   M_AXI_ARSIZE,
output  [1 : 0]   M_AXI_ARBURST,
output            M_AXI_ARLOCK,
output  [3 : 0]   M_AXI_ARCACHE,
output  [2 : 0]   M_AXI_ARPROT,
output  [3 : 0]   M_AXI_ARQOS,
output            M_AXI_ARUSER,
input   [1 : 0]   M_AXI_RRESP,
input             M_AXI_RUSER   

//append apb signal to NV_nvdla_wrapper signal list
output            S_APB_PSLVERR,
```

并将相应信号分别添加到`NV_nvdla`和`NV_NVDLA_apb2csb`的信号列表中，并分别添加如下赋值代码，

```     
//add these code to NV_nvdla module
assign m_axi_awsize  = 3;
assign m_axi_awburst = 2'b01;
assign m_axi_awlock  = 1'b0;
assign m_axi_awcache = 4'b0010;
assign m_axi_awprot  = 3'h0;
assign m_axi_awqos   = 4'h0;
assign m_axi_awuser  = 'b1;
assign m_axi_wuser   = 'b0;
assign m_axi_arsize  = 3;
assign m_axi_arburst = 2'b01;
assign m_axi_arlock  = 1'b0;
assign m_axi_arcache = 4'b0010;
assign m_axi_arprot  = 3'h0;
assign m_axi_arqos   = 4'h0;
assign m_axi_aruser  = 'b1;

//add below code to NV_NVDLA_apb2csb module
assign pslverr = 1'b0;
```

**====Tips====**
        
        添加上述信号后，其实已经能够在BD工程中完成interface连接任务了，但是仅作上述修改，BD工程中必须要使用AXI
        smartconnect IP做PL与PS的接口互连，如果要使用AXI interconnect IP或干脆直接将NVDLA master与PS slave互连，
        就要进一步修改如下信号位宽, 否则，BD工程在validate时会报错.

```
//modify following signal bit width in NV_nvdla module
                
//input [7:0]     nvdla_core2dbb_b_bid;
input [5:0]       nvdla_core2dbb_b_bid;
//input [7:0]     nvdla_core2dbb_r_rid;
input [5:0]       nvdla_core2dbb_r_rid;
//output [7:0]    nvdla_core2dbb_aw_awid;
output [5:0]      nvdla_core2dbb_aw_awid;
//output [3:0]    nvdla_core2dbb_aw_awlen;
output [7:0]      nvdla_core2dbb_aw_awlen;
//output [7:0]    nvdla_core2dbb_ar_arid;
output [5:0]      nvdla_core2dbb_ar_arid;
//output [3:0]    nvdla_core2dbb_ar_arlen;
output [7:0]      nvdla_core2dbb_ar_arlen;
```

实际上，所有的*_id signal只有后4位work. 另外，不要忘了在`NV_nvdla` module的`NV_NVDLA_partition_o u_partition_o{...}`实例中修改相应信号位宽，这里就不写了.

3.添加`xdc`，新建两个`xdc`文件，一个为`nvdla IP`在`OOC`综合时使用，另一个则是在`global syth`时使用，`OOC xdc`可以直接约束两个primary clk，如

```
create_clock -period 10.001 -name u_dla_core_clk [get_ports u_dla_core_clk];
create_clock -period 10.001 -name u_dla_sys_clk [get_ports u_dla_sys_clk];
```

另一个`xdc`保持空白即可，这里涉及到了一个`xdc scope`问题，感兴趣的可以参考`xilinx UG903`. 除此之外，在`source`窗口选中`OOC`版本的`xdc`，并在属性窗口中，为`USED_IN`属性添加`out-of-context`选项，否则，`BD`综合时将多出两个主时钟. 选中另一版本`xdc`，在属性窗口中为`PROCESSING_ORDER`属性选择`LATE`, 这样，如果后期在该约束文件中为`nvdla IP`添加新约束，那么`BD`工程综合时不会与全局约束冲突.

4.封装`nvdla IP`，`Tools-->Create and Package New IP-->Package your current project`, next and finish;
        
5.`AXI master` interface推断，点击`Ports and Interfaces`，如Fig-1, 查看信号列表中是否存在自动推断出的`AXI master`接口，若没有，则在信号列表中选中全部`master`接口信号，右键选择`Auto Infer Interface`，推断出`master`接口信号后，需要检查位宽是否与源文件中声明的相一致(之前出现过标量矢量化的情况，工具坑)，即`Size Left`，`Size Right`. 另外，检查`Driver Value`，如Fig-2, 若官方源码中声明的信号在此列表中显示驱动强度为`0`，则选中该信号，在属性窗口删除`0`值，另外，需要对上面我们后添加的`master`信号，将驱动强度设置为`0`；
        
<div align="center">
        
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%231-%231.JPG?raw=true" />

Fig-1

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%231-%232.JPG?raw=true" />

Fig-2

</div>

6.`APB slave` interface推断,这个接口工具不会自动推断，需要选中全部`APB`信号，右键自动推断，在弹出窗口中依次选择`Advanced`-->`apb_rtl`. 在推断出`AXI`和`APB`信号后，分别右键两个接口信号，选择`Associate Clocks`，分别关联`AXI master`-->`*_core_clk`和`APB slave`-->`*_csb_clk`；
        
7.`APB memory map`, `AXI/APB master-slave`接口是通过`memory-map`机制做数据映射的，作者画了一个映射结构简图Fig-3. 不同于`AXI master memory block`的自动生成. `APB memory block`需要自行添加，选择`Addressing and Memory`-->`Memory Maps(for slaves)`，右键`IP Addressing and Memory Wizard`, 弹出窗口中选择`APB`接口信号，继续右键`Add Address Block`(因为一块连续地址，一个block便可)，弹出窗口键入`reg`，如Fig-4；

<div align="center">
        
<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%231-%233.JPG?raw=true" />

Fig-3

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%231-%234.JPG?raw=true" />

Fig-4

</div>

8.`Review and Package` -->`Package IP`.


### Step 4: Create Block Design Project (Win10)
1.新建RTL工程，`Settings`-->`IP`-->`Repository`将刚刚封装的`nvdla IP` (nvdla_ip_prj_name.srcs)添加到IP列表，新建`BD`工程, `Flow Navigator`-->`Create Block Design`；
        
2.添加`ps，nvdla ip，axi apb bridge，axi interconnect`等IP，`ps`要配置`AXI master，AXI slave`以及`pl_ps_irq`中断接口，之后连接接口即可. 对于`nvdla ip`的几个接口信号————`global_clk_ovr_on, tmc2slcg_disable_clock_gating, test_mode，nvdla_pwrbus_ram_*_pd`按照官网`small`版本[Integrator’s Manual](http://nvdla.org/hw/v2/integration_guide.html#integrator-s-manual)建议，使用`Constant ip`直接拉低，BD工程结构如Fig-5；

3.地址分配，待interface连接完成后，切换到`Address Editor`页面，右键选择`Auto Assign Address`对互连接口进行地址映射，`slave`接口保留`64k`即可(官网已指明`small`版本的[Address space layout](http://nvdla.org/hw/v2/scalability.html#address-space-layout)所占空间为`56KB`,所以这里分配`64k`足矣). 之后切换回`Diagram`窗口，`validate`.

4.添加`xdc`，综合、布局布线和输出`bit`文件，之后`export hardware`（复选`include bitstream`）.

<div align="center">

<img src="https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%231-%235.jpg?raw=true" />

Fig-5

</div>

### Step 5: [optional] Run Test Sets
Vivado+SDK/VIP/HW manager, 请自行选择，我没做 :sweat_smile:.

---
