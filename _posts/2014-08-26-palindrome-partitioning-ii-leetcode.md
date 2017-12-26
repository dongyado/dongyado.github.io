---
id: 212
title: Palindrome Partitioning II (LeetCode)
date: 2014-08-26T16:18:49+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=212
permalink: /palindrome-partitioning-ii-leetcode/
views:
  - "230"
categories:
  - 计算机
---
题目： 

<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  Given a string&nbsp;<i style="box-sizing: border-box;">s</i>, partition&nbsp;<i style="box-sizing: border-box;">s</i>&nbsp;such that every substring of the partition is a palindrome.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  Return the minimum cuts needed for a palindrome partitioning of&nbsp;<i style="box-sizing: border-box;">s</i>.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  For example, given&nbsp;<i style="box-sizing: border-box;">s</i>&nbsp;=&nbsp;<code style="box-sizing: border-box; font-family: Monaco, Menlo, Consolas, 'Courier New', monospace; font-size: 13px; padding: 2px 4px; color: rgb(199, 37, 78); white-space: nowrap; border-top-left-radius: 4px; border-top-right-radius: 4px; border-bottom-right-radius: 4px; border-bottom-left-radius: 4px; background-color: rgb(249, 242, 244);">"aab"</code>,<br style="box-sizing: border-box;" /><br /> <br /> Return&nbsp;<code style="box-sizing: border-box; font-family: Monaco, Menlo, Consolas, 'Courier New', monospace; font-size: 13px; padding: 2px 4px; color: rgb(199, 37, 78); white-space: nowrap; border-top-left-radius: 4px; border-top-right-radius: 4px; border-bottom-right-radius: 4px; border-bottom-left-radius: 4px; background-color: rgb(249, 242, 244);">1</code>&nbsp;since the palindrome partitioning&nbsp;<code style="box-sizing: border-box; font-family: Monaco, Menlo, Consolas, 'Courier New', monospace; font-size: 13px; padding: 2px 4px; color: rgb(199, 37, 78); white-space: nowrap; border-top-left-radius: 4px; border-top-right-radius: 4px; border-bottom-right-radius: 4px; border-bottom-left-radius: 4px; background-color: rgb(249, 242, 244);">["aa","b"]</code>&nbsp;could be produced using 1 cut.


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  思路：


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  Palindrome Partitioning需要收集所有回文切分的解，而这一题只需要求最少的切分次数，如果直接由原来的算法，然后搜索最小切分次数的解，复杂度是 $$O(N^3)$$ ，提交后TLE。


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  这里需要利用两次DP：


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  1）首先定义一个二维数组flag[N][N]，其中flag[i][j]表示子字符串s[i:j+1]是否是回文，显然若i==j，即长度为1的字符串，则f[i][i]=True，若i+1==j，则需要再查看s[i]和s[i+1]是否相等，如果相等则为True，若j-i>=2，同样先看s[i]和s[j]，如果相等，则返回flag[i+1][j-1]，否则直接设置为False；


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  2）再定义一个记录最小切分次数的一维数组count，其中count[i]表示子字符串s[:i+1]的最小切分次数，如果flag[0][i]==True，则为0，否则需要寻找最小的切分方式j，使得count[j]+1最小，且s[j+1:i+1]是回文。


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  不过可惜，思路虽然没错，但是Python实现依旧TLE，本地跑了一下超时的数据，大概0.807s左右，后来陈大哥用C++实现，AC......


<p style="box-sizing: border-box; margin: 0px 0px 10px; font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif; font-size: 14px; line-height: 30px;">
  网上看到一个更加简短的Python代码，非常赞，不过同样TLE，一起Po上来吧，三份代码如下，其中C++版本由同实验室陈大哥提供，Python版后面有几个测试数据。。。


<pre class="brush:cpp;">bool f[0x1010][0x1010];
class Solution {
private:
    int g[0x1010];
    string str;

public:
    int minCut(string s) {
        str = s;
        int n = str.length();

        for (int i = 0; i &lt; n; ++i) {
            for (int j = 0; j &lt;= i; ++j) {
                f[i][j] = true;
            }
        }
        for (int i = 1; i &lt; n; ++i) {
            for (int j = 0; j &lt; n; ++j) {
                if (str[j] == str[j + i]) f[j][j + i] = f[j + 1][j + i - 1];
                else f[j][j + i] = false;
            }
        }

        g[0] = 0;
        for (int i = 1; i &lt; n; ++i) {
            if (f[0][i]) g[i] = 0;
            else {
                g[i] = 0x1010;
                for (int j = 0; j &lt; i; ++j) {
                    if (f[j + 1][i]) g[i] = min(g[i], g[j] + 1);
                }
            }
        }
        return g[n - 1];
    }
};</pre>

<pre class="brush:python;">import time

class Solution:
	# @param s, a string
	# @return a list of lists of string
	def minCut(self, s):
		N = len(s)
		dp = [N-j for j in range(N+1)]
		p = [[False for i in range(N)] for j in range(N)]
		for i in xrange(N-1,-1,-1):
			for j in xrange(i, N):
				if s[i] == s[j] and (((j - i) &lt; 2) or p[i+1][j-1]):
					p[i][j] = True
					dp[i] = min(1+dp[j+1], dp[i])
		return dp[0]-1
	
	def minCut2(self, s):
		N = len(s)
		if N&lt;=1:
			return [[s]]
		count = [-1 for j in range(N)]
		flag = [[False for j in range(N)] for i in range(N)]
		for i in xrange(N):
			flag[i][i] = True
			if i+1&lt;N and s[i]==s[i+1]:
				flag[i][i+1] = True
		for i in xrange(2,N):
			for j in xrange(N-i):
				if s[j]==s[j+i] and flag[j+1][j+i-1]:
					flag[j][j+i] = True
		
		count[0] = 0
		for i in xrange(1,N):
			if flag[0][i]:
				count[i] = 0
			else:
				count[i] = i
				for j in xrange(i):
					if flag[j+1][i]:
						count[i] = min(count[i],count[j]+1)
		return count[N-1]

s = Solution()
print s.minCut2(&#39;aab&#39;)
print s.minCut2("fifgbeajcacehiicccfecbfhhgfiiecdcjjffbghdidbhbdbfbfjccgbbdcjheccfbhafehieabbdfeigbiaggchaeghaijfbjhi")
print s.minCut2("apjesgpsxoeiokmqmfgvjslcjukbqxpsobyhjpbgdfruqdkeiszrlmtwgfxyfostpqczidfljwfbbrflkgdvtytbgqalguewnhvvmcgxboycffopmtmhtfizxkmeftcucxpobxmelmjtuzigsxnncxpaibgpuijwhankxbplpyejxmrrjgeoevqozwdtgospohznkoyzocjlracchjqnggbfeebmuvbicbvmpuleywrpzwsihivnrwtxcukwplgtobhgxukwrdlszfaiqxwjvrgxnsveedxseeyeykarqnjrtlaliyudpacctzizcftjlunlgnfwcqqxcqikocqffsjyurzwysfjmswvhbrmshjuzsgpwyubtfbnwajuvrfhlccvfwhxfqthkcwhatktymgxostjlztwdxritygbrbibdgkezvzajizxasjnrcjwzdfvdnwwqeyumkamhzoqhnqjfzwzbixclcxqrtniznemxeahfozp")
t1 = time.time()
print s.minCut2("aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaabbaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa")
t2 = time.time()
print &#39;finished in %f seconds&#39; % (t2-t1)</pre>

&nbsp;