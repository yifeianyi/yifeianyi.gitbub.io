---
title: LeetCode 735.行星碰撞
mathjax: true
date: 2021-9-26 15:05:10
tags: 链表
categories: [题解,LeetCode]
---

### 问题描述

[原题链接](https://leetcode-cn.com/problems/asteroid-collision/)

给定一个整数数组 `asteroids`，表示在同一行的行星。

对于数组中的每一个元素，其绝对值表示行星的大小，正负表示行星的移动方向（正表示向右移动，负表示向左移动）。每一颗行星以相同的速度移动。

找出碰撞后剩下的所有行星。碰撞规则：两个行星相互碰撞，较小的行星会爆炸。如果两颗行星大小相同，则两颗行星都会爆炸。两颗移动方向相同的行星，永远不会发生碰撞。

 **提示：**

- `2 <= asteroids.length <= 104`
- `-1000 <= asteroids[i] <= 1000`
- `asteroids[i] != 0`

### 分析