---
title: 【初入电子坑之stm32篇（一）】初识stm32与寄存器编程
mathjax: true
date: 2021-1-4 13:49:19
tags: [stm32]
categories: [嵌入式,MCU]
---

## 前言

> 以下为本人基于野火stm32F103EVT6开发板学习stm32的学习笔记。

<!--more-->

## 导图概览

![初识stm32概览](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E5%88%9D%E8%AF%86stm32%E6%A6%82%E8%A7%88.png)

## 初识32

### 1、下载方式

把单片机耍起来，除了要写程序外，当然还得把程序下载到单片机的上啦！而单片机一般有以下两种下载程序方式：

- 仿真器下载
- ISP下载

曾经玩过简单玩过一下下51和arduino。这两个硬件平台玩的时候都对下载方式进行了一定的简化，所以直到现在我才知道下载方式也是有讲究的orz。

#### 仿真器下载

所谓仿真器下载就是，下载程序的时候，在片外连接一个叫仿真器的东西。连上这个东西之后可以实时烧录程序观察板子的现象，无需按reset按键，并可以单步跟踪调试观察实时现象（跟写纯软件的单步调试是一个意思）。

虽然目前木有入手仿真器，不过以前简单玩过的板子内集成了仿真器的功能，所以也算体验过了。

#### ISP下载

全称：In-System Programmability，即：在系统可编程。

一开始我看到这名字的时候有点蒙蔽，ISP下载？啥玩意儿，网络运营商下载？查过资料才知道，在很久很久之前~~~老前辈们烧录程序时居然还要把芯片取下来，拿到专门烧录程序的机器上烧录程序，然后再插会电路板上，十分麻烦。而ISP下载，就是现在习以为常的在板上插根线，直接连接电脑烧录程序的方式。

在目前“以实现功能为主，不深究具体实现。”的学习思想指导下，对于这东西，没啥需要注意的。了解一下，表达下对前辈们的敬意即可。

在这里有个小建议：**在缺乏相关电路知识的情况下，不建议头铁的去研究ISP电路实现，那么做的话就有点偏离我们现在的主要目的了。**

### 2、芯片选型

即使是同一款芯片,也有很多不同的型号。它们的差别体现在引脚数、晶振频率、flash、可用外设的差别上，虽然对于个人学习、实验来说只要功能相对够齐全一般都没啥问题。但在DIY项目、工业生产上，每bit都要尽量抠着用。（多一毛钱，乘于1w、100w都是好多好多钱啊！！！）

所以简单了解下，还是很有必要的。

对了，上面提到的“引脚”，可以简单理解为黑黑的芯片周围那很多根的白色金属。

#### stm32命名规则

![stm32命名规则](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/stm32%E5%91%BD%E5%90%8D%E8%A7%84%E5%88%99.png)

7部分：

- STM32：ST公司生产的Cortex-M内核的32位微控制器
- F：芯片的基础型子系列。还有其他的诸如：S标准型、L超低功耗等。
- 103：跟上面类似。
- V：引脚数代号，V表示100引脚。
- E：flash大小代号，E表示512KB。
- T：芯片封装种类。
- 6：芯片适应温度等级。

以上信息做到大概了解即可。有具体需求时，看官方的选型手册即可。

### 3、系统结构

![stm32芯片架构](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/stm32%E8%8A%AF%E7%89%87%E6%9E%B6%E6%9E%84.png)

上图为stm32的芯片架构简图。其中大概可以分由ARM公司设计的Cortex-M3内核，和由ST公司以Cortex-M3内核为基础设计的“外围结构”。（这里所指的外围，是相对于内核而言的。）

而可以进行的绝大部分操作，都集中在外设的寄存器上，通过操作外设寄存器的值，从而控制芯片I/O引脚的输入输出，从而来完成我们需要的操作。

关于更多的寄存器的相关知识，可参考我的汇编学习笔记：

{% post_link 汇编语言(一)基础知识与寄存器 汇编语言(一)基础知识与寄存器 %}

（还没填完坑。。。）

![stm32系统框图](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/stm32%E7%B3%BB%E7%BB%9F%E6%A1%86%E5%9B%BE.png)

上图为stm32的系统结构框图，其中AHB系统总线（蓝色方框4那）连接着外设。换句话说，我们可以通过AHB总线，进行最大限度的自由发挥，是可以为所欲为的地方。

AHB通过桥接的方式分出APB1、APB2两条外设总线，它们分别挂载着一些外设，其中APB1的访问速率比APB2慢一倍。（PS：为什么会慢一倍，在后面《理解时钟系统》中会解释，别钻牛角尖哈。。。）

