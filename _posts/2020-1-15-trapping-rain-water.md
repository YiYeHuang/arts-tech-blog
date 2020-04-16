---
layout: post
title:  "Encounter the Hard: Trapping Rain Water"
date:   2019-11-07 15:59:53 -0700
categories: algorithm
tag: [leetcode, two-pointer, BFS, hard]
---

## Algorithm
又到了一看题就被吓傻的经典Hard题
![line4](/public/img/water3.png)
```text
TrappingRainWater

Given n non-negative integers representing an elevation map where the width of each bar is 1,
compute how much water it is able to trap after raining.

The above elevation map is represented by array [0,1,0,2,1,0,1,3,2,1,2,1].
In this case, 6 units of rain water (blue section) are being trapped. Thanks Marcos for contributing this image!

Example:

Input: [0,1,0,2,1,0,1,3,2,1,2,1]
Output: 6
```

这题开启了2D图，3D图吓人的传统（包括类似的skyline problem）， 十分容易直接迷失在图里而忽略了对本质算法的思考。

要了解这可以先从 `Container With Most Water`开始
![line4](/public/img/water1.png)
```text
Given n non-negative integers a1, a2, ..., an , where each represents a point at coordinate (i, ai).
n vertical lines are drawn such that the two endpoints of line i is at (i, ai) and (i, 0).
Find two lines, which together with x-axis forms a container, such that the container contains the most water.

Note: You may not slant the container and n is at least 2.


The above vertical lines are represented by array [1,8,6,2,5,4,8,3,7].
In this case, the max area of water (blue section) the container can contain is 49.

Example:

Input: [1,8,6,2,5,4,8,3,7]
Output: 49
```



#### 说3D 3D就到
![line4](/public/img/water5.png)
```text
407. Trapping Rain Water II

Given an m x n matrix of positive integers representing the height of each unit cell in a 2D elevation map,
compute the volume of water it is able to trap after raining.

Note:

Both m and n are less than 110. The height of each unit cell is greater than 0 and is less than 20,000.

Example:

Given the following 3x6 height map:
[
  [1,4,3,1,3,2],
  [3,2,1,3,2,4],
  [2,3,3,2,3,1]
]

Return 4.

The above image represents the elevation map [[1,4,3,1,3,2],[3,2,1,3,2,4],[2,3,3,2,3,1]] before the rain.

After the rain, water is trapped between the blocks. The total volume of water trapped is 4.
```