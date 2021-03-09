---
title: 第十一届蓝桥杯B组—E矩阵
date: 2020-10-14 23:33:30
tags: [算法,蓝桥杯,动态规划]
categories: 题解
mathjax: true
---

### 问题描述

> 把 1 ∼ 2020 放在 2 × 1010 的矩阵里。
>
> 要求同一行中右边的比左边大，同一列中下边的比上边的大。一共有多少种方案？
>
> 答案很大，你只需要给出方案数除以 2020 的余数即可。

**答案提交**
这是一道结果填空题，你只需要算出结果后提交即可。
本题的结果为一个整数，在提交答案时只填写这个整数，填写多余的内容将无法得分。

<!--more-->

### 分析

由题意我们大概可以简要得到下图的信息。

<img src="https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/20210309051634.PNG" alt="示例图（1）" style="zoom:50%;" />
$$
\begin{cases}
  A<C<D \\
  A<B<D
  
\end{cases}
$$
从上图示意中，我们可以发现A一定是最小的数。而与A相邻的B和C一定是仅小于A的两个数。例如：**若A=1，那么D绝对不可能等于2。**

因此，对于每个数的放法，一定是从最左边开始的。

对于一个新放的数：

- 要么紧挨着放在上边一行
- 要么紧挨着放在下边一行

且对于第二种情况必然是建立在上边一行已经放置的情况之下。

这种后一个的放法，仅受前一个放法影响的问题，显然具备无后效性，可以用DP的思想思考它。

<img src="https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/20210309051649.PNG" alt="DP分析过程" style="zoom:33%;" />

上述的 $ f\[i][j] $表示第一行有 i 个数，第二行有 j 个数放法。

当我们要放一个数的时,合理放法（即: **i - 1 ≥  j**）:

- 若放第一行，那放法数就有f\[ i-1 ][ j ] 种
- 若放第二行，那放法数就有f\[ i ][ j-1 ] 种

经我们上面的推导，**第一行已放置的i个数的个数必然大于等于第二行已放置的j个数的个数**。

所以，当出现如下图这种$ i = j $的情况时：

![流程解释](https://photos-1302100213.cos.ap-guangzhou.myqcloud.com/imgs/Blog/20210309051726.png)

新的数必然只能放第一行，此时的放法总数与 $ f\[i][j-1]  $ 相等。

所以综上所述
$$
f[i][j]=
\begin{cases}
f[i-1][j]+f[i][j-1] \quad ,i-1≥j\\
f[i][j-1]\quad \quad\quad\quad\quad\quad, i-1<j
\end{cases}
$$


#### 代码

~~~c++
#include <iostream>
using namespace std;
int f[1020][1020];
int main()
{
    f[0][0] = 1;               // 两行一个数字都不放，也是一种方案
    for (int i = 0; i <= 1010; i ++)
        for (int j = 0; j <= i; j ++)
        {
            if(i - 1 >= j)    
            	f[i][j] += f[i - 1][j] % 2020;
            if(j)
            	f[i][j] += f[i][j - 1] % 2020;
        }
   	cout<<f[1010][1010]<<endl;
}
~~~

这题其实是经典例题[Acwing 271. 杨老师的照相排列](https://www.acwing.com/problem/content/description/273/)的简化版。