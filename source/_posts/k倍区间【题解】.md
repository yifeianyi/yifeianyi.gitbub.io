---
title: k倍区间【题解】
date: 2020-10-08 18:52:23
tags: [蓝桥杯,算法]
categories: 题解
---

### 问题描述

给定一个长度为 N的数列，A<sub>1</sub>,A<sub>2</sub>,…A<sub>N</sub>，如果其中一段连续的子序列 A<sub>i</sub>,A<sub>i+1</sub>,…A<sub>j</sub>之和是 K的倍数，我们就称这个区间 【i,j】 是 K倍区间。

你能求出数列中总共有多少个 K 倍区间吗？

#### 输入格式

输出一个整数，代表 K倍区间的数目。

#### 数据范围

1 ≤ N , K ≤ 100000
1 ≤ A<sub>i</sub> ≤100000

### 分析

首先看到 Max(n)=1e5 ,可以判断出这题时间复杂度需在O(NlogN)以内。

题目问的是子序列和是k的倍数的个数。那么很容易想到枚举所有区间的子序和的暴力做法。纯暴力的话时间复杂度为O(n<sup>3</sup>)，经过前缀和简单优化后果能做到O(n<sup>2</sup>)，但显然还不能**AC**此题，但可以先写出来，再慢慢想优化问题。

```c++
#include<iostream>
using namespace std;
typedef  long long LL;
const int N = 1e6;
LL n,k;
LL a[N];
//纯暴力做法 
LL fun1(){
	LL ans = 0;
	for(int i=1;i<=n;i++){
		for(int j=i;j<=n;j++){
			LL sum = 0;
			for(int k=i;k<=j;k++)sum=sum+a[k];
			if(sum%k==0)ans++; 
		}
	}
	return ans;
}
//前缀和简单优化 
LL fun2(){
	LL ans = 0;
	for(int i=1;i<=n;i++){
		LL sum = 0;
		for(int j=i;j<=n;j++){
			sum=(sum+a[j])%k;
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

在以上基础认知的情况下，我们再来观察子序和与每一项的关系：
$$
s\%k=(a_{i}+a_{i+1}+a_{i+2}+...+a_{j})\%k
$$
根据同余定理，我们可以得到下面的等式：
$$
a_{i}\%k+a_{i+1}\%k+a_{i+2}\%k+...+a_{j}\%k
$$
即
$$
s_{i}a_{i+1}\%k=0\\等价于\\
a_{i}\%k+a_{i+1}\%k+a_{i+2}\%k+...+a_{j}\%k=0\\
\therefore a_{i}\%k=a_{i+1}
$$
