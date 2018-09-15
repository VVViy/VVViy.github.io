---
layout:     post
title:      nv_small FPGA Mapping Workflow
subtitle:   Part II-Petalinux project
date:       2018-09-13
author:     Max
header-img: img/post-gray-backgroud.jpg
catalog: true
tags:
    - NVDLA
    - nv_small
    - FPGA
    - Petalinux
---


### Preface
参考Part I，基本可以跑起FPGA工程，而对于使用Xilinx heterogeneous architecture FPGA chip的小伙伴，完成PL.hdf设计后，肯定要进一步配置ARM linux，下板跑跑`github.com/nvdla/sw`Sanity测试集，所以，本文将结合作者实践及官网`issue`中的信息，尽可能完整的描述Xilinx petalinux工程构建流程及对官方SW工程源码的定制化修改.

此外，无论是Part I还是本文，文章内容描述方式都以‘How to do’为第一目标，背后的技术点并未阐述，因为不想干扰操作过程。实际上，作者在三个月的摸索过程中，遇到很多值得留意的技术点，特别是在搭多时钟域工程的时候，后期希望与同道讨论后逐步添加到文章中.
a
### Step 1: Install petalinux in host
Xilinx petalinux是一个定制版的`Yocto`工具，Xilinx已经把BSP准备的妥妥的了，我们要做的是定制自己的kernel. 安装过程在Xilinx `UG1144`写的比较清楚，这里做个概述.

1.linux环境下安装依赖库，按照`UG1144 Table 2-1`补全依赖工具包和库，串口调试工具作者使用的是`putty`和`gtkterm`.

2.修改shell，`UG1144`提示要修改`shell`为`bash`，可以`echo $SHELL`查看一下当前`/bin/sh`是否为`bash`，如果不是，`sudo dpkg-reconfigure`,在弹出界面中点`否`或重建`ln`.

3.安装petalinux，到[Xilinx download center](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools.html)下载`petalinux [version number] installer`,`[version number]`要和Vivado版本一致，其他的`BSP, sstate`无需下载，`petalinux-build`工程时，会自动download. 运行下述命令安装，

```
$ ./petalinux-v2018.1-final-installer.run <path-to-target-install-dir>
```

和S家EDA工具安装.run文件一样，不要以sudo权限安装，`permission denial`就`chmod`改下权限. 安装目录自选即可，之前有个Agilent工程师说必须安装在`/opt/pkg/`下，否则可能会crash，也许是`centos`系统原因，`ubuntu`并不存在此问题. 另外，安装进程到`license`时，有限几处会出现`...[y/N]?`，别急着回车，这里默认是`N`，需要手动`y`.

4.验证安装正确性，安装完成后，应该会出现`Warning:No tftp server found...`, 如果不是使用`NFS`文件系统做远程boot，则无需在意此Warning，强迫症请参考下述命令.

```
$ apt-get install tftpd tftp openbsd-inetd
$ echo tftp dgram udp wait nobody /usr/sbin/tcpd /usr/sbin/in.tftpd /tftproot >> /etc/inetd.conf
$ mkdir -p <specified-dir>/tftproot
$ /etc/init.d/openbsd-inetd restart
```

**注意**，千万不要把重定向追加`>>`,写成重定向覆盖`>`.

运行下述命令验证安装正确性，

```
$ source <path-to-installed-PetaLinux>/settings.sh
$ echo $PETALINUX 

#if output path is <path-to-installed-PetaLinux>, then success.
```

### Step 2: Create and configure petalinux project 
1.创建petalinux工程，工具安装成功后，运行如下命令建立工程，

```
$ cd <specified-prj-dir>
$ source <path-to-installed-Petalinux>/settings.sh
...
$ petalinux-creat -t project --template [zynq/zynqMP/microblaze] -n [project_name]
```

对于`template`，如果使用`zynq-7000`系列芯片，选择`zynq`；`zynq +UltraScale MPSoC`，则选择`zynqMP`.

