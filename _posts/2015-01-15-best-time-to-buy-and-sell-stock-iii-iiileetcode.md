---
id: 574
title: Best Time to Buy and Sell Stock I,II, III(LeetCode)
date: 2015-01-15T17:48:08+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=574
permalink: /best-time-to-buy-and-sell-stock-iii-iiileetcode/
views:
  - "183"
categories:
  - 计算机
---
<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  第一题：


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  Say you have an array for which the&nbsp;<i style="box-sizing: border-box;">i</i><span style="box-sizing: border-box; position: relative; font-size: 11px; line-height: 0; vertical-align: baseline; top: -0.5em;">th</span>&nbsp;element is the price of a given stock on day&nbsp;<i style="box-sizing: border-box;">i</i>.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  If you were only permitted to complete at most one transaction (ie, buy one and sell one share of the stock), design an algorithm to find the maximum profit.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  分析：


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  题目要求找到利润最大的一笔交易，暴力法很简单，遍历每个元素，然后查找其右边最大的值，记录差值最大的一组，复杂度为 $$O(N^2)$$ ，那么有没有可能只扫描一遍就得到这个最大差值？一开始考虑的时候我就想复杂了，后来发现，其实只要从左到右依次扫描，用一个变量保存遍历到第i个元素时的最大<span style="font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">利润</span>，再用另一个变量保存左边最小值，每次遍历到一个新的元素时，如果它和左边最小值的差值大于之前的最大利润，则更新最大利润，如果这个新的元素小于旧的最小值，也把这个旧的最小值更新。


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  代码（由于提交一次过了，所以没有本地测试的程序）：


<pre class="brush:cpp;">class Solution {
public:
    int maxProfit(vector&lt;int&gt; &prices) {
        if(prices.size()&lt;=1) return 0;
        int max = 0, min_price = prices[0];
        for(int i=1; i&lt;prices.size(); ++i) {
            int diff = prices[i]-min_price;
            if(diff&gt;max) max = diff;
            if(min_price&gt;prices[i]) min_price = prices[i];
        }
        return max;
    }
};</pre>

&nbsp; 

第二题： 

<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  Say you have an array for which the&nbsp;<i style="box-sizing: border-box;">i</i><span style="box-sizing: border-box; position: relative; font-size: 11px; line-height: 0; vertical-align: baseline; top: -0.5em;">th</span>&nbsp;element is the price of a given stock on day&nbsp;<i style="box-sizing: border-box;">i</i>.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  Design an algorithm to find the maximum profit. You may complete as many transactions as you like (ie, buy one and sell one share of the stock multiple times). However, you may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again).


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  分析：


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  这题允许无限次交易，只不过不能同时操作多笔。解法也很简单，从左到右遍历，检查a[i]-a[i-1]，如果大于0，就累加到利润上，否则舍弃。


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  代码：


<pre class="brush:cpp;">class Solution {
public:
    int maxProfit(vector&lt;int&gt; &prices) {
        if(prices.size()&lt;=1) return 0;
        int sum = 0;
        for(int i=1; i&lt;prices.size(); ++i) {
            if(prices[i]-prices[i-1]&gt;0)
                sum+=prices[i]-prices[i-1];
        }
        return sum;
    }
};</pre>

&nbsp; 

第三题： 

<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  Say you have an array for which the&nbsp;<i style="box-sizing: border-box;">i</i><span style="box-sizing: border-box; position: relative; font-size: 11px; line-height: 0; vertical-align: baseline; top: -0.5em;">th</span>&nbsp;element is the price of a given stock on day&nbsp;<i style="box-sizing: border-box;">i</i>.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  Design an algorithm to find the maximum profit. You may complete at most&nbsp;<i style="box-sizing: border-box;">two</i>&nbsp;transactions.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  <span style="box-sizing: border-box; font-weight: 700;">Note:</span><br style="box-sizing: border-box;" />


You may not engage in multiple transactions at the same time (ie, you must sell the stock before you buy again). 

<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  分析：


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  这题会复杂一些，暴力的方法可以借助第一题的方法，把整个价格数组分成第1天到第i天和第i+1天到第n天两段，然后分别计算两边的最大利润，最后线性遍历一次i查找最大的一组即可，这种解法复杂度也是 $$O(N^2)$$ ，想要优化，可以从求解i两边最大利润入手。


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  暴力解法在每次求解第i天的时候完全没有使用到之前的状态，显然不符合动态规划的思想，那么，前i天和前i+1天的最大利润有什么关系？能不能在O(1)时间内从前i天最大利润推出前i+1天的？显然，答案是yes。。。我们可以用数组f[n]存储前i天最大利润（i=1~n），再用一个变量min表示前i天最低股价，那么当计算f[i+1]时，如果prices[i+1]-min>f[i]，则可以更新f[i+1]为新值，否则赋为f[i]（表示和前i天的最大利润一样），如果prices[i+1]<min，那么更新min。类似的我们也可以计算右边一部分，只不过遍历时注意要从右往左。


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  另外，注意不一定要做两笔利润大于0的交易，可以有一笔当天买入再卖出（或者说只进行一次交易）。


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  代码：


<pre class="brush:cpp;">class Solution {
public:
    int maxProfit(vector&lt;int&gt; &prices) {
        if(prices.size()&lt;=1) return 0;
        if(prices.size()==2) return (prices[1]-prices[0])>0?(prices[1]-prices[0]):0;
        int global = 0;

        int *f = new int[prices.size()];
        int *g = new int[prices.size()];

        f[0] = 0;
        g[prices.size()-1] = 0;

        int min_l = prices[0], max_r = prices[prices.size()-1];

        for (int i = 1; i &lt; prices.size(); ++i) {
            if( prices[i]-min_l>f[i-1] )
                f[i] = prices[i]-min_l;
            else
                f[i] = f[i-1];
            if( prices[i]&lt;min_l )
                min_l = prices[i];
            if( f[i]&gt;global )
                global = f[i];
        }

        for (int i = prices.size()-2; i &gt;=0 ; --i) {
            if( max_r-prices[i]>g[i+1] )
                g[i] = max_r-prices[i];
            else
                g[i] = g[i+1];
            if( prices[i]&gt;max_r )
                max_r = prices[i];
            if( g[i]&gt;global )
                global = g[i];
        }

        for( int i=1; i&lt;prices.size()-1; ++i) {
            if( f[i]+g[i+1]&gt;global )
                global = f[i]+g[i+1];
        }

        delete [] f;
        delete [] g;

        return global;
    }
};</pre>

&nbsp;