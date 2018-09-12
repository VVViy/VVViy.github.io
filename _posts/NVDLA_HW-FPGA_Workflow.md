---
layout:     post
title:      nv_small FPGA Mapping Workflow
subtitle:   part I: Vivado project
date:       2018-09-12
author:     Max
header-img: 
catalog: true
tags:
    - NVDLA
    - nv_small
    - FPGA
---

# Preface
	大家好，本文将简要介绍映射nvdla nv_small版本到FPGA过程中可能遇到的‘坑’，旨在给做相同工作的小伙伴一个参考，不作指导性建议。

	本文会详细描述从使用官方源码构建vivado IP工程到最终BD工程网表生成的全部步骤以及每个步骤中容易出错的地方。但需要**说明**的是，该映射工作是本人公司项目的一部分，所以，受规则所限，文中不会直接展示官方开源内容以外的设计内容，望谅解。

# Development environment setup
	* Win10: build vivado prj for HW;
	* Vmware Ubuntu 16.04: host for petalinux prj;
	* Vivado: 2017.4
	* Petalinux: 2017.4, Linux kernel v4.9(不同版本petalinux的linux kernel版本不同，从而DMA API不同，导致对nvdla/sw/kmd驱动源码的修改方式有些区别).

# Step 1: build tree and vmod (Linux)
	官方HW工程是通过在branch/spec/defs/下定义不同的spec来对相同源码构建不同的行为模型架构，所以，源码内部有很多的C++和perl相关的条件编译。因此，在搭建vivado工程前，要先按照官方[NVDLA Environment Setup Guide](http://nvdla.org/hw/v2/environment_setup_guide.html)将nv_small/vmod/nvdla编译为纯RTL code。

	**Tips** 
	1). <requied> build tree所需的工具中，cpp，gcc，g++，perl，python采用linux系统默认，java需要额外安装，若只是build RTL，其他tool可直接enter。
	2). <requied> 在build RTL过程还需要perl module的支持，可以直接使用类似pip的perl CPAN安装错误信息中指定的module，如

	```perl
		$ perl -MCPAN -e shell
	
		...
	
		CPAN > install YAML
	
		...
	
		CPAN > exit
	
	```

	成功`[TMAKE]:DONE`后，在目录下会生成一个`outdir`目录，目录下便是搭建vivado所需的全部code。

