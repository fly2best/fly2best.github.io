---
layout: post
title: "线段树"
description: "数据结构"
category: algorithm
tags: [algo, data structure]
---
{% include JB/setup %}

线段树

定义
----
线段树是一棵二叉树,树中的每一个结 点表示了一个区间[a,b]。a,b通常是整数。
每一个叶子节点表示了一个单位区间。 对于每一个非叶结点所表示的结点[a,b],
其左儿子表示的区间为[a,(a+b)/2],右儿 子表示的区间为[(a+b)/2 + 1,b]

特征
----
1.  线段树的深度不超过log(L), L表示区间长度.
2.  线段树吧区间上的任意一条线段,都分成不超过2Log(L)条线段.   
    因为每条线段在同一层上不会超过有超过两条线段.
3.  点更改的,时间复杂度,为O(Log(L)), 只访问了从根结点到叶子结点的路径.
4.  如果区间更改采用延迟标记, 那么时间复杂读O(Log(L)).
    在向下进行更新的时候,每层最多方位4个结点, 想想为什么?
5.  结点数目  2L -1   
    因为n2 = n0 -1; n1 =0; n0 = N; so n= 2L -1.  
    n2表示度为2的结点个数, n1表示度为1的结点个数, n0表示叶子结点个数.  
    是不是想起huffman树了呢, 他们都没有度为1的结点,在结点个数上其性质是一样的.
6.  最大结点编号   
    可以到4L   
    如果用数组存放线段树, 空间开4L.

基本用途
-------
线段树适用于和区间统计有关的问题。
比如某些数据可以按区间进行划分,按区间动态进行修改,
而且还需要按区间多次进行查询,那么使用线段树可以达到较快查询速度。

应用举例
--------
-   点更改, 区间查询

    [hdu 1166 敌兵步阵][1]   
    题目大意, 可以修改某个值, 然后求区间和.   
    方法:线段树结点, 记录区间的和.  如果更新某个点的值, 递归向上更新到根结点.   
    代码在[这里][2]

-   区间更改, 区间查询

    [poj 3468 A Simple Problem with Integers][3]   
    题目大意, 区间更改, add, 区间查询和
    方法:   
    线段树结点, 记录区间的和, 另附加一个字段记录, 区间上的add值.   
    更新的时候, 延迟标记, 并步更改到底.
    查询或更新的时候, 要注意把延迟标记向下传递.
    更新的时候, 要注意把更改向上传递.    
    这是我做的线段树第二题目, 写的还不是很飘逸..
    sum+add才是区间的和.   
    代码在[这里][4].   
    也推荐去hanghang大牛那里去看看.

-   区间更改, 区间查询一次

    [poj 2528 Mayor post][5]    
    题目大意: 区间set, 统计不同的set数目.
    这是我线段树第一题.
    这个题目要离散化处理的, 我记得这个地方还有个坑.  
    方法:   
    更新,区间set, 加延迟标记, 加离散化.
    查询, 遍历一次好了, hash计数, 输出不同的颜色个数.
    代码在[这里][6].   

-   区间更改,多次区间查询

    [poj 2777 count colors][7]   
    题目大意: 区间染色,统计区间不同颜色的个数.
    方法:
    更新,区间set, 延迟标记.
    colors存放颜色的种类, 只有30中颜色用bitmap存放.   
    color存放了是否是一种颜色, 如果是只有一种颜色, color就是颜色值,否则是0.   
    color的作用是set, colors的作用的sum, 特殊的sum.向上pushup时候, 用|运算符, 算或父结点的值.   
    代码在[这里][8].   

上面的题目都还是比较直观的, 一看就是区间问题, 下面是一些要拐些弯儿的.

