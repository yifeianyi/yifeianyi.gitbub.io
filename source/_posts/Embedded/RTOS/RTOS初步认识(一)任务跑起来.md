---
title: RTOS初步认识(一)任务跑起来
mathjax: true
date: 2021-11-21 00:52:56
tags: RTOS
categories:	[嵌入式,RTOS]
---

## 前言

> 最近开始对RTOS感兴趣，故开新坑做下学习记录。
>
> 因为是基于stm32基础上学习RTOS，且本人在学习RTOS之前没有使用过hal库开发，所以阅读本系列的前置技能应为有一定固件库基础的读者朋友。

<!--more-->

## 内容概览

![RTOS_Primer_catalogue](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/RTOS_Primer_catalogue.png)

RTOS（实时操作系统）属于 OS 的一种，而 OS 的知识体系非常庞大，RTOS作为它的子集体量也不小，所以为了兼顾 原理的认识 和 便于萌新易上手 两方面考虑，RTOS的初步认识将分为三篇叙述。本文为第一篇——跑起来。

## 跑起来

对于刚接触 RTOS 的朋友们来说，我相信使用固件库并手动移植RTOS代码是个很蛋疼的过程。这个过程容易出错且繁琐，大大增加了我们学习 RTOS 核心内容的难度。

所以在这里，我强烈建议原来使用固件库的朋友们使用 stm32CubeMX 学习 RTOS，这能让我们少掉很多根头发。。。

ok，在本文中我们先尽可能抛开所有和 OS 相关的概念，先让一个运行着 FreeRTOS 的 stm32板子跑起来，让一盏 LED 闪烁先。

### 一、系统移植

#### 1.1、CubeMX的使用

这小节将叙述如何使用 Hal库和CubeMX，如果已经掌握的朋友，可以跳过。

首先，Hal库我们不需要自己下载，只需要下载安装 CubeMX 即可，下载完后，所需要的Hal库包 CubeMX 会为我们自动下载，至于 CubeMX 的下载安装过程，这里略过。

回忆一下，对于一个LED工程的创建流程，大概有以下步骤：

1. 芯片选型
2. 时钟配置
3. LED引脚配置

##### 1.1.1、芯片选型

![CubeMX_UI_select(1)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_UI_select(1).png)

点击第一个即可。

![CubeMX_UI_select(2)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_UI_select(2).png)

右下方的是各种 STM32 的，我们只要找到想要的芯片型号并双击即可创建一个专属的芯片设置环境。但因为这实在太多，所以我们可以使用左上方的搜索框输入芯片型号即可。

这里我使用的是蓝桥杯嵌入式的新板子，芯片型号为：STM32G431RBTx 。

![CubeMX_UI_select(3)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_UI_select(3).png)



##### 1.1.2、时钟界面

![CubeMX_Config(1)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_Config(1).png)

可以看到，进来后，右边是一个芯片的放大图，四周对应着的是芯片引出来的引脚，我们可以直接点击这些引脚，对其进行配置，相比使用固件库做相关配置时的体验来说是相当的香了。左边可以看到有一排东西，最上面的是 CPU 内核的一些外设配置。

我们要做的第一件事是配时钟，所以直接点击 RCC 。

![CubeMX_Config(2)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_Config(2).png)

多么熟悉的选项，还不用我们自己配置，直接在选最后一个 水晶/陶瓷谐振器 （外部晶振） 即可。

![CubeMX_Config(3)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_Config(3).png)

如果想要微操改动部分总线的时钟频率，可以点击界面上方的 Clock Configuration， 整个时钟树尽收眼底，要改哪直接在图形界面输入相应的频率数值即可，相当方便。当然，本次工程中我们使用默认配置即可。

##### 1.1.3、LED引脚配置

ok，接下来进入到LED引脚配置环节。

因为我使用的是蓝桥杯的板子，这比赛的板子向来有个特点：为了节约引脚，LED灯前面必加一个锁存器。

![Blue_led](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Blue_Led.png)

通过原理图上的写着的锁存器的芯片型号 **SN74HC573ADWR** ，我们可以找到它的芯片手册查看它的真值表：

![image-20211121232320456](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Lock_FunctionTable.png)

可以看到，当  $\overline{OE}$  为低电平时，这个锁存器才有意义。当 $LE$ 处于高电平时，输出电平随 $D$ 变化而变化，而 $LE$ 处于低电平时，就像上锁了一样，无论 $D$ 端输入如何变化，输出的电平都为 $LE$ 为高电平时的电平状态。

有了以上对锁存器的认识，还有使用固件库配置 gpio 的经验，我们就可以开干了。

从上面的原理中可以看到，  $\overline{OE}$ 已经接地，我们只需要通过配置 $PD2$ 对 $LE$ 操作即可。