**====Tips====**

    1). 创建Petalinux工程有两种方式：i)使用下载的.bsp文件创建，ii)建个空工程.对于nvdla工程，后续都要使用PL.hdf
        重新build，没有区别；但如果不使用PL逻辑，可以直接使用第一种的prebuilt bootloader文件下板，而不用重新build;
        
    2). petalinux的相关命令，可参考UG1157.
    
2.配置petalinux工程，首先将Vivado `export hardware`输出的`*.hdf`文件拷贝到新建的petalinux工程目录下，之后运行如下命令，

```
$ cd <path-to-petalinux-prj>
$ petalinux-config --get-hw-description=./
```

* 在配置界面中，选中`DTG settings`-->`template...`, enter进入修改为开发板版本，如`zcu102-rev1.0`，详见`UG1144 Chapter 3，p22`；
* 进入`Image Packaging Configuration`-->`Root Filesystem Type`，enter进入选中`SD card`. 修改此处后，linux根目录系统`rootfs`将配置到SD中，而非默认的`raminitfs`，后者是将根目录系统镜像在boot阶段加载到内存中，一旦裁剪的kernel较大（大概超过120M），那么系统boot不起来；
* 退出并保存配置.

3.配置kernel，由于`rootfs`配置到SD boot，那么就要取消掉kernel的....ram init....，否则在boot阶段，kernel在内存中找不到rootfs的符号镜像，便会出错，

```
$ petalinux-config -c kernel
```

弹出窗口中，选择..........................，取消掉.....................，如Fig-1，退出并保存配置.


    Fig-1

### Step 3: Customize the linux kernel for building UMD
`nvdla/sw/prebuilt/linux/`中包含了官方在`kernel v4.13.3`下预编译的`nvdla_runtime`ELF文件和依赖库`libnvdla_runtime.o`文件，但`patelinux 2017.4`的kernel是`v4.9`，两个版本的DMA API不同，导致依赖于`DRM`实现`DMA`数据搬移的`KMD`驱动无法工作，从而，`UMD`无法正常运行，因此，需要重新为4.9版本编译`UMD`. 

本来最初打算写`recipe`通过petalinux的`bitbake`直接编译`/nvdla/umd/`下的源码，将动态库`.o`和`elf`添加到`rootfs`下，但研究发现这很难实现，petalinux工具只能添加`prebuilt`的.o文件，而不能新建以库文件为目标的子工程（只能创建`apps`，`modules`和`install`子工程），即使将`.o`的编译添加到`nvdla_runtime`的`apps`子工程`makefile`编译中，如何配置对应`recipe`保存中间生成的`.o`文件到`rootfs`，也是个难题. 所以，最后选择配置kernel，添加`GNU toolchain`，直接下板编译`UMD`的一切所需,`nvdla/sw/umd/`源码目录结构对应的`makefile`将编译依赖关系妥善处理，所以，直接下板`make`即可，比较方便.

1.定制rootfs，实际上，petalinux不仅提供了很多不同平台版本的可配置kernel镜像，还提供了各种系统工具包和库文件，像OpenCV，Xen等等，配置根目录

```
$ petalinux-config -c rootfs
```