# step 2: 新建RTL工程 (Win10)
	1. <requied> 添加源文件，可按照nv_small/spec/defs/nv_small.spc指定vmod/nvdla/下待添加的内核源文件---small版本不包含bdma，retiming和rubik内核(文件夹)以及其他文件夹中的部分.v---实际，可先添加top文件下的modules，之后，vivado显示的hierarchy中缺少什么文件，加之即可;
	2. <requied> 添加RAM文件时，要选择*outdir/nv_small/vmod/rams/fpga/small_rams/*文件夹下的RAM源文件;

	**Tips** 
	1). Xilinx RAM资源的使用有4中方式---flexibility依次递减---i)源码推断, ii)XPMs, iii)直接使用RAM primitive, iv)例化RAM IP.
	2). 待全部缺失文件补全后，可能会发现在`NV_nvdla` hierarchy以外，还存在一些modules，这个是因为一个文件中定义了多个modules，可以将未用到的modules comment掉.
	
	3. <requied> 关闭clock gating，原设计对RAM存储op使用了大量clock gating以降低功耗，但与processor，ASIC不同，FPGA的时钟树是设计好的，且buffer资源有限，若不关闭gating，有很大skew(之前因为部分gating未关闭，测试一直不过)；使用到开关宏包括以下4个，我是定义了一个.vh文件，定义了这些宏，然后将头文件include到指定.v文件；之前使用`set global header`没设置成功，所以只能傻傻搜索查找相关.v，用notepad++处理，倒是还可以，大家自行处理;

	- VLIB_BYPASS_POWER_CG
	- NV_FPGA_FIFOGEN
	- FIFOGEN_MASTER_CLK_GATING_DISABLED
	- FPGA

	**Tips** 
	1). 其实，也可以不关闭gating信号逻辑，使用syth选项支持相关逻辑的clock gating，clk buf会增加, 测试了部分信号, 待测试全部信号的资源消耗和逻辑稳定性.
	2). nvdla_pwrbus_ram_*_pd相关逻辑，可以转化使用BRAM的sleep达到相同效果，待测试.

	4. [optional] 之后可在top module里添加generated时钟，进行综合、布局布线，查看资源开销、功耗和时钟频率等信息.

# step 3: 封装IP (Win10)
	1. <required> 添加wrapper，如果在NV_nvdla里例化了generated clock，请删除；另外，为了在block design中能够与PS的AXI master和slave接口连接，需要再增加一个NV_nvdla_wrapper module封装NV_nvdla和apb2csb modules;
	2. <required> 补全AXI与APB信号，NVDLA使用AXI和APB协议与MCU通信，但是源码中缺失了部分信号需要按照`<AMBA AXI and ACE Protocol Spec>`和`<AMBA 3 APB Protocol spec>`补全，缺失的AXI,APB信号包括，

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

		//append apb signal
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

	**Tips** : 正常来说，添加如上信号便可以在BD工程中建立连接了，但是在仅作如上修改的情况下，在BD工程中必须要使用AXI smartconnect做PL和PS的接口互连，如果想使用AXI interconnect或干脆直接将NVDLA master与PS slave互连，还要进一步修改如下信号位宽,否则，BD工程在`validate`的时候会报错

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
	3. <required> 添加xdc，新建两个xdc文件，一个为IP在OOC综合时使用，另一个则是在global syth时使用，OOC xdc可以直接约束两个primary clk，如

	```
		create_clock -period 10.001 -name u_dla_core_clk [get_ports u_dla_core_clk];
		create_clock -period 10.001 -name u_dla_sys_clk [get_ports u_dla_sys_clk];
	```

	另一个xdc保持空白就行，这里涉及到了一个xdc的scope问题，感兴趣的可以参考Xilinx的UG903.除此之外，在`source`窗口选中OOC版本的xdc，在属性窗口的`USED_IN`属性里添加`out-of-context`选项，选中另一版本xdc，在属性窗口中为`PROCESSING_ORDER`属性选择`LATE`.

	4. <required> 建IP工程，Tools->Create and Package New IP->Package your current project 
	2. <required> 因为BD工程本身是interface级别的，所以为了在BD里能够与其他interface 协议的IP直接做互连（本工程里主要是axi和apb interface），需补全源码中缺失的标准axi和apb协议信号，参考。。。；
	3. <required> 为IP添加ooc和全局使用xdc文件，ooc要在属性窗口的USED_IN进行修改，而IP的全局xdc要在processing_order选择late属性，内容为空，参考。。。；
	4. <required> 到IP定制页面，选中axi相关信号，右键选择自动推断，这里要注意查看，推断出的信号位宽是否与源码一致，不一致的在属性窗口修改，并对后续添加的协议信号添加driver value——0；
	5. <required> 分别选中axi和apb接口信号，右键选择关联各自的时钟信号；之后，切换到'Addressing and Memory'页面为apb接口添加memory block，因为ARM的系列总线协议都是memory mapping机制的，映射机制可参考。。。，下面是我画的一个简图，如果这里不为apb接口添加memory block那么PS访问不了apb，右键。。。。

# step 4: 新建BD工程 (Win10)
            # <required> 新建立一个RTL project，首先，在’setting‘的IP->repository下’+‘刚封装好的IP工程（选择src），之后，选择左侧导航栏’create block design‘，搭建BD工程，分别加入ps，nvdla ip，axi interconnect/smartconnect，apb bridge，processor system reset，constant等IP，编写xdc，综合、布局布线和输出bit文件，之后export hardware（复选'include bitstream'）。

# step 5: [optional] 测试, HW prj, vivado+sdk, VIP.

