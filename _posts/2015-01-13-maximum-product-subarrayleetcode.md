---
id: 569
title: Maximum Product Subarray(LeetCode)
date: 2015-01-13T22:54:29+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=569
permalink: /maximum-product-subarrayleetcode/
views:
  - "95"
categories:
  - 计算机
---
题目： 

<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  Find the contiguous subarray within an array (containing at least one number) which has the largest product.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  For example, given the array&nbsp;<code style="box-sizing: border-box; font-family: Menlo, Monaco, Consolas, 'Courier New', monospace; font-size: 13px; padding: 2px 4px; color: rgb(199, 37, 78); border-radius: 4px; background-color: rgb(249, 242, 244);">[2,3,-2,4]</code>,&nbsp;the contiguous subarray&nbsp;<code style="box-sizing: border-box; font-family: Menlo, Monaco, Consolas, 'Courier New', monospace; font-size: 13px; padding: 2px 4px; color: rgb(199, 37, 78); border-radius: 4px; background-color: rgb(249, 242, 244);">[2,3]</code>&nbsp;has the largest product =&nbsp;<code style="box-sizing: border-box; font-family: Menlo, Monaco, Consolas, 'Courier New', monospace; font-size: 13px; padding: 2px 4px; color: rgb(199, 37, 78); border-radius: 4px; background-color: rgb(249, 242, 244);">6</code>.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  思路：


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  这道题的原型是求最大和的子序列，二者差别主要在于乘法和加法不太一样，如果是加法，动态规划只要设置一个保存最大值的一维数组f[N]，如果f[i]>0，那么f[i+1]=f[i]+A[i]，否则f[i+1]=A[i]，现在变成了乘法，显然不能直接套用加法的思路，因为f[i+1]不能直接从f[i]和A[i+1]推导出来，还需要看A[0-i]。


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  最暴力的解法很简单，就是把所有子序列A[i]*...A[j]迭代算出来，然后找到最大的，不过提交之后呵呵，很自然的内存爆了。。。


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  最后AC的思路又是参考了别人的博客。。。在动态规划迭代<span style="font-family: Arial, Tahoma, Helvetica, FreeSans, sans-serif; line-height: 18.4799995422363px;">维护一个局部最大的同时，再维护一个局部最小，这样如果下一个元素遇到负数时，就可能与这个最小相乘得到当前最大的乘积和，然后每次迭代过程再维护一个全局最大值。</span>


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  C++代码如下，其中maxProduct为AC的代码，maxProduct1为暴力算法：


<pre class="brush:cpp;">#include &lt;iostream&gt;
#include &lt;vector&gt;
#include &lt;stdlib.h&gt;
#include &lt;time.h&gt;

using namespace std;

class Solution {
public:
    long min(long x1, long x2) {
        return x1&lt;x2?x1:x2;
    }

    long max(long x1, long x2) {
        return x1&gt;x2?x1:x2;
    }

    /* fast solution! */
    long maxProduct(long A[], long n) {
        if(n==1) return A[0];
        long min_local = A[0], max_local = A[0], max_global = A[0];

        for (long i = 1; i &lt; n; ++i) {
            long max_local_old = max_local;
            max_local = max(A[i], max(A[i]*min_local, A[i]*max_local_old));
            min_local = min(A[i], min(A[i]*min_local, A[i]*max_local_old));
            max_global = max(max_local, max_global);
        }

        return max_global;
    }
    /* Memory Limit Exceeded */
    long maxProduct1(long A[], long n) {
        if(n==1) return A[0];

        long **f = new long *[n];
        for(long i=0; i&lt;n; ++i) {
            f[i] = new long[n];
            f[i][i] = A[i];
        }
        long max = A[0];
        for(long i=1; i&lt;n; ++i) {
            for(long j=0; j&lt;n-i; ++j) {
                f[j][j+i] = f[j][j+i-1]*A[j+i];
                if(f[j][j+i]&gt;max)
                    max = f[j][j+i];
            }
        }
        for(long i=0; i&lt;n; ++i)
            delete [] f[i];
        delete [] f;
        return max;
    }
};

#define MAX_NUM 10
int main(int argc, char *argv[])
{
    srand((unsigned)time(NULL));
    long n = 100;
    Solution s;
    long *A = new long[n];
    for(long i=0; i&lt;n; ++i) {
        A[i] = (rand()%MAX_NUM)-(rand()%MAX_NUM);
    }
    clock_t t1 = clock();
    long p1 = s.maxProduct(A, n);
    clock_t t2 = clock();
    long p2 = s.maxProduct1(A, n);
    clock_t t3 = clock();

    cout &lt;&lt; "algorithm 1 finished in " &lt;&lt; 1.0*(t2-t1)/CLOCKS_PER_SEC &lt;&lt; " seconds, result: " &lt;&lt; p1 &lt;&lt; endl;
    cout &lt;&lt; "algorithm 2 finished in " &lt;&lt; 1.0*(t3-t2)/CLOCKS_PER_SEC &lt;&lt; " seconds, result: " &lt;&lt; p2 &lt;&lt; endl;

    delete [] A;
    return 0;
}
</pre>

&nbsp;