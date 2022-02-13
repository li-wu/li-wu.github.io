---
layout: post
title:  最长公共子序列（LCS）和最长递增子序列（LIS）
date:   2013-03-30 20:26:26 +0800
categories: algorithm
---

## 最长公共子序列问题（LCS）

可以用动态规划来解决LCS问题，假设有字符串X和字符串Y，dp[i,j]表示的是X的前i个字符和Y的前j个字符的最长公共子序列的的长度。如g果X[i]==Y[j]，那么这个字符就可以和之前的LCS一起构成一个新的LCS；
如果X[i]!=Y[j],那么分别考虑dp[i-1,j]和dp[i,j-1]，选择其中较大的为LCS。
```c
if(i==0||j==0)
    dp[i,j]=0;
else if(X[i]==Y[j])
    dp[i,j]=dp[i-1,j-1]+1;
else
    dp[i,j]=max(dp[i-1,j],dp[i,j-1]);
```
具体实现参考[这里](http://www.slyar.com/blog/poj-1458-cpp.html).

## 最长递增子序列（LIS）
要求长度为i的序列的Ai{a1,a2,……,ai}最长递增子序列，需要先求出序列Ai-1{a1,a2,……,ai-1}中以各元素(a1,a2,……,ai-1)作为最大元素的最长递增序列，然后把所有这些递增序列与ai比较，如果某个长度为m序列
的末尾元素aj(j<i)比ai要小，则将元素ai加入这个递增子序列，得到一个新的长度为m+1的新序列，否则其长度不变，将处理后的所有i个序列的长度进行比较，其中最长的序列就是所求的最长递增子序列。
实现参考[这里](http://blog.csdn.net/joylnwang/article/details/6766317).

LIS的O(nlogn)的实现参考[这里](http://fayaa.com/code/view/13122/).