![Blue_led(1)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Blue_led(1).png)

点击配置，完成后对应引脚显示绿色。

![Blue_led(2)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Blue_led(2).png)

如上对 GPIO 进行操作。流程跟固件库一样，只是原来需要手敲的代码，变成了点点点~

##### 1.1.4、代码生成

在自动代码生成前，我们还需要对设置一下这个工程的一些信息。

![Blue_led(3)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Blue_led(3).png)

如上图：

1. 点击 Project Manager ，进入界面。
2. 选择左侧 Project 栏。
3. 设置 工程名字。
4. 选择工程存放路径。（这里需要注意存放路径中不能出现中文！！！）
5. 选择代码生成的 IDE 环境。
6. 基本不用管，默认值就好。



![Blue_led(4)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Blue_led(4).png)

然后是上图，这里需要注意几点：

1. 点击 Code Generator 可以进入当前界面。
2. 这里是选择导入工程的包文件方式：
   1. 可以全部导入
   2. 导入需要部分
   3. 导入的是本地库的链接，不是拷贝，非常不利于移植工程，不推荐使用
3. 这里强烈建议勾上第一个，不然所有配置代码都堆在 main 函数里，代码会非常难看且难调。

Project Manager 页面还有最后一个子页面：

![Blue_led(5)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Blue_led(5).png)

这里主要是调节函数相关的内容，你可以设置生成代码中函数的调用顺序、函数的属性等。

哦，对了还有一个配置。

![Blue_led(6)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Blue_led(6).png)

看玩野火的教程，我觉得最好把 SYS 中的 Debug 选上 Serial Wire。据说不这么操作的话，烧录一次程序过后程序会烧不进去？具体原因我还没有研究，不过为了减少初学时的麻烦就先照着做吧，反正总比刚开始学灯都还没点亮就出现各种问题影响心情强不是嘛。

好了，晚上以上步骤之后，点击右上方的 GENERATE CODE 即可生成代码。

![Blue_led(7)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Blue_led(7).png)



#### 1.2、工程源码

![Hal_code(1)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Hal_code(1).png)

如上图，打开 main 函数可以看到有上图中框出来的三个函数，其中后两个是我们自己配置的。

![Hal_code(2)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Hal_code(2).png)

进入 GPIO 的初始化函数可以看到，端口使能、GPIO引脚初始化、电平设置 都和我们在 CubeMX 中设置的一模一样。硬要说不同的话可能就是一些 api 的名字有一丢丢差别了。

但是这对我们的使用影响不大，后面照着固件库时的开发方式在keil中把具体功能逻辑实现即可。

### 二、任务创建

#### 2.1、CubeMX设置

任务是RTOS运行的基本单位，一个按键控制可以是一个任务，一个LED闪烁也可以是一个任务。这里，暂时先不刨根问底的让一个任务跑起来。

![CubeMX_RTOS(1)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_RTOS(1).png)

回到 CubeMX 界面，点开 Middle ware 中间件选项，可以看到有个大大的 FreeRTOS 在这，我们进入相关页面后，选择 V1 即可。这里 v1 和 v2 的区别我还没有研究过，不过就先这样吧。

![CubeMX_RTOS(2)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_RTOS(2).png)

选择玩其中一个版本后，我们就可以看到上图的场景（没有 3 的场景）。

1. 点击 Tasks and Queues ，可以看到任务和队列相关项。
2. 可以直接点击 defaultTask 进行设置。
3. 因为我们这里只有一个任务，所以只需要设置任务名 Task Name 和  进入函数 Entry Function 的名字即可。

这时候，基本已经完成了。但是如果这时候直接生成代码的话，会有这么一个弹框：

![CubeMX_RTOS(3)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_RTOS(3).png)

这是因为 Hal 库自身实现了一个 Delay 函数，而这个函数的实现用的定时器是 systick 。systick 这个定时器本身在 os 中起的是一个脉搏的作用具体情况后面会叙述。所以这里我们需要先修改一下 Hal 库延时使用的定时器。

![CubeMX_RTOS(4)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/CubeMX_RTOS(4).png)

这个设置在 sys 里，有个 TimeBase Source的选项，我们换成其他任意一个时钟即可。

#### 2.2、工程源码

![RTOS_code(1)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/RTOS_code(1).png)

打开源码可以看到有 RTOS 的初始化调用函数、系统内核的启动函数，这些我们都不用动它。

![RTOS_code(2)](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/RTOS_code(2).png)

我们需要操作的在 app_freertos.c 这个文件里。点进去后我们可以发现有一个 LedTask 的函数（这个名字就是 CubeMX 中任务设置里的 TaskName）。

我们把 Led 闪烁的功能写进 for 循环里即可。

### 效果图

![Led_photo](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/Embedded/RTOS/Led_photo.png)