转化后才能用线段树来解决的
--------------------------
-   [poj 2828 buy tickets][9]
    这个题目
    一个队列, 随便插队.   
    现在给了你插队的位置序列 x0...xn, xi表示第i个人插在了第xi个位置,求最终的队列的顺序.   

    比较naive的方法是用个链表或数组记录顺序, 插入的时间复杂度是O(n), 总的时间复杂度是O(n^2).  
    但是这个题目的n有2 * 10^5, 时间是给了4s, 数量级4 * 10^10, 这个绝对要TLE的.   

    看了discuss.....   
    倒来看的话, 第i个人插在了第xi个位置, 表示其前面有xi的空位.   
    哇卡卡, 这个思路真巧妙啊.   
    这样就转化为了点修改, 求区间和的问题.

    代码在[这里][10].

-   [poj 3667 Hotel][11]
    题意: 客官每次住店的想要连续的房间, 如果有多个满足要求的房间序列, 选择房间号小的.

    我当时这道题目没想清楚, 写的一陀阿....   
    看了看别人的思路，又想了想.
    就是要求满足要求连续空房间数.
    可以在结点上记下, 从区间左边开始的连续空白房间数, 到区间结束的连续空白房间数.
    pushup的时候

        sum[rt] = max(max(sum[lc],sum[rc]), rsum[lc] + lsum[rc]);
        lsum[rt] = lsum[lc];
        rsum[rt] = rsum[rc];
        if (lsum[lc] == len - len / 2)
            lsum[rt] += lsum[rc];
        if (rsum[rc] == len / 2)
            rsum[rt] += rsum[lc];

    查询的话, 就好找了, 如果区间满足, 则先看左儿子是否满足, 再看左儿子和右儿子一起是否满足, 最后再看右儿子是否满足.

    我看到disscuss的一句话, 激起了我的联想.   
    discuss 上，有人说, [这道题目其实是模拟的内存管理里面的一个内存分配策略][12].   
    我一想对阿, 这不是首次适应分配策略嘛.   
    我又想linux内核是怎么弄的呢? 去翻翻了代码，kernel用的是红黑树来管理线性地址空间的.
    为什么呢? 这个容我以后再谈.

    一个小小的acm题目, 搞到内存分配上面去了.... orz..    

    代码在[这里][13].

-   [poj 2750 plotted flowers][14]

    带环的最大连续子序列和问题.
    转化的思路也很巧妙阿,
    转化成了 不带环的最大连续子序列和 和 不带环的最小连续子序列和.   
    max(sum[rt]- summin[rt], summax[rt])

    代码在[这里][15].

-   [poj 2886 who get max candies?][16]   
    加强版的约瑟夫环.  
    这个题目到是不难想, 先前做过poj 3667那到题, 也容易联想过来.  
    线段数结点记录区间上有多少个空位, 这样更新和查找都可以在O(logn)的时间搞定.  
    代价是O(n)空间复杂度.

    这里面有个反素数的什么东西, 我没看的特别明白, 直接从别人的代码扣过来了... 1AC.
    改天我再去看看反素数.    

    代码在[这里][17].

-   [poj 2352 Stars][18]

    这是我的第一次看到二维的数, 因为数据的排序方式, 转化为了一唯的了.  
    转化为点更新, 区间和.   
    代码在[这里][19].

-   [poj 2482 Stars in Your Window][20]

    详细的解题报告在[这里][21]

注意事项
--------
离散化

因为线段数的时间和空间复杂度和区间长度有关,  有时区间的范围可能非常大, 但是点的个数并不多, 这是可以尝试离散化处理.

如 1,100000, 000000   
可以映射为1, 2 ,3   
一般是把区间里面的点排个序, 去重, 然后用下标来作为其映射后的值.
二分查找很容易找到其下标.

但有时会出问题.
比如加入输出是线段, 
比如 [1, 4], [6,7] 映射后 [1, 2], [3,4]
以前[1, 7]不是完全覆盖的线段的.
离散化后，就把[1 4]全覆盖了.
处理方法,  加入点, 保证以前不相邻的，映射后也不相邻.
如:
    1,    4,     6, 7
    1, 2, 3,  4, 5, 6
这样就好了.


总结
--------
线段数，确实很灵活, 在区间问题的时候很有用哈...
搞清楚你在区间上存放了什么, 怎么查询? 怎么更新?
线段树是一个相当实用的数据结构.


参考资料:

