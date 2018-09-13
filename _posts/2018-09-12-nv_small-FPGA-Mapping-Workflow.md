---
layout:     post
title:      nv_small FPGA Mapping Workflow
subtitle:   part I-Vivado project
date:       2018-09-12
author:     Max
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - NVDLA
    - nv_small
    - FPGA
---

### Preface
  本文将简要介绍映射nvdla nv_small版本到FPGA过程中可能遇到的‘坑’，旨在给做相同工作的小伙伴一个参考，不作指导性建议. 
  
  本文会尽可能完整地描述从官方源码构建vivado IP工程到最终BD工程网表生成的全部步骤以及每个步骤中容易出错的地方。但需要**==说明==**的是，该映射工作是本人公司项目的一部分，所以，受规则所限，文中不会直接展示官方开源内容以外的设计内容，望谅解.
  
 
### Development environment setup
* Win10: build vivado prj for HW;
* Vmware Ubuntu 16.04: host for petalinux prj;
* Vivado: 2017.4
* Petalinux: 2017.4, Linux kernel v4.9

  (不同版本petalinux的linux kernel版本不同，从而DMA API不同，导致对nvdla/sw/kmd驱动源码的修改方式有些区别).
* Board: Xilinx zcu102 rev1.0

### Step 1: Build tree and vmod (Linux)
官方HW工程是通过在branch/spec/defs/下定义不同的spec来对相同源码构建不同的行为模型架构，所以，源码内部有很多的c++和perl相关的条件编译。因此，在搭建vivado工程前，要先按照官方[NVDLA Environment Setup Guide](http://nvdla.org/hw/v2/environment_setup_guide.html)将nv_small/vmod/nvdla编译为纯RTL source code.

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

成功`[TMAKE]:DONE`后，在目录下会生成一个`outdir`目录，目录下便是搭建vivado所需的全部RTL code.


### step 2: 新建RTL工程 (Win10)
1.添加源文件，可按照nv_small/spec/defs/nv_small.spec描述来指定vmod/nvdla/下待添加的内核源文件————small版本不包含bdma，retiming和rubik内核(文件夹)以及其他文件夹中的部分.v————实际中，可先添加top文件下的modules，之后，vivado显示的hierarchy中缺少什么文件，加之即可;

2.添加RAM文件时，要选择`outdir/nv_small/vmod/rams/fpga/small_rams/`文件夹下的RAM源文件;

**====Tips====** 

    1). Xilinx RAM资源的使用有4种基本方式————flexibility依次递减————i)源码推断, ii)XPMs, iii)直接例化
        8k/16k RAM primitives, iv)例化RAM IP.
	
    2). 待全部缺失文件补全后，可能会发现在`NV_nvdla` hierarchy以外，还存在一些modules，这个是因为一个文
        件中定义了多个modules，可以将未用到的modules comment掉.
	
3.关闭clock gating，原设计对RAM存储op使用了大量clock gating以降低功耗，但与processor，ASIC不同，FPGA的时钟树是设计好的，clock buf资源有限，若不关闭gating，可能产生很大skew(之前因为部分gating未关闭，测试一直不过)；使用到的clock gating开关宏包括以下4个，我个人是定义了一个.vh文件，define了这些宏，然后将该头文件include到指定.v文件，最初使用`set global header`没设置成功，所以只能傻傻搜索查找相关.v，不过用notepad++全局搜索，效率倒是还可以，大家自行处理;

- VLIB_BYPASS_POWER_CG
- NV_FPGA_FIFOGEN
- FIFOGEN_MASTER_CLK_GATING_DISABLED
- FPGA

**====Tips====** 

    1). 其实，也可以不关闭gating信号逻辑，使用syth选项支持相关逻辑的clock gating，clk buf会增加, 测试了
        部分信号, 待测试全部信号的资源消耗和逻辑稳定性.
	
    2). nvdla_pwrbus_ram_*_pd相关逻辑，可以转化使用BRAM的sleep达到相同效果，待测试.

4.[optional] 之后可在top module里添加generated时钟，进行综合、布局布线，查看资源开销、功耗和时钟频率等信息.


### step 3: 封装IP (Win10)
1. 添加wrapper，如果在NV_nvdla里例化了generated clock，请删除，另外，为了在block design中能够与PS的AXI master和slave接口连接，需要在当前工程结构下，再增加一个NV_nvdla_wrapper module封装NV_nvdla和apb2csb modules;

**====Tips====** 
	
	也可以将axi apb bridge ip同时封装在wrapper中，这样在BD工程中，PS->nvdla ip通信就变成了master->slave
	接口互连映射,而非本文采用的master->axi apb bridge->slave互连模式. 
	
2.补全AXI与APB信号，NVDLA使用AXI和APB协议与MCU通信，但是源码中缺失了部分信号需要按照`AMBA AXI and ACE Protocol Spec`和`AMBA 3 APB Protocol spec`补全缺失的AXI,APB信号，即

```
//append axi signal to NV_nvdla_wrapper signal list
output  [2 : 0]   M_AXI_AWSIZE,
output	[1 : 0]   M_AXI_AWBURST,
output            M_AXI_AWLOCK,
output  [3 : 0]   M_AXI_AWCACHE,
output  [2 : 0]   M_AXI_AWPROT,
output  [3 : 0]   M_AXI_AWQOS,
output    	  M_AXI_AWUSER,
output            M_AXI_WUSER,
input  	[1 : 0]   M_AXI_BRESP,
input             M_AXI_BUSER,
output  [2 : 0]   M_AXI_ARSIZE,
output  [1 : 0]   M_AXI_ARBURST,
output            M_AXI_ARLOCK,
output  [3 : 0]   M_AXI_ARCACHE,
output  [2 : 0]   M_AXI_ARPROT,
output  [3 : 0]   M_AXI_ARQOS,
output            M_AXI_ARUSER,
input   [1 : 0]   M_AXI_RRESP,
input   	  M_AXI_RUSER	

//append apb signal to NV_nvdla_wrapper signal list
output            S_APB_PSLVERR,
```