在ST公司的《STM32F10x-中文参考手册》，我们可以找到AHB的存储器映射地址表，如下图：

![AHB地址映射](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/AHB%E5%9C%B0%E5%9D%80%E6%98%A0%E5%B0%84.png)

通过这个表，我们可以找到所有外设寄存器的地址，设计程序改变相关寄存器的值，从而完成我们想要的控制。

## 入门运用

### 0、启动文件

所有单片机都有这么一个东西，从运用的角度来看，我们无需深究它，只需对它的作用有个大概的感性认知即可。它的作用如下：

- 在单片机上电之后，使用汇编指令对内核进行一些必要的初始化。
- 利用汇编指令在stm32芯片上搭建了一个c语言的运行环境。

（如果你想深挖这大哥的大概运用原理，这边建议你去刷一遍刘爽的《汇编语言》）

### 1、GPIO

#### 简介

**GPIO**（英语：General-purpose input/output），功能类似8051的P0—P3。不同的是，这哥们比较高级，下面放张它的结构图，让大家感受下它的“高级”：

![GPIO端口结构](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/GPIO%E7%AB%AF%E5%8F%A3%E7%BB%93%E6%9E%84.png)

在51里，P0——P3的引脚我们可以直接修改它们对应的输出数据寄存器的值从而控制I/O口的输出。但从上图我们可以看到，进行同样的操作，用GPIO我们需要操作多个寄存器，才可以修改输出数据寄存器ODR内的数据。

当然啦，既然操作相对繁琐辣么多，必然有它的意义所在——这哥们有8个输入/输出模式：

##### 输入：

- 输入浮空
- 输入上拉
- 输入下拉
- 模拟输入

##### 输出：

- 开漏输出
- 推挽输出
- 开漏复用功能
- 推挽复用功能

我曾经很舍本逐末的探究了上面几种输入输出模式背后的原理，然鹅。。。发现由于认知层面的缺失，即使勉强研究明白了几分，也没什么卵用。。。

所以还是先把它用起来吧，其他的以后再说。。。

#### 控制流程

想要令GPIO的I/O口输入/输出值，要进行以下流程：

1. 开启对应端口的时钟使能端（为了节省功耗，外设的使能默认关闭。时钟使能可简单理解为是一个控制开关）
2. 选择引脚，并配置对应的IO模式、若是输出模式还需选择输出速率。
3. 控制端口寄存器CRL/CRH清零
4. 两种情况：
   1. 输出模式：设置引脚(即：数据输出/输入寄存器ODR)的值
   2. 输入模式：读取引脚对应引脚信息，以便加以利用（如开关）

由以上对GPIO的认知，我们可以引申出两种在stm32上不同的编程方式：

- 基于寄存器编程
- 基于固件库编程

下面以点亮LED灯为例，进行分析。

### 2、基于寄存器编程

基于寄存器编程其实又可以叫**基于数据手册编程**，因为它很简单粗暴。

![野火LED模块](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E9%87%8E%E7%81%ABLED%E6%A8%A1%E5%9D%97.png)

上面是野火指南者开发板的LED原理图。可以看到这是个三原色LED灯，通过PB1、PB0、PB5端口输出低电平就可以让LED发出对应的灯光颜色。

根据上一小节中，我们分析控制GPIO端口的流程。我们先从官方的数据手册分别找出对应的寄存器地址。

![RCC基地址](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/RCC%E5%9F%BA%E5%9C%B0%E5%9D%80.png)

![APB2使能控制地址](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/APB2%E4%BD%BF%E8%83%BD%E6%8E%A7%E5%88%B6%E5%9C%B0%E5%9D%80.png)

![GPIOB基地址](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/GPIOB%E5%9F%BA%E5%9C%B0%E5%9D%80.png)

从而得出Level1版的点灯程序：

#### Level_1：无脑表示外设寄存器地址

##### 宏定义：

~~~c
	typedef unsigned int uint;
	#define RCC_APB2ENR *(uint*)0x40021018
	#define GPIOB_CRL  *(uint*)0x40010C00
	#define GPIOB_ODR *(uint*)0x40010C0C
~~~

##### 主函数：

~~~c
	/***********初始化*************/
	/*
			1、开启GPIO端口时钟 
			2、端口清零
			3、配置输入/输出模式、输出速率
	*/
	//开启端口时钟
	RCC_APB2ENR |= (1<<3);
	
	//清零控制PB0的端口位
	GPIOB_CRL &= ~( 0x0f );
	//配置PB0为通用推挽输出，速度为10M
	GPIOB_CRL |= 1;
	/*********控制引脚电平输出*********/
	//PB0输出低电平
	GPIOB_ODR &= ~(1);
