---
title: Cisco Telnet与SSH配置
date: 2020-10-09 00:18:03
tags: cisco
categories: 杂项
---

### 简介

> 对于诸如像路由器、交换机、服务器这种东西，如果要亲身待在机器面前做相关配置未免过于不便。所以远程访问控制就成了刚需。
>
> 而远程访问的常用协议有Telnet和SSH两个，它们的区别可以简单的理解为：
>
> Telnet在传输过程中直接以明文传输，简单粗暴。
>
> SSH考虑到安全性问题，传输经过加密，相对Telnet而言消耗多了一丢丢计算资源。

<!--more-->

具体原理上在此不多展开，有以上认知就够了。不过简单查了一下资料，因为安全性问题，现在几乎所有场景都用SSH了，为啥还教Telnet，雾。

好了牢骚发完，下面正式进入到配置环节。

### Telnet配置

#### 一、路由器

##### 1.1、示例拓扑图

<img src="Telnet路由器拓扑图.png" alt="Telnet路由器拓扑图" style="zoom:80%;" />

##### 1.2、实验步骤

- 设置连接数
- 设置连接密码
- 使设置生效
- 设置权限等级（也可以不设）
- 给路由器接口设置IP
- 设置特权密码

##### 1.3、命令

```
Router(config)#line vty 0 4			//设置允许远程登录，不需用户插Console线缆
Router(config-line)#password 123	//设置用户登录密码
Router(config-line)#login			//登录启动
Router(config-line)#exit
Router(config)#enable password 123	//设置特权模式密码
/*
	PS:
		1)CISCO一般支持16个并行的远程虚拟终端，按照编号就是：0 - 15
		2)line vty 0 4 ，表示允许0、1、2、3、4共个用户登录
		2)特权密码一定要设，不设好像远程登不上特权模式，原因不明(主要也没深究的兴趣)
		3)以上命令省略了IP配置
*/
```

##### 1.4、结果

<img src="Telnet路由结果.png" alt="Telnet路由结果" style="zoom: 67%;" />

~~(别问为什么辣么多次连接失败......问就是理解错Telnet的作用，把IP地址配成不同网段的了。(逃 )~~

#### 二、交换机

##### 2.1、示例拓扑图

##### <img src="Telnet交换机拓扑图.png" alt="Telnet交换机拓扑图" style="zoom:80%;" />

##### 2.2、实验步骤

- 设置连接数
- 设置连接密码
- 设置权限等级（也可以不设）
- 使设置生效
- 给交换机设置IP(这里是vlan)
- 设置特权密码

##### 2.2、命令

```
Switch(config)#line vty 0 4 
Switch(config-line)#password 123 
Switch(config-line)#login 
Switch(config-line)#exit
Switch(config)#int vlan 1
Switch(config-if)#ip address 192.168.1.1 255.255.255.0 （设置vlan ip用于连接）
Switch(config-if)#no shutdown （开启vlan 1）
Switch(config-if)#exit
Switch(config)#enable password 123 （设置用户特权密码）
```

##### 2.3、结果

<img src="Telnet交换机结果.png" alt="Telnet交换机结果" style="zoom:67%;" />

### SSH配置

##### 拓扑图

同上

##### 步骤

相较于Telnet，多了一下三步：

- 创建域名
- 生成RSA密钥对
- 设置登录方式为SSH（因默认为Telnet）

##### 命令

```
config# ip domain-name xxx		//起域名为xxx

```

