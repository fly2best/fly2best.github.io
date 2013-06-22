---
layout: post
title: "next_permutation下一个排列算法"
description: ""
category: algo
tags: [algo, stl]
---
{% include JB/setup %}

这是stl中的一个算法, 功能是按字典序获取下一个排列.

##源码

    template<class BidirIt>
    bool next_permutation(BidirIt first, BidirIt last)
    {
        if (first == last) return false;
        BidirIt i = last;
        if (first == --i) return false;
     
        while (1) {
            BidirIt i1, i2;
     
            i1 = i;
            if (*--i < *i1) {
                i2 = last;
                while (!(*i < *--i2))
                    ;
                std::iter_swap(i, i2);
                std::reverse(i1, last);
                return true;
            }
            if (i == first) {
                std::reverse(first, last);
                return false;
            }
        }
    }

摘自 <http://en.cppreference.com/w/cpp/algorithm/next_permutation>

##实现过程分析

1.  从尾端开始往前寻找两个相邻的元素满足, *i < *i1
2.  如果找不到返回false, 否则转3
3.  从末尾找第一个大于*i的元素*i2, 交换*i和*i2, 对i1, last之间的元素逆序.

举例: 3, 2, 4, 5, 1

找到 $$*i = 4, *i_1 = 5, *i_2 = 5$$  
交换$$*i_2$$和$$*i$$得  
    3, 2, 5, 4, 1  
对$$i_1$$和last之间的元素逆序得   
    3, 2, 5, 1, 4

就是下一个排列.

##怎么证明这个算法是对的呢?

**只对当序列为1到n上的一个排列的这种特殊情况时进行证明.**

如果对1-n上的全排列按字典序排序, 然后计算出每个序列的序号;  
如果证明下个排列的序号比上一个排列的序号大1, 那么就证明了他们是相邻的.

怎么计算一个排列的序号呢?

如果可以计算出这个序列前面有多少个序列, 也就求出了其序号.

让我们来看看怎么计算这个序列前面有多少个序列.

对序列 $$a_0, a_1, a_2, ..., a_{n-1}$$

对与$$a_i$$, 找到$$j$$满足$$i < j 且 a_i > a_j$$,
交换$$a_i$$和$$a_j$$, 然后对$$a_{i+1}, ..., a_{n-1}$$ 上的元素全排列, 就得到$$(n-i -1)!$$个排列, 都比$$a_0, a_1, a_2, ..., a_{n-1}$$小.

对于$$a_i$$我们可以计算出在元素$$a_i$$后面的小于$$a_i$$的元素的个数$$r_i$$.

因此对于$$a_i$$, 经过交换和排列可以得到小于原来序列的个数为$$r_i*(n-i-1)!$$.

总数即为 $$\sum_{i=0}^{n-1} {r_i *(n-i-1)!}$$

对于序列
$$a_0, ..., a_x, a_y, ..., a_{z-1}, a_z, a_{z+1}, ..., a_{n-1}$$

若$$i = x, i_1 = y, i_2 = z$$

则有  
a. $$y = x + 1$$  
b. $$a_x < a_y > ... > a_{z-1} > a_z > a_{z+1} > ... > a_{n-1}$$  
c. $$a_z > a_x > a_{z+1}$$  

交换后的序列为
$$a_0, ..., a_z, a_{n-1}, ..., a_{z+1}., a_x, a_{z-1}, ..., a_y$$

记原序列的ri = $$b_0, b_1, ..., b_i, b_{i+1}, ..., b_{n-1}$$

记交换后序列的ri = $$c_0, c_1, ..., c_i, c_{i+1}, ..., c_{n-1}$$


则

$$  y = \left\{ \begin{array}{ll}
    c_k=b_k & \textrm{for k = 0 to i -1}\\
    c_k=b_k + 1 & \textrm{k = i}\\
    b_k = n - 1 - k, c_k = 0 & \textrm{for k = i to n -1}
    \end{array} \right.
$$

因此序号之差 

    = (n-i-1)! - (n-i-2)*(n-i-2)! - ... - 3 * 3 ! - 2 * 2! - 1 * 1!
    = (n-i-1)! - (n-i-2)*(n-i-2)! - ... - 3 * 3 ! - 2 * 2! - 1 * 1! - 1 + 1 
    = (n-i-1)! - (n-i-2)*(n-i-2)! - ....- 3 * 3 ! - 2 * 2! - 2! + 1 
    = (n-i-1)! - (n-i-2)*(n-i-2)! - ....- 3 * 3 ! - 3! + 1 
    = (n-i-1)! - (n-i-2)*(n-i-2)! - ....- 4! + 1 
    ....
    = (n-i-1)! - (n-i-1)! + 1 
    =1

这样便证明了此算法在1-n上的全排列的时候是正确的.

参考资料:

1. [康托展开](http://www.nocow.cn/index.php/%E5%BA%B7%E6%89%98%E5%B1%95%E5%BC%80)
2. [一种变进制及其应用(全排列之hash实现)](http://bbs.chinaunix.net/forum.php?mod=viewthread&tid=1283459&page=1&authorid=12454074)