并将相应信号分别添加到NV_nvdla和NV_NVDLA_apb2csb的信号列表中，并分别添加如下赋值代码，

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
	
	添加如上信号后，其实已经能够在BD工程中完成interface连接任务了，但是仅作上述修改，BD工程中必须要使用AXI
	smartconnect做PL和PS的接口互连，如果想使用AXI interconnect或干脆直接将NVDLA master与PS slave互连，
	还要进一步修改如下信号位宽,否则，BD工程在`validate`的时候会报错.

```
//modify following signal bit width in NV_nvdla module
		
//input [7:0] 	  nvdla_core2dbb_b_bid;
input [5:0] 	  nvdla_core2dbb_b_bid;
//input [7:0] 	  nvdla_core2dbb_r_rid;
input [5:0] 	  nvdla_core2dbb_r_rid;
//output [7:0] 	  nvdla_core2dbb_aw_awid;
output [5:0] 	  nvdla_core2dbb_aw_awid;
//output [3:0] 	  nvdla_core2dbb_aw_awlen;
output [7:0] 	  nvdla_core2dbb_aw_awlen;
//output [7:0] 	  nvdla_core2dbb_ar_arid;
output [5:0] 	  nvdla_core2dbb_ar_arid;
//output [3:0] 	  nvdla_core2dbb_ar_arlen;
output [7:0] 	  nvdla_core2dbb_ar_arlen;
```

实际上，所有的*_id signal只有后4位work. 另外，不要忘了在NV_nvdla module的NV_NVDLA_partition_o u_partition_o{...}实例中修改相应信号位宽，这里就不写了.

3.添加xdc，新建两个xdc文件，一个为IP在OOC综合时使用，另一个则是在global syth时使用，OOC xdc可以直接约束两个primary clk，如

```
create_clock -period 10.001 -name u_dla_core_clk [get_ports u_dla_core_clk];
create_clock -period 10.001 -name u_dla_sys_clk [get_ports u_dla_sys_clk];
```

另一个xdc保持空白就行，这里涉及到了一个xdc的scope问题，感兴趣的可以参考Xilinx的UG903.除此之外，在`source`窗口选中OOC版本的xdc，在属性窗口的`USED_IN`属性里添加`out-of-context`选项，否则，BD综合时就出问题了，选中另一版本xdc，在属性窗口中为`PROCESSING_ORDER`属性选择`LATE`.

4.封装nvdla IP，Tools-->Create and Package New IP-->Package your current project, next and finish;
	
5.AXI master interface推断，点击`Ports and Interfaces`，如Fig-1, 查看是否有自动推断出的master接口，若没有，选中全部master接口信号，右键选择`Auto Infer Interface`，推断出master接口信号后，需要检查位宽是否与源文件中声明的相一致(之前出现过标量矢量化的情况，工具坑)，即`Size Left`，`Size Right`，另外，检查`Driver Value`，如Fig-2, 若官方源码中声明的信号在此列表中显示驱动强度为0，则选中该信号，在属性窗口删除0值，另外，需要对上面我们后添加的master信号，将驱动强度设置为0；
	
![Interface part](https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%231-%231.JPG?raw=true)

                                              Fig-1

![Width and Driver](https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%231-%232.JPG?raw=true)

                                              Fig-2

6.APB slave interface推断,这个接口不会自动推断，需要选中全部APB信号，右键自动推断，在弹出窗口中依次选择`Advanced`-->`apb_rtl`, 推断出AXI和APB信号后，右键两个接口信号，选择`Associate Clocks`，分别关联两个clk即可，若core和csb跑异步时钟，那么AXI关联core clk，APB关联csb clk；
	
7.APB memory map, 因为AXI/APB master-slave接口是memory-map机制做数据映射的，我画了一个简图Fig-3. 不同于AXI memory block的自动生成，APB memory block需要额外添加，选择`Addressing and Memory`-->`Memory Maps(for slaves)`，右键`IP Addressing and Memory Wizard`选择APB接口信号，继续右键`Add Address Block`(因为一块连续地址，一个block便可)，如Fig-4；

![Memory map](https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%231-%233.JPG?raw=true)

                                              Fig-3

![Address block](https://github.com/VVViy/VVViy.github.io/blob/master/img/blog%231-%234.JPG?raw=true)

                                              Fig-4

8.`Review and Package` -->`Package IP`.


### step 4: 新建BD工程 (Win10)
1.新建RTL工程，`Settings`-->`IP`-->`Repository`将刚刚封装的IP(nvdla_ip_prj_name.srcs)添加到IP列表，新建BD工程`Flow Navigator`-->`Create Block Design`；
	
2.添加ps，nvdla ip，axi apb bridge，axi interconnect等ip，ps要配置master，slave以及PL-PS中断接口，之后连接接口即可；

3.添加xdc，综合、布局布线和输出bit文件，之后export hardware（复选'include bitstream'）.

### step 5: [optional] 测试
Vivado+SDK/VIP/HW manager, 请自行选择，我没做 :sweat_smile:.
