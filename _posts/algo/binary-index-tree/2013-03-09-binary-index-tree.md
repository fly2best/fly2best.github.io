---
layout: post
title: "binary index tree"
description: "data structure"
category: algorithm
tags: [algo, data structure]
published: false
---
{% include JB/setup %}


对于序列a, 我们设一个数组C
*   C[i] = a[i - 2^k + 1]
*   k为i在二进制下末尾0的个数
*   i从1开始

2^k = i&(i xor (i-1))
ie.. i&(-i)
lowbit(x) = x&(-x)

C[i] = a[i - lowbit[i] + 1] + .. + a[i]

设sum(k) = a[1]+a[2]+...+a[k]
则 a[i] + a[i+1] + ... + a[j] = sum(j)-sum(j-1)

sum(k) = C[n1]+C[n2] + ...+ C[nm]
其中 nm= k
ni-1 = ni - lowbit(ni)

C[n1], C[n2], ...C[nm]
其中,n1 = i ,ni+1 = ni + lowbit(ni)
nm + lowbit(nm) 必须大于 a 的元素个数 N

建树
C[k] = sum(k) – sum(k-lowbit(k))

sum(k) = C[n1]+C[n2] + ...+ C[nm-1] +C[nm] (nm = k)
nm-1 = k-lowbit(k)
sum(k-lowbit(k)) = C[n1]+C[n2] + ...+ C[nm-1]
