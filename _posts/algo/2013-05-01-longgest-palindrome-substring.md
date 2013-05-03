---
layout: post
title: "最长回文子串"
description: ""
category: algorithm
tags: [algo, dp, enum, suffix array, extend kmp]
---
{% include JB/setup %}

本文对求最长回文子串的方法进行了总结:

1. [暴力枚举](#bruce)([枚举所有子串](#enum_substring)及[一个动态规划改进](#improved_by_dp), [枚举中心点](#enum_middle))
2. [使用后缀数组](#by_suffix_array)
3. [用扩展kmp方法](#extend_kmp)
4. [manacher算法](#manacher)

回文串, 就是顺着念和倒着念是一样的.
如: abcba, abba.

最长回文子串, 就是长度最长的回文子串.


## 暴力的方法  {#bruce}

### 枚举每个字串, 判断是否是回文串.  {#enum_substring}
因为有$n^2$个字串, 判断是回文串需要$O(n)$的时间复杂度.
因此要花费$O(n^3)$的时间.

### 一个动态规划的改进  {#improved_by_dp}

上述的枚举方法, 有不少重复计算.

比如判断$str[i...j]$, 是否是回文子串,  会重复计算$str[i+1...j-1]$是否是回文字串.

因此一个简单的改进就是保存$str[i+1...j-1]$是否是回文字串, 这样在计算的$str[i...j]$的时候, 就可以在$O(1)$的时间内得到结果.

$dp[i][j]$表示$str[i...j]$是否是回文子串.

so...

$$dp[i][j] = \left\{ \begin{array}{ll}
1 & \textrm{if $i = j$ }\\
1 & \textrm{if $i + 1 = j$ and $str[i] = str[j]$ }\\
1 & \textrm{if $str[i] = str[j]$ and $dp[i + 1][j -1]$}\\
0 & \textrm{else}
\end{array} \right.$$

根据子问题之间的依赖关系, 可以得出应该按步长递增的顺序来求解.

    for (int i = 0; i < n; ++i)
        dp[i][i] = 1;

    //为方便计算
    for (int i = 1; i < n; ++i)
        dp[i][i - 1] = 1;

    for (int s = 2; s <= n; ++s) {
        for (int i = 0 ; i + s <= n; ++i) {
            int j = i + s - 1;
            dp[i][j] = (str[i] == str[j]) && dp[i+1][j-1];
        }
    }

时间复杂度, $O(n^2)$个子问题, 每个$O(1)$可以搞定, so时间复杂的是$O(n^2)$, 不过有个$O(n^2)$的空间复杂度.

### 枚举对称点  {#enum_middle}

还有一种暴力的方法, 就是枚举对称点.

奇数长度的回文串的对称点就是最中间一个字母.

偶数长度的回文串的对称点是最中间的两个字母.

枚举中间点, 然后向两边扩展, 求以此为对称点最长的回文串.

因为最多有$2n$个对称点(偶数和奇数), 求以此为对称点的回文串的时间复杂度是$O(n)$

所以, 这种方法的时间复杂度是$O(n^2)$.

**这种方法的较好的原因是, 枚举的子问题的个数较少.**


## 用后缀数组来解决  {#by_suffix_array}

转化为求LCP(longgest common prefix, 最长公共前缀).

方法:
把str逆过来记为r(str), 然后用一个未出现过的字符, 把str和r(str)连起来.

对

i = 1 to n - 1

求

1. $suffix[i]$和$suffix[2n + 1 - i]$的LCP, 回文子串长度是$2LCP$. //长度是偶数

2. $suffix[i]$和$suffix[2n - i]$的LCP,回文子串的长度是$2LCP - 1$. //长度是奇数

取较大值, 就是最长回文字串的长度.

如: aab, aab#baa.

求suffix[1] = ab#baa  
  suffix[6] = a, lcp = 1, 回文串长度为2  
  suffix[5] = aa, lcp = 1, 回文串长度为1  

  suffix[2] = b#baa  
  suffix[5] = aa, lcp = 0, 回文串长度为0  
  suffix[4] = baa, lcp = 1, 回文串长度为1  

时间复杂度分析

1. 后缀数组, 可以在O(n)的时间复杂度内求出sa, height数组.
2. 有了height数组, 可以用rmq求出lcp, rmq有O(n)的预处理算法,每次查询只要O(1).
3. 最多查询O(n)次, 因此时间复杂度是O(n)

## 扩展的KMP方法, 分治的思路. {#extend_kmp}

扩展的KMP方法,是指
给定母串S，和子串T。定义n=|S|, m=|T|，extend[i]=S[i..n]与T的最长公共前缀长度。
可以在线性的时间复杂度内，求出所有的extend[1..n]。

这总方法的思路是,把字符串分成左右两部分, 分别对左右求最长回文串, 然后考虑跨界的情况.
如下图, 分成[0,m)和[m,n)两部分.

设存在一个跨界,中心在左边在左边的回文串, 且[x,i)和[m, m + i -x)对称.
则[i, m)是一个回文串.

上述中有两个变量, i, x.

记r(L),为[0,m)的逆, r(R)为[m, n)的逆.

记L与r(L)的扩展kmp的问题解为extend_east, 记r(L)与R的一个扩展kmp问题解为extend_west.

显然, 若[i, m)为回文串则, extend_east[i] = m - i;  
而跨界字符产的长度即len = m - i + 2 * extend_west[i-1];

枚举i, O(n)个子问题, O(1)时间可求出跨界在重心在左边的字符串的长度.

同理可以求出重心在右边的跨界字符串的长度.

so... 
T(n) = 2T(n/2) + O(n)   
T(n) = O(nlog(n))

利用扩展的kmp, 可以在O(nlog(n))的时间复杂度求出最长回文串.

## Manacher算法  {#manacher}

可以认为Manacher算法是对上文所述的第二枚举方法的改进.

先对字符串进行处理, 把长度偶数回文串转化为长度为奇数的回文串.

处理办法, 用一个未出现的字符对原串进行分割.

如: abba, 转化为a#b#b#a  
    abcba, 转化为a#b#c#b#a  

这样把统计转化为长度为奇数的回文串.

记数组p,为以字符i为中心的回文串的长度的一半,上取整, 转化后p[i]就是以此为中心的回文串的长度.

p[i]的最大值就是最大回文字串的长度.

怎么就pi呢? 用了一个辅助变量mx.

mx = max(id + p[id])

计算i的时候,

若i < mx, 则有

if (i + p[2*id - i] < mx)
    p[i] = p[j];  //根据对称性
else 
    p[i] = mx - i; //还要接着比较

如 i > mx, 没有已知信息, p[i] = 1, 从头搞.

so ...

    int id;
    int i;
    int mx = 0;

    for (i = 1; i <= n; ++i) {
        p[i] = 1;
        if (mx > i) {
            p[i] = min(p[2*id -i], mx -i);
        }

        while (str[i + p[i]] == str[i - p[i]])
            p[i]++;

        if (p[i] + i > mx) {
            mx = p[i] + i;
            id = i;
        }
    }

时间复杂度, O(n).

分析过程:
主循环, 显然是O(n), 关键点在分析循环内while循环的执行次数.

1. i < mx 且 i + p[2*id -i] < mx
   则执行一次退出.
2. i < mx 且 i + p[2*id -i] >= mx
   可能会进行多次比较
3. i > mx.
   可能会进行多次比较

在case 2和case 3情况下,成功比较多少次,就会对mx加上多少.
因为mx是递增的, 最多增加到N.
so, case2和case3 下的比较次数是O(n)的.

因此总的时间复杂是O(n).

参考资料:

1. 后缀数组方法求回文子串: <http://blog.csdn.net/leolin_/article/details/7218446>
2. 扩展kmp方法求回文子串: <http://greatkongxin.blog.163.com/blog/static/17009712520117285717159/>
3. mancher算法: <http://www.felix021.com/blog/read.php?2040>
4. manacher算法: <http://larrylisblog.net/WebContents/images/LongestPalindrom.pdf>