~~~

上面这段代码值得说道的是，使用无符号整形指针对十六位地址进行强制类型转换，使一个十六进制数变成了地址标号。然后就顺理成章的在它前面加个“*”解引用，对那块十六进制数表示的地址赋值。手段很妙，学到了，orz。。。

```c
(uint*)0x40010C0C;		//十六进制数地址化
*(uint*)0x40010C0C;		//解引用
```

但以上做法过于简单粗暴，每个寄存器都用一个8位十六进制表示，写起来麻烦不说，还容易出错且调试麻烦，代码的维护性比较困难。所以很自然的引申出level2版，基于基址+偏址的宏定义版本。

***

#### Level_2: 基于基址+偏址 

我们可以看到，内存地址映射表的地址是分布是有规律的。

![image-20210422210004917](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/image-20210422210004917.png)

从层次上，外设的起始地址是什么，属于哪条总线从表中可以看得一清二楚。

从顺序上，因为地址的连续性，所有外设的地址有连在一起有迹可循。

基于以上两点，我们就可以用一个地址作为基础地址（比如这里的0x4000 0000），然后根据各个总线起始地址与基础地址的差值，作为偏移量；外设的起始地址与外设所在的总线起始地址的差值作为偏移量。由此形成了多层级宏定义 的**基址+偏址**模式。

 基址+偏址的方法的好处是，每次定义寄存器地址的时候都可以借用之前定义的宏，写的时候更加便于理解且调试比Level1方便不要太多。

##### 宏定义：

```c
typedef unsigned int uint;

//外设宏定义 peripheral
#define PERIPH_BASE  			(uint)0x40000000
#define APB2PERIPH_BASE  		PERIPH_BASE+(0x10000)
#define AHBPERIPH_BASE  		PERIPH_BASE+(0x20000)	//时钟控制

//定义时钟复位寄存器和GPIOB的基地址
#define RCC_BASE 		(AHBPERIPH_BASE+0X1000)
#define GPIOB_BASE		(APB2PERIPH_BASE + 0X0C00)

#define RCC_APB2ENR		*(uint*)(RCC_BASE + 0X18)	//apb2总线使能地址

#define GPIO_CRL		*(uint*)(GPIOB_BASE + 0X00)
#define	GPIO_CRH		*(uint*)(GPIOB_BASE + 0X04)
#define GPIO_ODR		*(uint*)(GPIOB_BASE + 0X0C)
```

##### 主函数：

同 level1。

本人用Level_2的方式耍了两个多小时，发现GPIO的宏定义都要加上同一个偏址且每次初始化GPIO端口干的是都差不多。这就导致产生了很多冗余代码，这能忍吗？必然不能！

因为每个GPIO口内部的地址都是规律连续的。所以很容易引申出Level_3——结构体与函数封装

***

#### Level_3：初步封装

##### 头文件：

~~~c
typedef unsigned int uint32_t;
typedef unsigned short uint16_t;

/********************总线基址编号***********************/
#define PERIPHER_BASE           ((uint32_t)0x40000000)
#define PERIPHERAHB_BASE        (PERIPHER_BASE + 0x20000)
#define PERIPHERAPB2_BASE       (PERIPHER_BASE + 0x10000)

/*****************GPIO基址and时钟基址编号*********************/
#define GPIOA_BASE              (PERIPHERAPB2_BASE + 0x0800 )
#define GPIOB_BASE              (PERIPHERAPB2_BASE + 0x0C00 )
#define RCC_BASE                (PERIPHERAHB_BASE  + 0x1000 )

/****************结构体设置*********************/
//GPIO
typedef struct 
{
    uint32_t CRL;
    uint32_t CRH;
    uint32_t IDR;   
    uint32_t ODR;
    uint32_t BSRR;
    uint32_t BRR;
    uint32_t LCKR;

}GPIO_TypeDef;
//RCC
typedef struct
{
    uint32_t CRR_CR;
    uint32_t CRR_CFGR;
    uint32_t RCC_CIR;
    uint32_t RCC_APB2RSTR;
    uint32_t RCC_APB1RSTR;
    uint32_t RCC_AHBENR;
    uint32_t RCC_APB2ENR;
    uint32_t RCC_APB1ENR;
    uint32_t RCC_BDCR;
    uint32_t RCC_CSR;
}RCC_TypeDef;

/***************编号转化为地址****************/
#define GPIOA                   ((GPIO_TypeDef*) GPIOA_BASE) 
#define GPIOB                   ((GPIO_TypeDef*) GPIOB_BASE)     
#define RCC                     ((RCC_TypeDef* ) RCC_BASE)