弹出界面中，选择..............,选择....，如Fig-2 ~ Fig-3，退出并保存.如果要实现更复杂的功能，可以选择.....，但该包过大(~10G);也可以选择添加其他如`ldd`，`sudo`等工具包和库. `misc`下各种`group`包的功能描述，可参考[Building a Custom Linux Distribution](http://www.informit.com/articles/article.aspx?p=2514911)


    Fig-2
    
    
    Fig-3

2.编译petalinux工程，配置到这里可以先编译一次工程，否则，后续修改`device tree`时无法查看`PL nvdla`的节点信息，编译petalinux project

```
$ petalinux-build
```

### Step 4: Create and add KMD module by petalinux tool
1.创建`modules`子工程,驱动模块kmd的编译只能通过petalinux工具编译添加，且编译方式不同于`nvdla/sw/readme`中介绍的`generic kernel out-of-tree module build`（可参考linux kernel官网[Building External Modules](https://www.kernel.org/doc/Documentation/kbuild/modules.txt)）,创建modules子工程

```
$ petalinux-create -t modules -n opendla --enable
```

modules子工程创建后，将`nvdla/kmd/`下的所有`.c,.h`文件拷贝到`<path-to-petalinux-prj>/project-spec/meta-user/recipes-modules/opendla/files/`目录下.

2.修改驱动源文件,之前提到不同版本的kernel，对应的API有所不同，需要根据自己的内核版本修改驱动源码中的调用函数. 作者使用4.9内核，需做如下修改

* `nvdla/sw/kmd/port/linux/nvdla_gem.c,line 332`的`drm_gem_object_put_unlocked()`函数是在`v4.12`之后版本中才出现，其替代了之前版本中的`drm_gem_object_unreference_unlocked`，详情可参考[Linux kernel v4.12 DRM TODO List](https://www.kernel.org/doc/html/v4.12/gpu/todo.html#switch-from-reference-unreference-to-get-put)

```
...
332: //drm_gem_object_put_unlocked(dobj);
333: drm_gem_object_unreference_unlocked(dobj);
...
```

* `line 439: dma_declare_coherent_memory(drm->dev, 0xC0000000, 0xC0000000,0x40000000, DMA_MEMORY_MAP | DMA_MEMORY_EXCLUSIVE);`该函数表示需要在`PS DDR`中保留一块不被kernel支配，大小为`0x40000000`的地址空间，以便`nvdla core`使用`DMA`方式对`CNN`输入图像和处理结果在Cache与DDR间进行数据搬移，不知为何官方源码采用了硬编码方式处理物理地址，总线地址和存储块尺寸参数. 按照源码中的注释`/* TODO Register separate driver for memory and use DT node to read memory range */`， 表明官方不希望我们修改函数中的参数信息，而是根据参数值去修改`device tree`中对应的`Node`属性值.但在Vivado BD工程中，`PS DDR`的有效地址为`0x00000000 ~ 0x7FFFFFFF`, 显然，参数列表中的物理地址和总线地址不在`PL`的有效访问范围内，必须对此进行修改，至于具体改为什么值，可自行选择，只要是`page size`整数倍便可，如`256 MiB`,那么便将两个`0xC0000000`修改为`0x70000000`(为何取高位地址，感兴趣的可参考[Memory Mapping and DMA](http://static.lwn.net/images/pdf/LDD3/ch15.pdf),老司机请略过), 同时，存储块尺寸不需要修改. 作者保留了1G空间,

```
...
439: //dma=dma_declare_coherent_memory(drm->dev, 0xC0000000, 0xC0000000,0x40000000, DMA_MEMORY_MAP | DMA_MEMORY_EXCLUSIVE);
440: dma=dma_declare_coherent_memory(drm->dev, 0x40000000, 0x40000000,0x40000000, DMA_MEMORY_MAP | DMA_MEMORY_EXCLUSIVE);
...
```

**====Tips====**

    对于使用2018版Xilinx工具的小伙伴，除了上述修改，还要删除DMA_MEMORY_MAP flag，官方开发者在issue中提到该flag已经
    在kernel v4.13后续版本中删除了该flag.

* 对于`nv_small`版本，还要修改`nvdla/sw/kmd/firmware/include/opendla.h`，添加`DLA_SMALL_CONFIG`宏，这样`KMD`驱动才能根据`opendla_small.h`寄存器声明来完成对`nvdla core`内部子内核的裁剪,使其符合`nv_small/spec/defs/nv_small.spec`定义.

### Step 5: Modify the default device tree for NVDLA identification
`Device tree`是`ARM`处理器`Bootloader`必备之品，`FSBL`在系统boot阶段将`device tree`加载到内存，之后的kernel才会根据其内部`Node`配置内核，确定有哪些硬件设备可用.这个"确定"过程首先便是通过特定`diver`的`probe`函数搜索`DTB`中是否存在匹配的`Compatible`属性. 那么对于nvdla工程，需要查看`KMD`驱动`nvdla probe`函数的`Compatible`属性值是否与petalinux工程中`device tree`中`PL node`中的`Compatible`相一致，查看`nvdla/sw/kmd/port/linux/nvdla_core_callbacks.c, line 338`显示`kmd probe`函数指定的`compatible`属性值为`.compatible = "nvidia,nvdla_2"`(`nvdla/sw/kmd/Documentation/devicetree/bindings/nvdla/nvdla.txt`中的`compatible`属性值`compatible = "nvidia,nvdla-1"`是针对`nvdla_full`版本的)，而`<path-to-petalinux-prj>/component/plnx_workspace/device-tree/pl.dtsi`中的节点信息描述了我们在Vivado中创建的`nvdla`工程，其中的`compatible`属性值显然与驱动函数中的不同，需要修改一致. 即在`<path-to-petalinux-prj>/project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi`覆盖掉`compatible`属性值，并添加之前提到的`reserved memory`(`reserved memory`的节点定义可参考[Linux Reserved Memory](http://www.wiki.xilinx.com/Linux+Reserved+Memory))

```
/include/ "system-conf.dtsi"
/ {
    reserved-memory {
            #address-cells = <2>;
            #size-cells = <2>;
            ranges;

            nvdla_reserved: buffer@0 {
                      no-map;
                      reg = <0x0 0x70000000 0x0 0x40000000>;
            };
    };
};

&[your lable-name appeared in pl.dtsi]{
    compatible = "nvidia,nvdla_2";
    memory-region = <&nvdla_reserved>;
}
```

**====Tips====**

    1). 根据`UG1144 APPX.B Table B-1`所言,<path-to-petalinux-prj>/component/plnx_workspace/device-tree/
        目录下后缀为`.dtsi`和`.dts`的`device tree`定义文件是petalinux工具自动生成的，每次`petalinux-build`都会
        自动覆盖更新，所以对其修改无用.而`system-user.dtsi`不会被工具修改.
    
    2). Device tree格式目前尚无统一标准，Linaro要牵头出标准，已经发布了DeviceTree Specification Release v0.2 
        (www.devicetree.org)但内容尚不完整，作者之前参考了Linaro, Linux， Raspberry Pi， ARM， NXP， Toradex等
        多家的Device Tree文档，后期会在Blog里挂出survey and summary文档.

### Step 6: Build petalinux project and package bootloader files
1.重新编译petalinux,在之前的`petalinux-build`之后，不仅创建了`modules`子工程，而且修改了`device tree`，所以需要重新编译整个工程，如果仅仅是添加了`apps/modules`，那么只要按照`UG1144 Chapter 7`做增量编译即可.

2.制作`bootloader`， `petalinux bootloader`使用`u-boot`, 可按照下述命令打包`bootloader`相关文件，下板需要使用到`BOOT.BIN`, `image.ub`和`rootfs.ext4`, `BOOT.BIN`包含`fsbl, bitstream, pmu, u-boot`, `image.ub`包含`kernel image, DTB, rootfs image`.

```
$ cd <path-to-petalinux-prj>/images/linux/
$ petalinux-package --boot --fsbl [zynq_fsbl/zynqmp_fsbl] --fpga [your *.bit file] --pmufw pmufw --u-boot 
```

### Step 7: Partition and configure SD card
1.SD卡分区，根据`UG1144 Chapter 6 “Configuring SD Card ext filesystem Boot” section`对SD分区，作者将SD卡划分为`BOOT(fat32)`, `rootfs(etx4)`, `Workbench(etx4)`三个分区，分别放置boot文件，rootfs，UMD源文件和测试相关文件.

2.

### Step 8: Download and run tests









