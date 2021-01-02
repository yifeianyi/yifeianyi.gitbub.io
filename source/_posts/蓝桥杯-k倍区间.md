---
title: 蓝桥杯 k倍区间
date: 2020-10-08 18:52:23
tags: [蓝桥杯,算法,前缀和]
categories: 题解
mathjax: true
---

[题目链接](https://www.acwing.com/problem/content/1232/)

### 问题描述

给定一个长度为 N的数列，A<sub>1</sub>,A<sub>2</sub>,…A<sub>N</sub>，如果其中一段连续的子序列 A<sub>i</sub>,A<sub>i+1</sub>,…A<sub>j</sub>之和是 K的倍数，我们就称这个区间 【i,j】 是 K倍区间。

你能求出数列中总共有多少个 K 倍区间吗？

<!--more-->

#### 输入格式

输出一个整数，代表 K倍区间的数目。

#### 数据范围

1 ≤ N , K ≤ 100000
1 ≤ A<sub>i</sub> ≤100000

### 分析

首先看到 Max(N)=1e5 ,可以判断出这题时间复杂度需在O(NlogN)以内。

题目问的是子序列和是k的倍数的个数。那么很容易想到枚举所有区间的子序和的暴力做法。纯暴力的话时间复杂度为O(n<sup>3</sup>)，经过简单的前缀和优化后能做到O(n<sup>2</sup>)，虽然还不能**AC**此题，但可以先写出来，再慢慢想优化问题。

#### 暴力源码

```c++
#include<iostream>
using namespace std;
typedef  long long LL;
const int N = 1e6;
LL n,k;
LL a[N];
//纯暴力做法:枚举起始位置、终点位置后，再用一重循环求得子序和
LL fun1(){
	LL ans = 0;
	for(int R=1;R<=n;R++){
		for(int L=1;L<=R;L++){
			LL sum = 0;
			for(int k=L;k<=R;k++)sum=sum+a[k];
			if(sum%k==0)ans++; 
		}
	}
	return ans;
}
//简单前缀和优化：省去求得子序和的循环
LL fun2(){
	LL ans = 0;
	for(int R=1;R<=n;R++){
		LL sum = 0;
		for(int L=1;L<=R;L++){
			sum=(sum+a[L])%k;
			if(sum%k==0)ans++; 
		}
	}
	return ans;
}
int main(){
	cin>>n>>k;
	for(int i=1;i<=n;i++)cin>>a[i];
//   cout<<fun1()<<endl;
//	cout<<fun2()<<endl;
	return 0;
} 
```

在以上认知的基础上，要想再优化，那么就只能从第二重循环下手了。

对于第二重循环的作用，我们可以理解为是：**在R固定的情况下，在【1，R】中有多少个L满足:**
$$
(s[R]-s[L-1])\%k=0
$$
根据同余定理，我们可以得到下面的等式：
$$
s[R]\%k-s[L-1]\%k=0
$$
等价于
$$
s[R]\%k = s[L-1]\%k
$$
所以，我们只要统计一下,在R固定的情况下，【1，R】中有多少个s[L-1]与k取模后，与**S[R]%k**得到的数相同即可，这个操作过程是O(1)的，固可以再优化一重循环。

最后值得注意的是，因为模为0的情况下不需要有跟其它前缀和配对即可确认，因此为了书写形式的统一性，我们可以先让Mod[0]=1。

#### 优化源码

```c++
#include<iostream>
using namespace std;
typedef long long LL;
const int N = 1e6;
LL a[N],Mod[N];
int main(){
	int n,k;
	cin>>n>>k;
	Mod[0]=1;
	LL ans = 0;
	for(int i = 1;i<=n;i++){
		cin>>a[i];
		a[i]=(a[i]+a[i-1])%k;
		ans+=Mod[a[i]];
		Mod[a[i]]++;
	}
	cout<<ans<<endl;
	return 0;
}
```

#### 写法二

整体原理跟上面相同，都是先求得子序的余数情况。不同的是，这里是利用组合数公式来求得不同余数情况的配对总数。

```c++
#include<iostream>
using namespace std;
typedef long long LL;
const int N = 1e6;
LL a[N],Mod[N];
int main(){
	int n,k;
	cin>>n>>k;
	Mod[0]=1;
	LL ans = 0;
	for(int i = 1;i<=n;i++){
		cin>>a[i];
		a[i]=(a[i]+a[i-1])%k;
		Mod[a[i]]++;
	}
    
	for(int i = 0;i<k;i++){
		ans+=(Mod[i])*(Mod[i]-1)/2;//组合数公式c(n,2)
	}
	cout<<ans<<endl;
	return 0;
}
```