/***********************GPIO引脚*************************/
    /*
        以下宏定义一般用于IDR or ODR的配置，所以一般16位即可
    */
    #define GPIO_PIN_0              ((uint16_t)0x0001)
    #define GPIO_PIN_1              ((uint16_t)0x0002)
    #define GPIO_PIN_2              ((uint16_t)0x0004)
    #define GPIO_PIN_3              ((uint16_t)0x0008)
    #define GPIO_PIN_4              ((uint16_t)0x0010)
    #define GPIO_PIN_5              ((uint16_t)0x0020)
    #define GPIO_PIN_6              ((uint16_t)0x0040)
    #define GPIO_PIN_7              ((uint16_t)0x0080)

    #define GPIO_PIN_8              ((uint16_t)0x0100)
    #define GPIO_PIN_9              ((uint16_t)0x0200)
    #define GPIO_PIN_10             ((uint16_t)0x0400)
    #define GPIO_PIN_11             ((uint16_t)0x0800)
    #define GPIO_PIN_12             ((uint16_t)0x1000)
    #define GPIO_PIN_13             ((uint16_t)0x2000)
    #define GPIO_PIN_14             ((uint16_t)0x4000)
    #define GPIO_PIN_15             ((uint16_t)0x8000)
    #define GPIO_PIN_ALL            ((uint16_t)0xffff)
 /*************************RCC引脚*************************/
	#define RCC_GPIOA             ((uint32_t)0x00000004)
	#define RCC_GPIOB             ((uint32_t)0x00000008)
	#define enum{ ENABLE = 0, UNENABLE = 1}State;

 /*******************初始化变量枚举定义***********************/
    typedef enum{
        GPIO_SPEED_10MHz = 1,
        GPIO_SPEED_20MHz ,
        GPIO_SPEED_50MHz ,
    }GPIOSPEED_TypeDef;

    typedef enum{
        GPIO_MODE_AIN = 0x0,
        GPIO_MODE_AIN = 0x4,
        GPIO_MODE_AIN = 0X28,
        GPIO_MODE_AIN = 0x48,

        GPIO_MODE_AIN = 0x14,
        GPIO_MODE_AIN = 0x10,
        GPIO_MODE_AIN = 0x1C,
        GPIO_MODE_AIN = 0x18
    }GPIOMODE_TypeDef;

	/******************GPIO初始化结构体******************/
    typedef struct 
    {
        uint16_t Pin;
        GPIOSPEED_TypeDef Speed;
        GPIOMODE_TypeDef Mode;
    }GPIO_InitTypeDef;

	/***********************相关库函数**************************/
    void RCC_InitConfig(RCC_TypeDef*RCC_Def,State NewState);		
    void GPIOx_Setbit(GPIO_TypeDef*GPIOx,uint16_t Pin);
    void GPIOx_Retbit(GPIO_TypeDef*GPIOx,uint16_t Pin);
    void GPIOx_Cofig(GPIO_TypeDef*GPIOx,GPIO_InitTypeDef* GPIO_Init);
~~~

做完以上的工作，就可以对初步的对GPIO为所欲为了。

至于上面4个库函数的实现分析，请移步以下链接：【链接】（此坑待填。。。）

##### 主函数：

~~~c
int main(void){
    GPIO_InitTypeDef GPIO_INIT;
    /***********时钟配置************/
	RCC_InitConfig(LED_CLK,ENABLE);

    /***********GPIO初始化**********/
    GPIO_INIT.GPIO_Pin 	 = GPIO_PIN_0;
	GPIO_INIT.GPIO_Mode	 = GPIO_Mode_Out_PP;
	GPIO_INIT.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB,&GPIO_INIT);
	
	while(1){
		GPIOx_Setbit(GPIOB,GPIO_PIN_0);
	 }
}

~~~

至此基于寄存器编程的基本操作原理算是结束了。

#### 总结：

纯寄存器编程而言应该是Level_2的阶段，Leve_3因为封装使用了函数调用，比只用寄存器而言消耗的cpu资源必然会更多。不过只要不是对性能苛刻到相当高的程度的话，这点开销就没必要计较了。

总的来说，寄存器编程开发遵从以下步骤：
- 看着数据手册，先把相关的寄存器映射、引脚顺序定义
- 根据需求找到对应的寄存器
- 利用定好的引脚去改变寄存器的值

#### 寄存器编程的意义

- 开销低，抠性能的时候就是它发挥威力的时候。
- 当芯片商家没有提供相关底层库时可以自食其力（以后不可能只用st公司的芯片不是）
- 用库时做到心中有数，出bug时可以有的放矢（不要过分相信官方库）。
