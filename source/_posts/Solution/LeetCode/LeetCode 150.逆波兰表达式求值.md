---
title: LeetCode 150.逆波兰表达式求值
mathjax: true
date: 2021-9-26 14:39:42
tags: 链表
categories: [题解,LeetCode]
---

### 问题描述

[原题链接](https://leetcode-cn.com/problems/evaluate-reverse-polish-notation/)
根据[ 逆波兰表示法](https://baike.baidu.com/item/逆波兰式/128437)，求该后缀表达式的计算结果。

有效的算符包括 `+`、`-`、`*`、`/` 。每个运算对象可以是整数，也可以是另一个逆波兰表达式。

**说明：**

- 整数除法只保留整数部分。
- 给定逆波兰表达式总是有效的。换句话说，表达式总会得出有效数值且不存在除数为 0 的情况。

<!--more-->

### 分析

就如题目名字，这题就是考察逆波兰表示法，或者说后缀表达式的求值过程。

我们知道后缀表达式求值遵循以下规律：

1. 遇到数字对象，压栈
2. 遇到符号对象，弹出两项并运算。

具体到这道题来说

 流程：
            待判定项：
                (1)正数判定：字符长度 l >=1
                (2)负数判定：字符长度 l > 1
                (3)符号判定：字符长度 l = 1
            1、遍历vector
            2、判断当前串长度是否大于1
            if l > 1:
                判断第一个字符是否为负号，是的话标记下。
                读取后面的数字
            if l==1：
                数字or操作符。
                优先处理操作符。

### 代码

```c++
class Solution {
public:
    int Calc(int a,int b,char op){
        switch(op){
            case '+': 
                return a+b;
            case '-':
                return a-b;
            case '*':
                return a*b;
            case '/':
                return a/b;
        }
        return 0;
    }
    int evalRPN(vector<string>& tokens) {
        stack<int> s;
        int len = tokens.size();
        for(int i = 0;i<len;i++){
            int a,b;
            int j=0;
            int l = tokens[i].size();

            if(l>1){          
                bool flag = false;
                if(tokens[i][j]=='-'){
                    flag=true;
                    j++;
                }
                int ans = 0;
                while(tokens[i][j]>='0' && tokens[i][j]<='9'){
                    ans=ans*10+tokens[i][j++]-'0';
                }
                if(flag)ans = -ans;
                s.push(ans);
            }
            else {
                switch(tokens[i][j]){
                    case '+':
                    case '-':
                    case '*':
                    case '/':
                        a = s.top();
                        s.pop();
                        b = s.top();
                        s.pop();
                        s.push(Calc(b,a,tokens[i][j]));
                        break;
                    default:
                        s.push(tokens[i][j]-'0');
                        break;
                }
            }
        }
        return s.top();
    }
};
```



