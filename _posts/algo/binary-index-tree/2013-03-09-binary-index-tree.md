---
layout: post
title: "binary index tree"
description: "data structure"
category: algorithm
tags: [algo, data structure]
---
{% include JB/setup %}

二进制索引树(树状数组)

基本思想
---------

按照Peter M. Fenwick的说法，正如所有的整数都可以表示成2的幂和，我们也可以把一串序列表示成一系列子序列的和。
采用这个想法，我们可将一个前缀和划分成多个子序列的和，而划分的方法与数的2的幂和具有及其相似的方式。
一方面，子序列的个数是其二进制表示中1的个数，另一方面，子序列代表的f[i]的个数也是2的幂. 
摘自[wiki][1].


定义
-----
对于序列a, 我们设一个数组C

*   C[i] = a[i - lowbit[i] + 1] + .. + a[i]
*   k为i在二进制下末尾0的个数
*   i从1开始

C即是a的二进制索引树.

2^k = i&(i xor (i-1))  
ie.. i&(-i)  
定义 lowbit(x) = x&(-x)  

C[i] = a[i - lowbit[i] + 1] + .. + a[i]

查询
-------
设sum(k) = a\[1\]+a\[2\]+...+a[k]   
则 a[i] + a[i+1] + ... + a[j] = sum(j)-sum(i-1)

sum(k) = C[n1]+C[n2] + ...+ C[nm]   
其中 nm= k, ni-1 = ni - lowbit(ni)   
因为ni-1 比ni少一个1, 而k最多有log2(k)个1, sum(k)最多log2 k项, 所以求和的时间复杂度就是O(log2(k))

对应代码:

    int sum(int x)
    {
        int ret = 0;
        while (x > 0) {
            ret += c[x];
            x -= lowbit(x);
        }
        return ret;
    }


修改
-------
修改a[i]会影响哪些C呢?
C[n1], C[n2], ...C[nm]    
其中,n1 = i ,ni+1 = ni + lowbit(ni)    
nm + lowbit(nm) 必须大于 a 的元素个数 N   

对与ni+1 = ni + lowbit(ni)   
每次修改, 最低为的1,都向高位移动, 对与数组大小为n，最多进行log2(n)次.   

    6 = [110]
    6 + lowbit(6)
    = 110 + 10
    = 1000

因此，修改的时间复杂度是O(log2(n))

代码:

    void add(int x, int v)
    {
        while (x <= N) {
            c[x] += v;
            x += lowbit(x);
        }
    }

建树
------
根据定义有:   
C[k] = sum(k) – sum(k-lowbit(k))
这个显然可以O(n)搞定.

总结
---------
单点修改, 区间查询,的时间复杂度都是O(log(n)), 建树是O(n).
因此适用与单点修改，多次区间查询的情况.

线段树也行, 但是运行效率和编码效率都比不上树状数组.
树状数组用起来可是相当简单的哈.

应用举例
--------------
*   [poj 3321 apple tree][1]
  
    题目大意(抽象后的):  
    一颗树, 树的结点有权值, 可能修改某个结点, 查询某个结点所在子树的权值和.

    这个题目的转化思路还是挺巧妙的.

    dfs之, 对于每个结点, 按开始访问时间和结束访问时间拆成两个点.   
    则对与A结点, 拆分成A+和A-, 则其所有子结点, 访问时间都在, A+, A-之间.   
    这是dfs的性质.   

    这样就把求子树结点和, 转化为了区间和,巧妙啊.

    代码在[这里][3].

    其实也不拆成两个点也行, 还是dfs,但是你要记下其子树的起始位置和终止位置.
    保证它们是连续的就行.

               A
              / \
             B   C
            / \
           D   E

    前序遍历, A B D E C, 记下每颗子树的区间就行了.
    后序也行, 但是中序不行, 其子树区间不太容易获得.


*   [poj 1195 mobile phone][4]

    题目大意(抽象后的):

    一个矩阵, 可能修改某个点的值, 求一个子矩阵的和.

    这是二维的树状数组问题.

    定义:  
    C[i][j] 为 (i-lowbit(i) + 1, j - lowbit(j) + 1) 和 (i,j), 所表示的子矩阵的和.

    这样查询就转化为:

        int sum(int x, int y)
        {
            int ret = 0;
            for(i = x; i > 0; i -= lowbit(i))
                for(j = y; j >0; j -= lowbit(j))
                    ret += C[i][j];
            retrun ret;
        }

    不太好描述, 自己画个图看看就明白了.

    更新呢?(xi, yi)
    在x轴上会影响到  
    xi, xi+1, xn
    其中xi+1 = xi + lowbit(xi).  
    因为C[xi+1][y] 包含了C[xi][y].

    对每一个点, C[xk][yj]  
    影响到了C[xk][yj], C[xk][yj+1], C[xk][yn]   
    因为, C[xk][yj+1]包含了C[xk][yj].

    因此, 更新的代码为:

        void udpate(int x, int y, int v)
        {
            for(i = x; i < n; i += lowbit(i))
                for(j = y; j < n; j += lowbit(j))
                    C[i][j] += v;
        }

    更新和查询的时间复杂度都为O(log(m) * log(n))

    代码在[这里][5].

<!--
这个数据结构, 把数组的区间和[1, p], 转化为q个区间和, 这写区间的长度都是2的幂.
其中q是其中是p的二进制数中1的个数.
比如7 = 111
那么, sum[1,7] = sum[1,4] + sum[5,6] + sum[7,7]
-->

参考了:    

*   线段树和树状数组(北大郭炜): <http://poj.org/summerschool/1_interval_tree.pdf>
*   算法竞赛入门经典-训练指南
*   Binary Indexed Trees: <http://community.topcoder.com/tc?module=Static&d1=tutorials&d2=binaryIndexedTrees>

[1]: http://zh.wikipedia.org/zh/%E6%A0%91%E7%8A%B6%E6%95%B0%E7%BB%84 "wiki 树状数组"
[2]: http://poj.org/problem?id=3321  "apple tree"
[3]: https://github.com/fly2best/oj/blob/master/poj/binary_index_tree/apple_tree_3321.cc
[4]: http://poj.org/problem?id=1195 "Mobile phones"
[5]: https://github.com/fly2best/oj/blob/master/poj/binary_index_tree/mobile_phones_1195.cc
