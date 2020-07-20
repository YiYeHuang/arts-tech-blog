---
layout: post
title:  "Encounter Hard: TrappingRainWater"
date:   2020-4-20 22:59:53 -0700
categories: algorithm
tag: [leetcode, heap, hard]
---

### Algorithm
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
整个“算水”系列的题的核心就是围绕木桶效应，即木桶最短板是盛水极限。

`Container With Most Water` 的计算很简单，底*最短的一边的木板。
由于木桶效应，这类题不能用`line sweep`的办法来做，锁头锁尾，那用`two pointer`就很明显了
- 锁头所尾巴
- 比较两边的高度，缩进较短的一边
  - 缩进一定是可取的，因为x轴每次固定损失`1`, y轴的增益很可能远大于`1`
- 用一个local variable记录最大值

```java
public class ContainerWithMostWater {

    public int maxArea(int[] height) {
        int left = 0;
        int right = height.length - 1;
        int maxArea= 0;

        // Need to go all over the list to avoid case like
        // [1,8,6,2,11111,11111,8,3,7]
        while (left < right) {
            maxArea = Math.max(maxArea, 
            Math.min(height[left], height[right]) * (right - left));

            // before move, maxArea = height[left] * (right - left)
            // when move by 1, right - left is reduce by 1, 
            // but left is at least increase by 1, so can not be smaller
            if (height[left] < height[right]) {
                left++;
            } else {
                right--;
            }
        }

        return maxArea;
    }
```

#### 回到 TrappingRainWater

- 核心思想也是一样，木桶效应，`two pointer`
- 比较两边的高度，缩进较短的一边,
  - 但是`Container With Most Water`的情况是不停更换拦截的高度的大小
  - `TrappingRainWater`每一根柱子的高度都有作用
  - 需要用local variable，`leftMax`和 `rightMax`记录目前桶的两边的最大高度
  - 缩进时需要时刻跟新`leftMax`和 `rightMax`的大小
- 计算方法有区别，需要逐格子计算水的含量
  - 累积 `leftMax`/`rightMax` 和当前高度的差

```java
public class TrappingRainWater {

    public static int trap(int[] height) {

        if (height == null || height.length == 0) {
            return 0;
        }

        int result = 0;
        int leftMax = Integer.MIN_VALUE;
        int rightMax = Integer.MIN_VALUE;

        int left = 0;
        int right = height.length - 1;

        // question is very close to 11, container with most water
        // we assuming nothing blocks between left most and right most
        // the water always bound by the lower bar, water will not overflow from higher
        // so always go to the higher bar
        // count the diff of each index, if leftMax or rightMax refreshed, that position will accumulate 0 volume of water
        // meet at middle
        while (left <= right) {
            leftMax = Math.max(leftMax, height[left]);
            rightMax = Math.max(rightMax, height[right]);

            if (leftMax < rightMax) {
                result += leftMax - height[left];
                left++;
            } else {
                result += rightMax - height[right];
                right--;
            }
        }

        return result;
    }
}
```

#### 说3D 3D就到
![line4](/public/img/water5.png)
原谅我的灵魂画手
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

The above image represents the elevation map 
[[1,4,3,1,3,2],[3,2,1,3,2,4],[2,3,3,2,3,1]] before the rain.

After the rain, water is trapped between the blocks. 
The total volume of water trapped is 4.
```

- 图虽然很吓人，但是木桶效应为核心还是没有变
  - 但是不能再`two pointer`了，平面的图用`two pointer`, 三维的就是用四边了
  - 可以先假设四边是最高的部分，那四边的任何一个格子最短，水就会从那里漏出来
  - 需要用一个不停找到最小的数据结构，所以选择Heap做这题。
- 先把所有的边的高度push进min heap
- pop heap
  - 第一个出来的一定是边上最短的一节
- 然后就是matrix的典型BFS搜索
  - 向相邻的四个方向出发`{-1, 0}, {1, 0}, {0, -1}, {0, 1}`
  - 检查边界
  - 需要isVisited的boolean matrix做辅助

![line4](/public/img/water6.png)

主while loop
  - pop next得到`next lowest`
  - 向相邻的四个方向出发`{-1, 0}, {1, 0}, {0, -1}, {0, 1}`
       - 检查边界, 检查新的坐标有没有被计算过
         - 计算 `next lowest`和current bar高度差
         - 因为 `next lowest`已经是当前最低的bar，所以一定能挡住所有的水
         - 标记新坐标已经visit
         - push进min heap

```java
public class TrappingRainWaterII {

    class Bar {
        int row;
        int col;
        int height;
        public Bar(int row, int col, int height) {
            this.row = row;
            this.col = col;
            this.height = height;
        }
    }

    public int trapRainWater(int[][] heightMap) {

        if (heightMap.length == 0) return 0;

        PriorityQueue<Bar> minHeap = new PriorityQueue<>((a, b)-> a.height - b.height);

        int h = heightMap.length;
        int w = heightMap[0].length;
        boolean[][] visited = new boolean[h][w];

        // Like Trapping rain water, in 2D all we care about is two side, in 3D, we care about broader.
        for (int i = 0; i < h; i++) {
            visited[i][0] = true;
            visited[i][w - 1] = true;
            minHeap.offer(new Bar(i, 0, heightMap[i][0]));
            minHeap.offer(new Bar(i, w - 1, heightMap[i][w - 1]));
        }
        for (int i = 0; i < w; i++) {
            visited[0][i] = true;
            visited[h - 1][i] = true;
            minHeap.offer(new Bar(0, i, heightMap[0][i]));
            minHeap.offer(new Bar(h - 1, i, heightMap[h - 1][i]));
        }

        // since using min heap, we can pop the shortest bar first, which is the lowest pointer where water can leak
        // when visited neighbour is lower, collect value.
        // int[][] dirs = (-1, 0), (1, 0), (0, -1), (0, 1);
        int result = 0;

        while(!minHeap.isEmpty()) {
            Bar bar = minHeap.poll();

            for (int[] dir : dirs) {
                int row = bar.row + dir[0];
                int col = bar.col + dir[1];

                if ((row >= 0 && row < h) && (col >= 0 && col < w) && !visited[row][col]) {
                    // not comparing the around unit, 
                    // coz it does not matter if the surrounding is lower
                    // if the current spot is lower
                    // this guarantee collect water, collect per each bar
                    visited[row][col] = true;
                    result += Math.max(0, bar.height - heightMap[row][col]);

                    // push a new bar in the min heap
                    // we dont always push the current bar height
                    // since the current one is visited,
                    // - if current spot is higher, push current height as this blocks more water
                    // - if current spot is lower, push the original blocking bar height.
                    minHeap.offer(new Bar(row, col, Math.max(heightMap[row][col], bar.height)));
                }
            }
        }

        return result;
    }
}
```