---
title: Cisco-VLAN与单臂路由配置
date: 2020-10-09 00:25:28
tags: cisco
categories: 杂项
---

### 简介

>我们都知道交换机共用一个广播域，当计算机A尝试获得计算机B的MAC地址，建立连接的时候必须先广播ARP请求。
>
>考虑这么一个情况：
>
>- 网络由多个交换机相连
>- 计算机A和计算机B连接在同一个交换机上
>- 计算机A发送ARP请求，尝试与计算机B建立连接
>
>对于以上叙述的情况，由于交换机有泛洪的特性，ARP请求除了向连接在同一交换机计算机B发送外，还会向其他接口发送ARP请求。
>
>显然，这么做会造成资源浪费。为了尽可能减少这种情况的影响，划分一个大的广播域的需求就应运而生了。
>
>而划分广播域用到的技术就是——VLAN(虚拟局域网)。

<!-- more-->

### 一、VLAN配置

VLAN的划分方法有很多种，其中最简单的莫过于基于端口划分。~~(学校目前也只教了这种啊)~~

因本人只想划水不挂科，所以以下讨论皆为基于端口划分的配置。

基于端口划分的VLAN有两种模式：

- Access Link（访问链接）：端口仅允许一个VLAN的数据通过。
- Trunk Link（汇聚链接）：端口仅允许多个VLAN的数据通过。

#### 例1：仅使用access链接

##### 1.1、示例拓扑图

![示例拓扑图](示例拓扑图.png)

##### 1.2、配VLAN前

<img src="配置vlan前.png" alt="配置vlan前" style="zoom: 67%;" />

上图表明当前4台PC处于正常可ping通状态。

##### 1.3、实验步骤

- 在创建VLAN
- 分别进入各个接口
- 配置接口的VLAN模式
- 分配接口给已存在的VLAN

##### 1.3、命令

```
Switch(config)#vlan 10
Switch(config-if)#exit
Switch(config)#vlan 20
Switch(config-if)#exit //创建vlan 10 和 vlan 20

Switch(config)#int f0/1							//进入fa0/0端口
Switch(config-if)#switchport mode access 		//配置端口为access模式
Switch(config-if)#switchport access vlan 10		//把该端口分配给vlan 10
/*
	同理配置其他3个端口
*/

```

##### 1.4、结果

<img src="配置vlan后(1).png" alt="配置vlan后(1)" style="zoom:50%;" />

<img src="配置vlan后(2).png" alt="配置vlan后(2)" style="zoom: 50%;" />

#### 例2：使用trunk连接多台交换机

##### 2.1、示例拓扑

<img src="trunk示例.png" alt="trunk示例" style="zoom:67%;" />

##### 2.2、实验步骤

- 重复例题的步骤
- 交换机连接方式设为trunk

##### 2.3、命令

```
Switch(config)#int f0/24
Switch(config-if)#swit mod trunk			//设置成trunk模式
Switch(config-if)#swit trunk allow vlan 10	//允许vlan 10通过
Switch(config-if)#swit trunk allow vlan add 20	//添加允许vlan 20
/*
	两台交换机都需要配置
*/
```

##### 2.4、查看配置情况（用于排错）

```
Switch#show vlan brief	//查看vlan情况，如下图1
Switch#show running-config
Switch#show startup-config
/*
	show vlan brief 好像不能查看trunk模式的分配情况，
	但我们可以直接查缓存和磁盘看各端口情况。
	如下图2
*/
```

<img src="show-vlan-bri.png" alt="show-vlan-bri" style="zoom: 50%;" />

<img src="show-缓存or磁盘.png" alt="show-缓存or磁盘" style="zoom:50%;" />



***

当用VLAN划分了不同的广播域后，即使是连接在同一交换机的不同vlan的主机也不可以直接进行通信了。这跟一台路由器连接两个不同网络是一个效果。所以想要让不同VLAN间进行通信，自然而然就要通过路由器啦！

根据我们让两个不同网络互联的配置方式，我们可以得到下面这张拓扑图：

![单臂路由引例图](单臂路由引例图.png)

但有时可能网络预算不足或者网络比较小的时候，我们也可以用一根线完成这个操作，那就是使用单臂路由！

简单来说，单臂路由就是通过把路由器的一个接口分成若干个子接口分别配置不同地址从而完成网络互联。

有意思的是，据本人查到的相关资料显示，单臂路由是一种应急技术，但老师貌似很看重。具体原因未知，也许上课提过？Anyway，先搞搞应付完这波再说。

### 二、单臂路由配置

#### 2.1、示例拓扑

<img src="单臂路由拓扑.png" alt="单臂路由拓扑" style="zoom:67%;" />

因该拓扑为同网段不同vlan，子端口ip设置失败（如下图）：<img src="单臂路由拓扑失败图.png" alt="单臂路由拓扑失败图" style="zoom:50%;" />

经询问得知，在cisco中处理这情况需要用到private vlan，HUAWEI中需要用到Mux-vlan。简单来说就是需要对vlan分层级，而学校木有教，抱着水过考试就行的心态，在此不做过多深入研究。所以此情况pass！！！

<img src="单臂路由拓扑简单图.png" alt="单臂路由拓扑简单图" style="zoom:67%;" />

以下配置以上图为准。

#### 2.2、实验步骤

- 在交换机与连接路由器的端口处配置trunk
- 进入路由器子端口
- 配置trunk的封装协议 dot1Q
- 添加子端口IP地址
- 给各PC配上网关地址

#### 2.3、命令

```
//路由器端
Router(config)#int g0/0/0.1						//进入.1子端口
Router(config-subif)#encapsulation dot1Q 10		//设置封装协议为vlan 10的
Router(config-subif)#ip address 192.168.1.254 255.255.255.0
Router(config-subif)#no shutdown
Router(config-subif)#exit
//下面同上
Router(config)#int g0/0/0.2
Router(config-subif)#encapsulation dot1Q 20		
Router(config-subif)#ip address 192.168.2.253 255.255.255.0
Router(config-subif)#no shutdown
//交换机端配置同上trunk例子，此处忽略

```

#### 2.4、结果

![单臂路由结果](单臂路由结果.png)

#### 2.5、查看配置情况

查看配置情况同上**VLAN配置的2.4**，在此不再叙述。

关于单臂路由排错，一般只需看两处：

- 三层设备：网关没有配
- 二层设备：vlan分配出问题

**（PS：同网段不同vlan不在上述描述情况讨论范围！！！）**
