---
title: 【初入电子坑之stm32篇（四）】理解时钟系统
mathjax: true
date: 2021-01-17 16:11:53
tags: stm32
categories: [嵌入式,MCU]
---

##  前言

> 时钟系统是MCU必不可少的一部分。本文将针对以下问题展开讨论：
>
> - 什么是时钟？
> - 如何理解stm32时钟结构？
> - 为什么MCU会有多个时钟源？
> - stm32如何配置时钟？
>
> 当然，由于本人学识不足，且时钟略复杂，所以这里只能说是对时钟有一个感性的初步理解，如有错误，还望不吝指正。

<!--more-->

## 目录

- 一、时钟简介
- 二、stm32时钟树
- 三、关于时钟的一些思考
- 四、时钟配置分析
- 五、总结

## 一、时钟简介

首先我们得搞清楚几个概念：

- 时钟：单片机的心脏，所有外设的运作都需要时钟供能。
- 时钟周期：又称为振荡周期，可以简单理解为传输一个 0 or 1 所需要的时间。
- 指令周期：执行一条指令（如：MOV A，#34H）所需的时间。对于不同类型的指令，指令周期长度可能会不同。
- 机器周期：执行一个动作的时间周期。如：执行一个指令需要 “取指令并驿码”、"执行操作数"两个动作。

以上三个周期的关系图如下：

<img src="https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E7%90%86%E8%A7%A3%E6%97%B6%E9%92%9F.png" alt="理解时钟" style="zoom:80%;" />

（其中蓝色部分包括了四个机器周期）

而因为时钟信号也是电信号的一种，所以它还兼具了供能的作用，且因电平变化的时间间隔一定，我们甚至可以用时钟来计时。

（PS：经老师纠正，时钟“供能”的说法是错的，时钟并不起供能作用。不过个人认为，从初学者角度来看，这样感性理解问题也不大，只要记得这种说法是不对的即可。）

## 二、stm32时钟树

有了以上的感观认识后，我们先来看看stm32f10x的时钟树：

![stm32时钟树](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/stm32%E6%97%B6%E9%92%9F%E6%A0%91.png)

### 2.1、stm32f10x时钟源

上图蓝色部分标出了stm32的5个时钟源，分别是：
- HSI：高速内部时钟（High speed internal），由内部RC振荡器产生，频率为8MHz。
- HSE：高速外部时钟（High speed external），可外接4-16MHz的晶振作为时钟源。
- LSE：低速外部时钟，外接32.768kHz时钟源。
- LSI：低速内部时钟，由内部RC振荡器产生，频率为32kHz。
- PLL：锁相环倍频输出。可以做到输入时钟的2-16倍的倍频输出。

这里把PLL也列为时钟源是参考了诸如野火资料的说法，个人觉得实际的时钟源只能算4个，因为PLL本身只起倍频作用，本身并不能产生时钟频率。当然，这些都是人为定义的东西，不影响使用和对该知识点的理解。

（PS：RC振荡器产生的时钟精度相对较低。）

### 2.2、系统时钟

知道有哪些时钟源后，我们可以发现时钟树是我们熟悉的AHB、APB1、APB2总线。

时钟源与这些外设之间都通过红色部分的**系统时钟SYSCLK**连接在一起，由此观之这个**系统时钟SYSCLK**很重要！！！

既然它辣么重要，我们就先对它分析一波啦！

可以看到，系统时钟有三个输入源：
- HSI：8MHz
- PLLCLK：8-128MHz(因为外接晶振一般为8MHz)
- HSE：4-16MHz

而SYSCLK可以接受的最大频率是72MHz，这也是后面所有外设可以正常运作的频率。

可以看到，想要达到这个数字，只能通过PLL倍频得到（不止得到，你甚至可以用它超频）。

对了，SYSCLK选择还连着个叫CSS的东西，查下数据手册，得到以下描述：

![时钟安全系统](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E6%97%B6%E9%92%9F%E5%AE%89%E5%85%A8%E7%B3%BB%E7%BB%9F.png)

可以看到，这东西简单来说就是个报警器。。。帮你检测HSE有没有出问题，有的话就先切到内部时钟源，然后给出相关提示信息。

分析到这里，我们对stm32的时钟树就可以有个相对清晰的认知了。

下面补充一下没有提到的点~~（主要是我没想到其他相对较散的知识点怎么用逻辑串起来）~~：

- 低速内外部时钟应用
- MCO时钟输出：用于示波器观察时钟信号，调试所用。

## 三、关于时钟的一些思考

### 3.1、时钟频率

