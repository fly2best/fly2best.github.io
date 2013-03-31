---
layout: post
title: "并查集"
description: "总结"
category: algorithm
tags: [algo, data structure]
---
{% include JB/setup %}

并查集是一种小巧的数据结构.

主要有两个操作
--------------
*   可以判断两个数是否是属于一个集合.
*   合并两个数所在的集合.

关键数据结构
-----------
father[M], 记录下结点父结点，初始化为-1.


代码
----

    find(x)
    {
        while( fatcher[x] != -1)
            x = fatcher[x];
        return x;
    }

    union(x, y)
    {
        s1 = find(x);
        s2 = find(y);

        if (s1 != s2)
            fatcher[s1] = s2;
    }

时间复杂度
----------
如果采用这种naive的方法来查找和合并的话.
可能会出现最后连成一条链的情况, 时间复杂度就是O(n)了。

优化
-----

* 路径压缩

    两趟法
    *   第一趟找出所在结合的代表.
    *   第二趟更新路径上结点的fatcher结点.

    这样find一次, 就对路径压缩, 下次在find的时候, 可以很快返回.

        find(x)
        {
            if( fatcher[x] != -1)
                return x;

            fatcher[x] = find(x);
            return fatcher[x];
        }

* 按rank合并

    这个思路就是就是记下当前树的高度.
    合并的时候, 把树的高度较小的合并到较大的上面去.

    目的还是减少find时候的循环次数.

        union(x, y)
        {
            s1 = find(x);
            s2 = find(y);

            if (rank[s1] > rank[s2])
                fatcher[s2] = s1;
            else {
                fatcher[s1] = s2;

                if (rank[s1] == rank[s2])
                    rank[s2] += 1;
            }
        }

优化后的时间复杂度
----------------
几位大大说，这个find时间复杂度为将为常数.

采用平摊分析什么的.   
不过那一块我不太熟悉, 有机会再补上吧.

有兴趣的读者, 可以去看算法导论的相关章节.

应用举例
-----------
*   [poj 2492 A bug life][1]

    题目大意: 不知道虫子性别, 但知道了谁和谁交配了.看同性恋的虫子存在不存在.

    输入: 点对 i, j. 表示虫子i和虫子j进行交配   
    输出: 是否存在同性恋的虫子

    方法:

    如果a和b交配, 就连一条边.   
    在同一个连同分量内部, 判断是否出现冲突.   
    连同分量之间, 比较没有意义, 因为不知道性别.

    连同分量1中的bug和连同分量2中的bug交配了, 就要进行合并, 如何合并呢?   
    我是这么处理的.

    *   维护一个并查集表示, 是否是一个连同分量
    *   在同一个连同分量内部, 维护另外一个并查集, 看是否是同一个性别.

    合并时

    *   首先合并连同分量.
        这个容易实现.

    *   然后合并性别.

        a 属于  连同分量A, b 属于连同分量B.   
        a, b交配, a 和 b 不属于一个性别   
        所有B中和b是一个性别的和a性别不相同.  
        所有B中和b不是一个性别的和a性别形同.

        注意事项: A 和 B 中可能只有一个元素.
        完了.

    代码在[这里][2].

*   在求最小生树

    最小生成树的kruskal算法, 要判断两个结点是否是属于一个连同分量, 用并查集是极好的.


[1]: http://poj.org/problem?id=2492
[2]: https://github.com/fly2best/oj/blob/master/poj/disjoined_set/a_bug_life_2492_sisjoined_set.cc
