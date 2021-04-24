---
title: 【初入电子坑之stm32篇（二）】固件库编程
mathjax: true
date: 2021-01-16 12:37:17
tags: stm32
categories: 电子
---

## 前言

> 在[【初入电子坑之stm32篇（一）】初识stm32与寄存器编程](https://zhuanlan.zhihu.com/p/344913189)中可以看到，寄存器编程虽然消耗CPU性能少、速度快。但于我们开发应用来说，那就是刀耕火种中的刀耕火种。。。
>
> 芯片厂家也考虑到了这点，所以一般会提供一些基本的固件库供开发人员使用。stm32的爸爸ST公司自然也不例外。

<!--more-->

## CMSIS标准

一种内核架构通常会被多家芯片厂家采用，而芯片厂家对于芯片内部的外设布置是有差异的，这就很容易导致软件在同内核，不同外设的芯片上移植困难。

为了解决这个问题，设计架构的公司和使用架构的芯片厂商间就会制定一个标准，让软件可以适配同内核不同外设的芯片的情况。

而CMSIS标准(Cortex MicroController Software Interface Standard) 就是 ARM 与芯片厂商建立的一种标准实例。

CMSIS标准其实是一个软件抽象层。

![层次关系](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E5%B1%82%E6%AC%A1%E5%85%B3%E7%B3%BB.png)

（上图中，黄色部分是MCU硬件层面的东西，蓝色CMSIS层其实就是程序。）

它的内容有两个方面：

-	内核函数层：内核寄存器的名字、地址定义。由ARM公司提供。
-	设备外设层：片上核外外设的地址和中断定义。有芯片生产商负责。

## 初识stm32固件库

了解了CMSIS标准后，我们就可以很愉快的去stm32提供的官方固件库找我们要的东西了。

打开固件库文件夹我们可以看到有以下两个文件夹：

![初识固件库](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E5%88%9D%E8%AF%86%E5%9B%BA%E4%BB%B6%E5%BA%93.png)

从文件名就可以盲猜，一个是CMSIS标准相关的配置文件夹，一个是相对具体型号的芯片外设配置文件夹。

![CMSIS](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/CMSIS.png)![CM3](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/CM3.png)

<img src="https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E5%86%85%E6%A0%B8%E5%AF%84%E5%AD%98%E5%99%A8%E6%98%A0%E5%B0%84%E7%9B%B8%E5%85%B3.png" alt="内核寄存器映射相关" style="zoom:50%;" /><img src="https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E6%97%B6%E9%92%9F%E9%85%8D%E7%BD%AE%E6%96%87%E4%BB%B6.png" alt="时钟配置文件" style="zoom:50%;" />

点进CMSIS可以发现，不出所料，就是内核的配置相关配置的源文件、stm3210x的时钟配置文件还有启动文件。

我们用的时候把它们复制到我们自己的工程里即可。

然后去隔壁 STM32F10x_StdPeriph_Driver 文件夹瞄瞄，这个就很简单粗暴了，一个inc（include）、一个src（source）。

<img src="https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E5%A4%96%E8%AE%BE%E6%96%87%E4%BB%B6.png" alt="外设文件" style="zoom:50%;" />

随便点进去就会发现ST官方按照不同的外设模块，stm32f10x_xxx.c的方式给分门别类的放好了。（见下图）

![固件库命名](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E5%9B%BA%E4%BB%B6%E5%BA%93%E5%91%BD%E5%90%8D.png)

既然东西齐活了，下面就可以先愉快的建个固件库工程模板，然后正式开启固件库编程之旅了~~~

（创建固件库工程方法，请移步以下链接：

[【初入电子坑之stm32篇（补充1）】新建固件库工程](https://zhuanlan.zhihu.com/p/367054113)

## stm32固件库编程

下面结合实例叙述固件库的编程方式。

### 实例：按键控制LED

首先，让我们回忆一下，点灯的流程：

- 开始GPIO口时钟使能
- 初始化GPIO
  - 选择引脚IO口引脚
  - 并选择需要的IO模式
- 设置对应IO口的位值

如果有按键开关的话就是，就加上以下流程：

- 判断读取开关对应IO口的值（IDR寄存器里）
- 根据判断情况做相应的动作

### 库函数运用

上述流程动作，固件库全都是给我们安排得妥妥，直接调用相关函数就可以了。

具体的相关函数都在对应外设的头文件最下面，如我们即将使用的GPIO:

<img src="https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/gpio%E5%A4%B4%E6%96%87%E4%BB%B6.png" alt="gpio头文件" style="zoom:80%;" />

<img src="https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E5%85%B7%E4%BD%93%E5%87%BD%E6%95%B0%E6%BC%94%E7%A4%BA.png" alt="具体函数演示" style="zoom:80%;" />

从上面两图可以看到：

- 想要什么基本操作？去对应外设的库头文件找啊！
- 函数参数怎么用？直接去找源码看定义啊！
- 不知道函数功能是啥？直接看源码注释啊，官方写得老详细了。。。

具体代码在本人github的study record库里。