从2.1和2.2的讨论中，我们可以发现，系统时钟正常工作的时钟频率是72MHz。但实际输入的系统时钟频率最高到可以到128MHz，最低8MHz，这意味着我们可以不遵循数据手册的规定，做点"违规操作"。

“违规操作”会怎样？由于目前只会点灯，而且LED这哥们对时钟没啥要求。。。基本上改变频率也就是亮灭快慢不同的事，看不出啥，于是我又去现查了下资料：~~（本菜鸡最近在补数电了 TAT ）~~

[时钟频率是个什么概念？？ - 虞己某的回答 - 知乎 ](https://www.zhihu.com/question/29685396/answer/145507426)

从这个回答中，我们大概可出一下结论：
- 频率低了，不能满足设备的频率要求的话，可能无法启动设备。
- 频率高了，系统稳定性会受影响。具体表现是进行通信的时候，外设可能无法接受到准确的信号。

### 3.2、多个时钟源

刚了解时钟树的时候，我第一反应是——为什么有那么多时钟源？

既然外设都是受系统时钟管理，那么只用一个时钟不是就够了嘛？

基于以上问题，翻阅学习了相关资料，得出了两个最显而易见的结论：

- 多个时钟可以在单个时钟源发生故障时，起到救急的作用。
- 一个外设有多个时钟源，可以根据需要选择相应频率的时钟源。

## 四、时钟配置分析

stm32的时钟树理解后，就是时钟配置问题了。

实际上，当我们创建工程导入启动文件的时候，在main函数开始调用前，启动文件就已经调用了SystemInit函数对系统时钟进行初始化了。

![启动文件&时钟](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E5%90%AF%E5%8A%A8%E6%96%87%E4%BB%B6&%E6%97%B6%E9%92%9F.png)

那为什么还需要做配置分析？直接拿来用不就好了嘛？

我们且不论其它芯片如何，单就看stm32的固件库给我们提供的库函数（下图为system_stm32f10x.c的截图）：

![库函数系统时钟设置](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/%E5%BA%93%E5%87%BD%E6%95%B0%E7%B3%BB%E7%BB%9F%E6%97%B6%E9%92%9F%E8%AE%BE%E7%BD%AE.png)

人家ST官方只提供了24MHz，36MHz，48MHz，56MHz，72MHz五种频率的系统时钟设置。

如果项目对时钟有这5种频率外的需求还是得自己动手。基于以上原因，对时钟配置的分析就是刚需了。

![stm32时钟树](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/stm32%E6%97%B6%E9%92%9F%E6%A0%91.png)

那么，先让我们对着时钟树的图再梳理一遍系统时钟的来龙去脉。

#### 输入源：

- HSI直接输出，频率只有8MHz。
- HSE直接输出，范围4-16MHz，一般为8MHz。
- PLL：锁相环倍频输出，倍频范围是2-16倍。且PLL自身有两个输入源：
  - HSI 二分频输出至PLL，这种方式的系统时钟最高可至64MHz。
  - HSE直接输出至PLL。


#### 输出源：

- 直接输出。
- 输出至AHB总线，经预分频器可做 1~512 分频。
- AHB总线预分频后，经低速外设总线APB1和高速总线APB2。

因为考虑到功耗的问题，外设只有用到的时候才会开启时钟，所以SYSCLK只需要初始化总线时钟即可。

基于这个原因，对于系统时钟配置的流程大概可以总结为以下步骤：

1. 选择并开启时钟源（使用PLL作为系统时钟源还需要选择倍频）。
2. 选择APB1、APB2的预分频。
3. 选择AHB预分频。

为什么不是照着图从左到右的顺序配置？

答：如果AHB先开启，而APB1、APB2未设置的话会出现紊乱。~~（看完源码后推测的）~~


下面以库函数 **SetSysClockTo72()**为示例，搭配数据手册学习配置过程，先把源码贴上：

```c
static void SetSysClockTo72(void)
{
  __IO uint32_t StartUpCounter = 0, HSEStatus = 0;
  
  /* SYSCLK, HCLK, PCLK2 and PCLK1 configuration ---------------------------*/    
  /* Enable HSE */    
  RCC->CR |= ((uint32_t)RCC_CR_HSEON);
 
  /* Wait till HSE is ready and if Time out is reached exit */
  do
  {
    HSEStatus = RCC->CR & RCC_CR_HSERDY;
    StartUpCounter++;  
  } while((HSEStatus == 0) && (StartUpCounter != HSE_STARTUP_TIMEOUT));

  if ((RCC->CR & RCC_CR_HSERDY) != RESET)
  {
    HSEStatus = (uint32_t)0x01;
  }
  else
  {
    HSEStatus = (uint32_t)0x00;
  }  

  if (HSEStatus == (uint32_t)0x01)
  {
    /* Enable Prefetch Buffer */
    FLASH->ACR |= FLASH_ACR_PRFTBE;

    /* Flash 2 wait state */
    FLASH->ACR &= (uint32_t)((uint32_t)~FLASH_ACR_LATENCY);
    FLASH->ACR |= (uint32_t)FLASH_ACR_LATENCY_2;    

 
    /* HCLK = SYSCLK */
    RCC->CFGR |= (uint32_t)RCC_CFGR_HPRE_DIV1;
      
    /* PCLK2 = HCLK */
    RCC->CFGR |= (uint32_t)RCC_CFGR_PPRE2_DIV1;
    
    /* PCLK1 = HCLK */
    RCC->CFGR |= (uint32_t)RCC_CFGR_PPRE1_DIV2;

#ifdef STM32F10X_CL
    /* Configure PLLs ------------------------------------------------------*/
    /* PLL2 configuration: PLL2CLK = (HSE / 5) * 8 = 40 MHz */
    /* PREDIV1 configuration: PREDIV1CLK = PLL2 / 5 = 8 MHz */
        
    RCC->CFGR2 &= (uint32_t)~(RCC_CFGR2_PREDIV2 | RCC_CFGR2_PLL2MUL |
                              RCC_CFGR2_PREDIV1 | RCC_CFGR2_PREDIV1SRC);
    RCC->CFGR2 |= (uint32_t)(RCC_CFGR2_PREDIV2_DIV5 | RCC_CFGR2_PLL2MUL8 |
                             RCC_CFGR2_PREDIV1SRC_PLL2 | RCC_CFGR2_PREDIV1_DIV5);
  
    /* Enable PLL2 */
    RCC->CR |= RCC_CR_PLL2ON;
    /* Wait till PLL2 is ready */
    while((RCC->CR & RCC_CR_PLL2RDY) == 0)
    {
    }
    
   
    /* PLL configuration: PLLCLK = PREDIV1 * 9 = 72 MHz */ 
    RCC->CFGR &= (uint32_t)~(RCC_CFGR_PLLXTPRE | RCC_CFGR_PLLSRC | RCC_CFGR_PLLMULL);
    RCC->CFGR |= (uint32_t)(RCC_CFGR_PLLXTPRE_PREDIV1 | RCC_CFGR_PLLSRC_PREDIV1 | 
                            RCC_CFGR_PLLMULL9); 
#else    
    /*  PLL configuration: PLLCLK = HSE * 9 = 72 MHz */
    RCC->CFGR &= (uint32_t)((uint32_t)~(RCC_CFGR_PLLSRC | RCC_CFGR_PLLXTPRE |
                                        RCC_CFGR_PLLMULL));
    RCC->CFGR |= (uint32_t)(RCC_CFGR_PLLSRC_HSE | RCC_CFGR_PLLMULL9);
#endif /* STM32F10X_CL */

    /* Enable PLL */
    RCC->CR |= RCC_CR_PLLON;

    /* Wait till PLL is ready */
    while((RCC->CR & RCC_CR_PLLRDY) == 0)
    {
    }
    
    /* Select PLL as system clock source */
    RCC->CFGR &= (uint32_t)((uint32_t)~(RCC_CFGR_SW));
    RCC->CFGR |= (uint32_t)RCC_CFGR_SW_PLL;    

    /* Wait till PLL is used as system clock source */
    while ((RCC->CFGR & (uint32_t)RCC_CFGR_SWS) != (uint32_t)0x08)
    {
    }
  }
  else
  { /* If HSE fails to start-up, the application will have wrong clock 
         configuration. User can add here some code to deal with this error */
  }
}
```

第一眼看上去很长，寄存器也很多，但是米有关系。先把出现的寄存器记下来：

- RCC->CR
- RCC->CFGR
- RCC->CFGR2
- FLASH->ACR

然后干嘛？当然是去查查手册这些寄存器是干嘛用的呀！由于还是小萌新阶段，只需要了解上面每个寄存器大概的作用结合源码学习即可，手册暂时不必深究。

![RCC_CR](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/RCC_CR.png)

![RCC_CFGR](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/RCC_CFGR.png)

![FLASH_ACR](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/FLASH_ACR.png)



结合以上认识，下面我们来一段一段分析源码：

### 1、开始HSE时钟

```c
/* SYSCLK, HCLK, PCLK2 and PCLK1 configuration ---------------------------*/    
  /* Enable HSE */    
  RCC->CR |= ((uint32_t)RCC_CR_HSEON);//HSEON -> HSE_ON
```

### 2、等待HSE就绪

如果在一定时间内等不到就退出作异常处理。

~~~c
  /* Wait till HSE is ready and if Time out is reached exit */
  do
  {
    HSEStatus = RCC->CR & RCC_CR_HSERDY;//HSERDY -> HSE_Ready
    StartUpCounter++;  
  } while((HSEStatus == 0) && (StartUpCounter != HSE_STARTUP_TIMEOUT));
/** HSE如果就绪，HSEStatus = 1 **/

/** 如果HSE_Ready = RESET，即:HSE没有准备就绪 **/
  if ((RCC->CR & RCC_CR_HSERDY) != RESET)
  {
    HSEStatus = (uint32_t)0x01;
  }
  else
  {
    HSEStatus = (uint32_t)0x00;
  }  
~~~

### 3、HSE状态正常就继续，异常就立刻处理

状态异常的话，经条件语句跳转到下图的模块处理异常。

![HSE状态判断](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/HSE%E7%8A%B6%E6%80%81%E5%88%A4%E6%96%AD.png)

#### 3.1、HSE状态正常情况分析

##### 3.1.1、等待机器取指令

~~~c
    /* Enable Prefetch Buffer */
    FLASH->ACR |= FLASH_ACR_PRFTBE;

    /* Flash 2 wait state */
    FLASH->ACR &= (uint32_t)((uint32_t)~FLASH_ACR_LATENCY);
    FLASH->ACR |= (uint32_t)FLASH_ACR_LATENCY_2;
~~~

##### 3.1.2、设置AHB、AP1、AP2的预分频因子

```c
    /* HCLK = SYSCLK */
    RCC->CFGR |= (uint32_t)RCC_CFGR_HPRE_DIV1;
      
    /* PCLK2 = HCLK */
    RCC->CFGR |= (uint32_t)RCC_CFGR_PPRE2_DIV1;
    
    /* PCLK1 = HCLK */
    RCC->CFGR |= (uint32_t)RCC_CFGR_PPRE1_DIV2;
```

##### 3.1.3、选择PLL的时钟源

源码里这段其实还有二三十行代码，不过那是互联型芯片的配置代码，这里f103是基础型，所以先暂时略过不看。

```c
    /*  PLL configuration: PLLCLK = HSE * 9 = 72 MHz */
    RCC->CFGR &= (uint32_t)((uint32_t)~(RCC_CFGR_PLLSRC | RCC_CFGR_PLLXTPRE |
                                        RCC_CFGR_PLLMULL));

	/*		HSE 与上 PLLMULL9 做到 72MHz的系统时钟输出		*/
    RCC->CFGR |= (uint32_t)(RCC_CFGR_PLLSRC_HSE | RCC_CFGR_PLLMULL9);
```

##### 3.1.4、开启PLL时钟并等待就绪

```c
/* Enable PLL */
    RCC->CR |= RCC_CR_PLLON;

    /* Wait till PLL is ready */
    while((RCC->CR & RCC_CR_PLLRDY) == 0)
    {
    }
```

##### 3.1.5、选择PLL作为系统时钟源

```c
 /* Select PLL as system clock source */
    RCC->CFGR &= (uint32_t)((uint32_t)~(RCC_CFGR_SW));
    RCC->CFGR |= (uint32_t)RCC_CFGR_SW_PLL;    

    /* Wait till PLL is used as system clock source */
    while ((RCC->CFGR & (uint32_t)RCC_CFGR_SWS) != (uint32_t)0x08)
    {
    }
```

以上。

## 五、总结

### 5.1、配置时钟的流程

- 开启时钟并等待就绪
- 若时钟开启异常就退出（可做相关异常处理）
- 开启正常，先配置相关总线的预分频因子
- 最后选择系统时钟，让外设得到能源供应

### 5.2、学习一款MCU的时钟流程

- 看时钟树——分析时钟源的来龙去脉
- 查数据手册，找到以下问题的答案：
  - 时钟源怎么开启？
  - 相关外设的时钟怎么控制？
  - 系统时钟源怎么选择？

## 参考资料

- [时钟频率是个什么概念？？ - 虞己某的回答 - 知乎 ](https://www.zhihu.com/question/29685396/answer/145507426)
- [【心得篇】指令周期、机器周期和时钟周期](https://zhuanlan.zhihu.com/p/97987044)
- [初涉STM32之浅谈时钟使能问题 --- 理解](https://zhuanlan.zhihu.com/p/77053601)
- [一篇文章，彻底搞懂单片机时钟架构！](https://zhuanlan.zhihu.com/p/250022175)
- [STM32时钟树分析](https://blog.csdn.net/bulebin/article/details/73433677)

