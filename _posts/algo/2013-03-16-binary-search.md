---
layout: post
title: "二分查找及其变形"
description: ""
category: algorithm
tags: [algo]
---
{% include JB/setup %}

前几天, 在勇幸的博客上看到了[二分查找,你真的会吗?][1]
他在文中提到这么几个问题?

1.  二分查找元素key的下标，如无 return -1
2.  二分查找返回key(可能有重复)第一次出现的下标，如无return -1
3.  二分查找返回key(可能有重复)最后一次出现的下标，如无return -1
4.  二分查找返回刚好小于key的元素下标，如无return -1
5.  二分查找返回刚好大于key的元素下标，如无return -1

马上要找实习了，就拿来练练手.

我对问题二和三比较感兴趣.

问题1, 是最基本的二分查找.   
问题4,5 很容易通过2和3实现.   
重点就在问题2和3了.

想了想, 调了调，结果如下:

问题1:
{% highlight cpp %}
    // [0,n), 查找key
    int bin_search(int key,int *p, int n)
    {
        if (p == NULL || n <= 0)   //最进看了一些面试的书, 提高要提高代码的健壮性, so...
            return -1;

        int l = 0;
        int r = n;

        while (l < r) {
            int m = l + (r - l) / 2;  //这样做的原因是防止溢出

            if (p[m] == key)
                return m;
            else if (p[m] < key) {
                l = m + 1;
            } else {
                r = m;
            }
        }
        assert(l == r);
        return -1;
    }
{% endhighlight%}

问题2:

先看代码:

{% highlight cpp %}
    int bin_search_first(int key,int *p, int n)
    {
        if (p == NULL || n <= 0)
            return -1;

        int l = 0;
        int r = n;

        int first = -1;    //添加first变量
        while (l < r) {
            int m = l + (r - l) / 2;

            if (p[m] == key) {
                first = m;   //保存找到的最左边的position
                r = m;
            }
            else if (p[m] < key) {
                l = m + 1;
            } else {
                r = m;
            }
        }

        assert(l == r);

        return first;
    }
{% endhighlight%}

为什么呢?

现在我们要找重复出现的数中的第一个.   
那么我们找到其中一个后, 就应该在其左边接着找.  
如果,左边有的话，那么一定能找到;
否则, first保存上次的位置.  
初始化first为-1, 如果没找到即为-1, 如果存在一定能找到最左边的.

如果找重复的最右边的一个, 怎么做呢.  
思路是一样的，保存一个last变量.  
具体见下面的测试代码.


完整的测试代码在[这里][2].

[1]: http://www.ahathinking.com/archives/179.html  "yx.ac 二分查找, 你真的会吗?"
[2]: https://github.com/fly2best/pastebin/blob/master/algo/bin_search.cc
