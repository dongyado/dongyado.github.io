---
id: 202
title: Gas Station(LeetCode)
date: 2014-08-25T17:49:36+00:00
author: nicklhy
layout: post
guid: http://closure11.com/?p=202
permalink: /gas-stationleetcode/
views:
  - "72"
categories:
  - 计算机
---
题目： 

There are&nbsp;_N_&nbsp;gas stations along a circular route, where the amount of gas at station&nbsp;_i_&nbsp;is&nbsp;<code style="color: #c7254e;">gas[i]</code>. 

You have a car with an unlimited gas tank and it costs&nbsp;<code style="color: #c7254e;">cost[i]</code>&nbsp;of gas to travel from station&nbsp;_i_&nbsp;to its next station (_i_+1). 

You begin the journey with an empty tank at one of the gas stations. 

Return the starting gas station's index if you can travel around the circuit once, otherwise return -1. 

**Note:** The solution is guaranteed to be unique. 

思路： 最简单的思路，每个元素遍历，判断其是否可以完成绕圈任务，复杂度 $$O(N^2)$$ ，提交之后果断TLE。。。 

需要加入一个小trick，如果上一次遍历，以i为起点，到j的时候发现gas不够，则下一次不应该从i+1开始，而应该从j+1开始，原因很简单，如果gas[i]>cost[j]，那么遍历i点应该是可以对i+1到j产生好处的，以此类推，i到j-1应该可以对遍历j产生好处（i到j-1之后，剩余的汽油大于0），所以若下一次仍从i+1开始，显然不可能通过。 

代码如下： 

<pre class="brush:python;">class Solution:
	# @param gas, a list of integers
	# @param cost, a list of integers
	# @return an integer
	def canCompleteCircuit(self, gas, cost):
		N = len(gas)
		if N==1:
			if gas[0]>=cost[0]:
				return 0
			else:
				return -1
		if sum(gas)&lt;sum(cost):
			return -1
		i=0
		while i&lt;N:
			flag = True
			num = 0
			for j in xrange(N):
				num += gas[(i+j)%N]
				if num&lt;cost[(i+j)%N]:
					flag = False
					i = (i+j)%N+1
					break
				num -= cost[(i+j)%N]
			if flag:
				return i
		return -1</pre>

&nbsp;