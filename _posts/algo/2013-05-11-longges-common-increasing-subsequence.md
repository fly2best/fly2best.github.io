---
layout: post
title: "最长公共上升子序列"
description: ""
category: algorithm
tags: [algo, dp]
---
{% include JB/setup %}

问题:

给定两个字符串x, y, 求它们公共子序列s, 满足si <  sj ( 0 <= i < j < |s|).
要求S的长度是所有条件序列中长度最长的.



##比较直观的做法(O(n^4))

可以仿照最长上升子序列用dp[i][j], 表示以xi, yj结束的公共字串的长度.

so, 我们可以得出递推公式

    if xi != yj
        dp[i][j] = 0
    else
        dp[i][j] = max(dp[ii][ij]) ( 0 <= ii < i, 0 <= ij < j, dp[ii][ij] != 0 && x[ii] < x[i]) + 1

时间复杂是O(n^4)

##O(n^3)的算法

LICS是从LIS和LCS演变而来的.我们来看看LIS和LCS的动态规划解决方法.

在LIS中dp[i]表示以xi结束的最长上升子序列的长度.
在LCS中dp[i][j]表示x[0...i]和y[0...j]的最长公共字串的长度.

为什么在LIS中dp[i]表示的不是x[0...i]中的最长子序列的长度?

因为在算LIS中dp[i]的时, 需要知道上一次字符的信息, 这样才能判断是否把x[i]加入.
而在计算LCS中dp[i][j]是不需要知道上一字符的信息, 只考虑当前字符就可以.

在LICS中, 和LIS中一样, 我们需要知道上一字符的信息, dp[i][j], xi和yj就是上一字符信息, 如果xi, yi 相等, 则信息重复冗余, 我们可以试着消除冗余, 以得到一个更好的算法.

这样我们可以定义dp[i][j]表示x[0...i]和y[0...j]上的LICS, 并且在y中的结束位置为j.

so, 我们可以得到递归公式

    if xi != yj
        dp[i][j] = dp[i-1][j]
    else
        dp[i][j] = max(dp[i-1][k])(0 < k < j && y[k] < y[j]) + 1

证明:

    设x[0...m]和y[0...n]上的, 以y[n]为结束字符的最长公共上升子序列为z[0...zn].

    若x[m] != y[n], 则显然z[0...zn]为x[0...m-1]和y[0...n]上的, 以y[n]为结束的LICS.

    若x[m] == y[n], 则z[0...zn-1]必为x[0...m-1]和y[0...k]上的, 以y[k]为结束的最长的LICS( 0 < k < j), 否则会得出矛盾.

        反证:
        设s, s[0...sn]为x[0...m-1]和y[0...k]上的, 以x[k]为结束的一个LICS, 并且sn > zn-1.

        那么,s[0...sn] 可以加上y[n], 得到长度sn+1的一个以y[n]为结束字符的最长公共上升子序列, sn+1 > zn, 与假设矛盾.

    所以，上述的递推公式的是对的.

时间复杂度为O(n^3).


##O(n^2)对O(n^3)的一个优化.

我们看到, dp[i][j]依赖于dp[k][j-1] (0 < k < i).

在计算的时候可以把i作为外层循环，也可以把i作为内层循环.

如果把i做为外层循环的, 可以做一个优化, 把时间复杂度将为O(n^2).

O(n^3)的算法

    memset(dp, 0, sizeof(dp));
    for (i = 1; i <= m; i++) {
        for(j = 1; j <= n; j++) {

            dp[i][j] = 0;
            if (x[i] != y[j]) {
                dp[i][j] = dp[i-1][j];
            } else {
                for (k = 1; k < j; ++k) {
                    if (dp[i][j] < dp[i - 1][k] && y[k] < y[j]) {
                        dp[i][j] = dp[i - 1][k];
                    }
                }
                dp[i][j] += 1;
            }

如果优化, 就只能优化当x[i] = y[j]的时, dp[i][j]的计算.

因为现在O(n^2)个子问题，这是怎么搞也搞不掉的.

看这段代码:

    for (k = 1; k < j; ++k) {
        if (dp[i][j] < dp[i - 1][k] && y[k] < y[j]) {
            dp[i][j] = dp[i - 1][k];
        }
    }

当y[j] = x[i]时, 就等于

    for (k = 1; k < j; ++k) {
        if (dp[i][j] < dp[i - 1][k] && y[k] < x[i]) {
            dp[i][j] = dp[i - 1][k];
        }
    }

这是在求dp[i-1][k] (0 < k < j)中的满足y[k]< x[i]最大值 

因为i是不变的(外层循环), j在递增, 因此没有必要从头计算.

保存一个mlen变量保存dp[i-1][k] (0 < k < j)中的满足y[k]< x[i]最大值, 当j增加时只用化O(1)的时间更新mlen和计算dp[i][j].

代码如下:

    for (i = 1; i <= m; i++) {

        mlen = 0;

        for(j = 1; j <= n; j++) {

            dp[i][j] = dp[i-1][j];

            //更新mlen
            if (y[j] < x[i] && dp[i - 1][j] > mlen) {
                    mlen = dp[i - 1][j];
            }

            //计算dp[i][j]
            if (y[j] == x[i]) {
                dp[i][j] = mlen + 1;
            }
        }
    }

时间复杂度O(n^2)

练习:

*   [poj 2127: Greatest Common Increasing Subsequence](http://poj.org/problem?id=2127)
*   [hdu 4512: 吉哥系列故事——完美队形I](http://acm.hdu.edu.cn/showproblem.php?pid=4512)

参考资料:

*  [Proverbs: POJ 2127 DP(n^2)](http://www.cnblogs.com/proverbs/archive/2012/10/05/2712029.html)
*  [最长公共上升子序列(LCIS)的平方算法](http://wenku.baidu.com/view/3e78f223aaea998fcc220ea0)
