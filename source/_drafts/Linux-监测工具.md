---
title: Linux 监控工具
mathjax: true
date: 2021-8-23 23:31:29
tags: 工具
categories: [Linux,基础操作]
---

## 概要

![监测工具导图](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/%E7%9B%91%E6%B5%8B%E5%B7%A5%E5%85%B7%E5%AF%BC%E5%9B%BE_1.png)

<!--more-->

## NetHogs

### 作用

一个开源的命令行工具（类似于Linux的top命令），用来按进程或程序**实时统计**网络带宽使用率。

### 使用

- 默认情况
- 交互式命令
- 命令行参数

#### 默认情况

在命令行中输入nethogs即可进入该界面。

![NetHogs界面](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/NetHogsUI.png)

从第一栏中可以看到，每列的内容分别是：

- PID：进程ID
- User：进程所属用户
- Program：程序在地址所在（如：对应的本地地址、ip地址端口号）
- Dev：如果进程属于某个设备，如：网卡eth0则会显示出来，否则不显示
- Sent、Received：发送、接受的流量的速率

#### 交互式命令

- m: 按 m键，切换单位或显示占用速度；切换顺序是（KB/sec，KB，B，MB）
-  r : 按 r 键，按接收流量排序
-  s : 按 s 键 ，按发送流量排序
-  q : 按 q 键退出

![NetHogsTest_1](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/NetHogsTest_1.png)

#### 命令行参数

 -h :显示可用命令的用法

 -V :打印版本信息

 -d :延迟刷新率(延迟刷新时间)，单位是秒，默认为每秒刷新一次

 -v :选择视图模式

 -p :混合模式下嗅探（不推荐）

 -t :跟踪模式

## htop

### 作用

**交互式**进程管理器

### 界面

![htopUI](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/htopUI.png)

#### 上半区

![htopUI_1](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/htopUI_1.png)

#### 下半区

![htopUI_2](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/htopUI_2.png)

- PRI：进程的优先级
- NI：进程的优先级别值，默认的为0，可以进行调整
- VIRT：进程占用的虚拟内存值
- RES：进程占用的物理内存值
- SHR：进程占用的共享内存值
- S：进程的运行状况，R表示正在运行、S表示休眠，等待唤醒、Z表示僵死状态
- CPU%：该进程占用的CPU使用率
- MEM%：该进程占用的物理内存和总内存的百分比
- TIME+：该进程启动后占用的总的CPU时间
- COMMAND：进程启动的启动命令名称

#### 功能栏

![image-20210821212336575](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/htopUI_3.png)

根据需求，用到时再探索

## nmon

### 作用

它能在系统运行过程中实时地捕捉系统资源的使用情况，记录的信息比较全面，

并且能输出结果到文件中，然后通过nmon_analyzer工具产生数据文件与图形化结果。

### 界面

![nmonUI](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/nmonUI.png)

通过初始界面显示的命令，如下图般的界面（使用了 C、c、n 三个指令）：

![nmonUItest_1](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/nmonUItest_1.png)

可以看到，各项指标数据都很详细，具体使用时可以用 

```
nmon -h
```

查看更多操作方式。

## dstat

### 作用

多功能系统资源统计生成工具（ versatile tool for generating system resource statistics），可以实时地看到所有系统资源。

### 界面

默认显示内容：

![dstatUI](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Linux/dstatUI.png)

从蓝色虚线看，分成了以下几个部分：

-  --total-cpu-usage--：CPU使用率
  - usr：用户空间的程序所占百分比；
  - sys：系统空间程序所占百分比；
  - idel：空闲百分比；
  - wai：等待磁盘I/O所消耗的百分比；
  - hiq：硬中断次数；
  - siq：软中断次数；
- -dsk/total-：磁盘统计
  - read：读总数
  - writ：写总数
- -net/total- ：网络统计
  - recv：网络收包总数
  - send：网络发包总数
- --paging--： 内存分页统计
  - in： pagein（换入）
  - out：page out（换出）
- --system--：系统信息
  - int：中断次数
  - csw：上下文切换

### 常用指令

- 监控CPU\MEN： dstat --top-mem --top-io --top-cpu
- 常用常规监控：dstat -cmsdnl -D sda1 -N lo,ens33 100 5
