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
参考Part I，基本可以跑起FPGA工程，而对于使用Xilinx heterogneous architecture FPGA chip的小伙伴，完成nv_small_prj.hdf设计后，肯定要进一步配置ARM linux，下板跑跑`github.com/nvdla/sw`Sanity测试集，所以，本文将结合作者自己实践，尽可能完整的描述Xilinx petalinux工程构建流程及对官方SW工程源码的定制化修改.

此外，无论是Part I还是本文，文章内容描述方式都以‘How to do’为第一目标，背后的技术点并未阐述，因为不想干扰操作过程。实际上，作者在三个月的摸索过程中，遇到很多值得留意的技术点，特别是在搭多时钟工程的时候，后期会与同道讨论并逐步添加到文章中。

### Step 1: Petalinux installation in linux
Xilinx petalinux相当于是一个定制版的Yocto工具，Xilinx已经把BSP准备的妥妥的了，我们要做的是定制自己的linux kernel. 安装过程，Xilinx `UG1144`写的比较清楚，这里做个概述.

1.linux安装依赖库，按照`UG144-Table 2-1`补全依赖工具包和库，串口调试工具我用的是`putty`和`gtkterm`.

2.修改shell，`UG1144`提示要修改`shell`为`bash`，可以`echo $SHELL`查看一下当前/bin/sh是不是`bash`，不是的话`sudo dpkg-reconfigure`,在弹出界面中点`否`或重建ln.

3.安装petalinux，到[Xilinx download center](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/embedded-design-tools.html)下载`petalinux [version number] installer`,`[version number]`要和Vivado版本一致，其他的`BSP, sstate`不用管，`petalinux-build`工程时，会自动download. 运行下述命令安装，

```
$ ./petalinux-v2018.1-final-installer.run <path-to-target-install-dir>
```

和S家EDA工具安装.run一样，不要以sudo权限安装，`permission denial`就`chmod`改下权限. 安装目录自选即可，之前有个agilent工程师说必须安装在`/opt/pkg/`下，否则可能会crash，也许是`centos`系统原因，`ubuntu`不存在的.另外，安装进程到license时，一旦出现`...[y/N]?`，别急着回车，这里默认是`N`，需要手动`y`.

4.验证安装正确性，安装完成后，应该会出现`Warning:No tftp server found...`, 这个如果不是`NFS`做远程boot的话，是不用在意的，强迫症请参考下述命令.

```
$ apt-get install tftpd tftp openbsd-inetd
$ echo tftp dgram udp wait nobody /usr/sbin/tcpd /usr/sbin/in.tftpd /tftproot >> /etc/inetd.conf
$ mkdir tftproot
$ /etc/init.d/openbsd-inetd restart
```

**注意**，千万不要把重定向追加`>>`,写成重定向覆盖`>`.

运行下述命令验证，

```
$ source <path-to-installed-PetaLinux>/settings.sh
$ echo $PETALINUX 

#if output path is <path-to-installed-PetaLinux>, then success.
```

### Step 2: Create petalinux project and configure SD card boot
1.创建petalinux工程，工具安装成功后，运行如下命令建立工程;

```
$ cd <specified-prj-dir>
$ source <path-to-installed-Petalinux>/settings.sh
...
$ petalinux-creat -t project -template [zynq/zynqMP/microblaze] -n [project_name]
```

这里的`template`，如果使用`zynq-7000`系列，则选择`zynq`，`zynq +UltraScale MPSoC`，选择`zynqMP`.

**====Tips====**

    创建Petalinux工程有两种方式：i)使用下载的.bsp文件创建，ii)建个空工程.对于nvdla工程，后续都要使用PL.hdf重新build，
    没有区别；但如果不使用PL逻辑，可以直接使用第一种的prebuilt bootloader文件下板，而不用重新build.
    
2.配置petalinux工程，首先将Vivado `export hardware`输出的`*.hdf`文件拷贝到新建的petalinux工程文件下，之后运行如下命令，

```
$ cd <path-to-petalinux-prj>
$ petalinux-config --get-hw-description=./
```

* 在配置界面中，方向键选中`DTG settings`-->`template...`, enter进入修改为开发板版本，如`zcu102-rev1.0`；
* 进入`Image Packaging Configuration`-->`Root filesystem Type`，enter进入选中`SD card`. 修改此处后，linux根目录系统`rootfs`将配置到SD中，而非默认的`raminitfs`，后者是将根目录系统镜像在boot阶段加载到内存中，所以，一旦裁剪的kernel增大（大概超过120M），那么系统boot不动.


### Step 3: Customize the linux kernel for building UMD


### Step 4: Create and add KMD module by petalinux tool


### Step 5: Modify the default devide tree for NVDLA identification


### Step 6: Build petalinux project and package bootloader files


### Step 7: Partition and configure SD card


### Step 8: Download and run tests