*   Not Only Success: <http://www.notonlysuccess.com/index.php/segment-tree-complete/>
*   勇幸|Thinking: <http://www.ahathinking.com/archives/136.html>
*   线段树和树状数组(北大郭炜): <http://poj.org/summerschool/1_interval_tree.pdf>
*   线段树讲稿(杨戈): <http://download.csdn.net/detail/pandm/2255479>

<!--
线段树上记录的信息应该是关于当前区间的信息. 这个应该总结一下的.

什么时候可以进行区间查询?
当相邻的区间的信息,可以被合并成两个区间的并区间的信息时,就可以回答区间查询。
第一例中,两个相邻区间的最小值中的较小值,就是并区间的最小值。
第二例中,同等长度的数字串的f函数可以合并,但是不同长度就不可以。因此,虽然可以
构造出这棵线段树,却不能回答我们所期望回答的区间询问。
really?

什么时候标记需要向上更新?
确实..., 杨戈的线段树讲稿，我卡，真心不错阿...

思考一下这个问题:
某些时候,在一些题目中我们需要涉及到变化的“区间”:一开始数轴上有一些相连的线
段,你每次可以查询某个点位于哪个线段,以及合并两个相邻线段,或者把某个线段一分为
二。对于这样的问题,我们可以看成是线段的端点处都是1,别的地方都是0。则“查找点位
于哪个线段”转化为“查找某点左侧区间的最右不小于1的数”以及“查找某点右侧区间的最左
不小于1的数”;而线段合并和分离分别就是查找到某个位置,然后把1改成0,或0改成1。查
找某个位置是第几条线段,可以统计某位置左侧区间的数字和。
可见,通过模型的转化,有相当多的问题都可以用线段树来解决

有空去看线段树的其他实现方式
链式存储和数组存储的差异.

典型应用
1. 区间查询(sum, max, min)， 点修改
2. 区间查询，区间修改(set, add)

key point
用线段树解题,关键是要想清楚每个节点要存哪些信息(当然区间起终点,以及左右子节点指针是必须的),以及这些信息如何高效更新,维护,查询。
不要一更新就更新到叶子节点,那样更新效率最坏就可能变成O(n) 的了。
-->

[1]: http://acm.hdu.edu.cn/showproblem.php?pid=1166 "hdu 敌兵布阵 segement tree"
[2]: https://github.com/fly2best/oj/blob/master/hdu/segement_tree_1166.cc "代码"
[3]: http://poj.org/problem?id=3468
[4]: https://github.com/fly2best/oj/blob/master/poj/segment_tree/a_simple_problem_with_integers_3468.cc
[5]: http://poj.org/problem?id=2528  "mayor's post"
[6]: https://github.com/fly2best/oj/blob/master/poj/segment_tree/mayor_post_2528.cc "mayor's post solution"
[7]: http://poj.org/problem?id=2777  "count color"
[8]: https://github.com/fly2best/oj/blob/master/poj/segment_tree/count_color_poj_2777.cc
[9]: http://poj.org/problem?id=2828 "buy tickets"
[10]: https://github.com/fly2best/oj/blob/master/poj/segment_tree/buy_tickets_2828.cc
[11]: http://poj.org/problem?id=3667 "hotel"
[12]: http://poj.org/showmessage?message_id=104617  "这不是模拟一个内存分配额吗?"
[13]: https://github.com/fly2best/oj/blob/master/poj/segment_tree/hotel_3667.cc
[14]: http://poj.org/problem?id=2750  "potted flower"
[15]: https://github.com/fly2best/oj/blob/master/poj/segment_tree/potted_flower_2750.cc
[16]: http://poj.org/problem?id=2886 "who get max candies?"
[17]: https://github.com/fly2best/oj/blob/master/poj/segment_tree/who_get_max_candies_2886.cc
[18]: http://poj.org/problem?id=2352 "stars"
[19]: https://github.com/fly2best/oj/blob/master/poj/segment_tree/stars_2352.cc
[20]: http://poj.org/problem?id=2482 "stars in your windos"
[21]: /algorithm/2013/03/07/poj-2482-stars-in-your-window/
