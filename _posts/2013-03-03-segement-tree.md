---
layout: post
title: "线段树"
description: "数据结构"
category: algorithm
published: false
tags: [algo, data structure]
---
{% include JB/setup %}

线段树

定义

特征
1.  线段树的深度不超过log(L), L表示区间长度.
2.  线段树吧区间上的任意一条线段,都分成不超过2Log(L)条线段.

典型应用
1. 区间查询(sum, max, min)， 点修改
2. 区间查询，区间修改(set, add)

key point
用线段树解题,关键是要想清楚每个节点要存哪些信息(当然区间起终点,以及左右子节点指针是必须的),以及这些信息如何高效更新,维护,查询。
不要一更新就更新到叶子节点,那样更新效率最坏就可能变成O(n) 的了。

注意事项:
离散化处理.
