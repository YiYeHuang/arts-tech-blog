---
layout: post
title:  "Encounter Hard: Line Sweep Problems"
date:   2020-03-20 22:59:53 -0700
categories: algorithm
tag: [leetcode, line-sweep, hard]
---
 
### Tips
用图吓人是LC的优秀传统，`The Skyline Problem`又是其中的佼佼者。 
扫描线（Line Sweep）一直是没有特别吃透的问题之一，reference [花花酱](https://www.youtube.com/channel/UC5xDNEcvb1vgw3lE21Ack2Q)  
特意整理了扫描线的题目的思路和思考办法。 

```
850. Rectangle Area II

We are given a list of (axis-aligned) rectangles.  Each rectangle[i] = [x1, y1, x2, y2] , 
where (x1, y1) are the coordinates of the bottom-left corner, 
and (x2, y2) are the coordinates of the top-right corner of the ith rectangle.

Find the total area covered by all rectangles in the plane.  
Since the answer may be too large, return it modulo 10^9 + 7.

Example 1:

Input: [[0,0,2,2],[1,0,2,3],[1,0,3,1]]
Output: 6
Explanation: As illustrated in the picture.
Example 2:

Input: [[0,0,1000000000,1000000000]]
Output: 49
Explanation: The answer is 10^18 modulo (10^9 + 7), which is (10^9)^2 = (-7)^2 = 49.
Note:

1 <= rectangles.length <= 200
rectanges[i].length = 4
0 <= rectangles[i][j] <= 10^9
The total area covered by all rectangles will never exceed 2^63 - 1 and thus will fit in a 64-bit signed integer.
```
![line2](/public/img/scan2.png)
天际线问题的简易解题版，能用`line sweep`, `union find`多种办法解决  
题目故名思议是需要得到长方形的面积，并且长方形可以随意的重合。
这题和天际线的解决方法不同，扫描Y轴方向更为简单。但是为了两题一起分析，这里一起做X轴扫描解题。

![line1](/public/img/scan1.png)
先假设所有的方块都是分开的，题目变得就很简单，所有的扫描到的第一个Y轴的高度都保持不变，面积也就变的很好求解。

以长方形 [0,1]为例，四个坐标点分别为： 
```
(0,0), (0,2), (1,2), (1,0)
```

Line Sweep几个重要的思路
- X 轴向移动，所以所有的 `-->`箭头都可以忽略，loop的过程就是模拟X轴横向的过程
- 剩下所有的事件就是向上和向下
     - 记录为进入事件和离开事件
     - 进入事件标记为`1`离开事件标记为`-1`
     - 扫描到进入事件的坐标，累加1，扫描到离开事件的坐标，累加-1
     - 完完整一个图形以后，整个图的累积坐标归零
- 大部分问题都需要sorting
     - during sorting handle tie
     - use data structure sort during insert

图一为例，记
```
(0,0)  1
(0,2) -1  // 由0,0进入，0,2离开
(1,2)  1
(1,0) -1  // 由1,2进入，1,0离开
```
按照X轴升序排序，X轴相等Y轴有小到大
```
(0,0), (0,2), (1,0), (1,2)
扫描x = 0 得到高度为2（0有唯一进入事件，2有唯一离开事件）
- Y轴0坐标有进入事件1, Y轴2坐标有离开事件-1，总和为0，但是没有归零
- 表示长方形还需要一个2坐标进入事件一个0坐标离开事件
扫描x = 1 得到面积为2 => (1-0) * 2
- 事件归零
```
但是图形一重叠，问题就变的很复杂
![line3](/public/img/scan3.png)
分析
```
扫描x = 0 得到高度为2（0有唯一进入事件，2有唯一离开事件）
- Y轴0坐标有进入事件1, Y轴2坐标有离开事件-1，总和为0，但是没有归零
- 表示长方形还需要一个2坐标进入事件一个0坐标离开事件
事件值累计
Y   event
0 -  1  Y轴0坐标有进入事件
2 - -1  Y轴2坐标有离开事件

扫描x = 1 Y轴0坐标又得到一个进入事件，Y轴3坐标有一个离开事件，表示有重合

x=1事件值累计
Y   event
0 -  2  Y轴0坐标有两次进入事件
2 - -1  Y轴2坐标有一次离开事件
3 - -1  Y轴3坐标有一次离开事件
累计面积(1 - 0) * 2

扫描x = 1 Y轴3坐标又得到一个离开事件，Y轴0坐标有一个离开事件，第二个方块归零

x=2事件值累计
Y   event
0 -  1
2 - -1
3 -  0
累计面积(2 - 1) * 3

由此总结计算正确的重合高度
- 事件累计值为0的数字不做计算
- 一次进入一次离开为计算高度的标准
- 如果有更多的离开事件(-1)则此时的高度会升高
- 如果有值归零或者降低，表示有长方形扫描完毕，高度可能会降低
```
![line4](/public/img/scan4.png)
```
新的例子：
x=1事件值累计
Y   event
0 -  2  Y轴0坐标有两次进入事件
2 - -1  Y轴2坐标有一次离开事件
3 - -1  Y轴3坐标有一次离开事件
4 - -1  Y轴4坐标有一次离开事件
此时最大高度为4

x=2事件值累计
Y   event
0 -  2  
2 - -1  
3 -  0  
4 - -1  
长方形2扫描完毕，长方形3依旧没有归零，最大高度保持为4
```
解题
- input为`rectangle[i] = [x1, y1, x2, y2]`
     - 为左下角和右上角
     - 需要补充 (x1, y2) 左上角， (x2, y1) 右下角
     - 左下和右上为进入事件，左上和右下为离开事件
- 按X升序排序所有的点
- 一边扫描X轴一边记录Y轴的累计事件值
- 扫描到新的X轴的值的时候，累计之前的Y轴的高度的面积

```java
public class RectangleAreaII {

    class Node {
      int x, y, val;
      Node(int x, int y, int val) {
        this.x = x;
        this.y = y;
        this.val = val;
      }

      public String toString() {
        return x + " " + y + " " + val;
      }
    }

    public int rectangleArea(int[][] rectangles) {
      int M = 1000000007;
      List<Node> data = new ArrayList<>();

      for (int[] r : rectangles) {
        data.add(new Node(r[0], r[1], 1)); // x1, y1, bottom left, entry point
        data.add(new Node(r[0], r[3], -1)); // x1, y2, up left, leaving point
        data.add(new Node(r[2], r[1], -1)); // x2, y1, bottom right, leaving point
        data.add(new Node(r[2], r[3], 1));  // x2, y2, up left, entry point
      }

      Collections.sort(data, (a, b) -> {
        if (a.x == b.x) {
          return b.y - a.y;
        }
        return a.x - b.x;
      });

      TreeMap<Integer, Integer> map = new TreeMap<>();
      int preX = -1;
      int preY = -1;
      int result = 0;

      for (int i = 0; i < data.size(); i++) {
        Node p = data.get(i);
        System.out.println(p);

        map.put(p.y, map.getOrDefault(p.y, 0) + p.val);

        if (i == data.size() - 1 || data.get(i + 1).x > p.x) {
          if (preX > -1) {
            result += ((long)preY * (p.x - preX)) % M;
            result %= M;
          }
          preY = getY(map);
          preX = p.x;
        }
      }
      return result;
    }

    private int getY(TreeMap<Integer, Integer> map) {
      int result = 0;
      int previousCutPoint = -1;
      int count = 0;

      for (Map.Entry<Integer, Integer> e : map.entrySet()) {
        if (previousCutPoint >= 0 && count > 0) {
          result += e.getKey() - previousCutPoint;
        }
        count += e.getValue();
        previousCutPoint = e.getKey();
      }
      return result;
    }
}
```

### The Skyline problem
```
218. The Skyline Problem

A city's skyline is the outer contour of the silhouette formed by all the buildings in that city when viewed from a distance.
Now suppose you are given the locations and height of all the buildings as shown on a cityscape photo (Figure A),
write a program to output the skyline formed by these buildings collectively (Figure B).

Buildings Skyline Contour
The geometric information of each building is represented by a triplet of integers [Li, Ri, Hi],
where Li and Ri are the x coordinates of the left and right edge of the ith building, respectively,
and Hi is its height. It is guaranteed that 0 ? Li, Ri ? INT_MAX, 0 < Hi ? INT_MAX, and Ri - Li > 0.
You may assume all buildings are perfect rectangles grounded on an absolutely flat surface at height 0.

For instance, the dimensions of all buildings in Figure A are recorded as:
[ [2 9 10], [3 7 15], [5 12 12], [15 20 10], [19 24 8] ] .

The output is a list of "key points" (red dots in Figure B) in the format of
[ [x1,y1], [x2, y2], [x3, y3], ... ]
that uniquely defines a skyline.
A key point is the left endpoint of a horizontal line segment. Note that the last key point,
where the rightmost building ends, is merely used to mark the termination of the skyline,
and always has zero height.

Also, the ground in between any two adjacent buildings should be considered part of the skyline contour.

For instance, the skyline in Figure B should be represented as:
[ [2 10], [3 15], [7 12], [12 0], [15 10], [20 8], [24, 0] ].

Notes:

The number of buildings in any input list is guaranteed to be in the range [0, 10000].
The input list is already sorted in ascending order by the left x position Li.
The output list must be sorted by the x position.
There must be no consecutive horizontal lines of equal height in the output skyline.
For instance, [...[2 3], [4 5], [7 5], [11 5], [12 7]...] is not acceptable;
the three lines of height 5 should be merged into one in the final output as such: [...[2 3], [4 5], [12 7], ...]
```
图画和上一题一毛一样，但是要求的是算天际线，输出的是关键的点（比算总长要简单不少）  
输入的格式有变化：
```
[2 9 10] 为一个方块，使用同一个code模板可以拆分成
[2,0]左下进入事件, [9,0] 右下离开事件
[2, 10]左上离开事件, [9, 10] 右上进入事件
```
其余的做法类似
- 排序所有的点，按照x轴的值从小到大
- 记录所有的进出事件
- 遍历所有的点，用tree map记录y轴的值，从大到小
- 一边扫描x轴一边用local variable记录上一个y轴的大小，如果发生改变，就记录点（以防记录非关键点）
- 不停更新y轴的高度

```java
public class TheSkylineProblem {

  class Node {
    int x, y, val;
    Node(int x, int y, int val) {
      this.x = x;
      this.y = y;
      this.val = val;
    }

    public String toString() {
      return x + " " + y + " " + val;
    }
  }
  
  public List<List<Integer>> getSkyline(int[][] buildings) {
    List<Node> data = new ArrayList<>();

    // [2 9 10] ->
    // [2,0] in, [9,0] in
    // [2, 10] out, [9, 10] out
    for (int[] r : buildings) {
      data.add(new Node(r[0], 0, 1)); // x1, y1, bottom left, entry point
      data.add(new Node(r[1], 0, -1)); // x2, y1, bottom right, leaving point
      data.add(new Node(r[0], r[2], -1)); // x1, y2, up left, leaving point
      data.add(new Node(r[1], r[2], 1)); // x2, y2, up left, entry point
    }

    Collections.sort(data, (a, b) -> {
      if (a.x == b.x) {
        return b.y - a.y;
      }
      return a.x - b.x;
    });

    TreeMap<Integer, Integer> map = new TreeMap<>();
    int preX = -1;
    int preY = -1;
    int lastHight = -1;

    List<List<Integer>> result = new ArrayList<>();

    for (int i = 0; i < data.size(); i++) {
      Node p = data.get(i);
      // System.out.println(p);

      map.put(p.y, map.getOrDefault(p.y, 0) + p.val);

      if (i == data.size() - 1 || data.get(i + 1).x > p.x) {
        if (preX > -1) {
          if (lastHight == -1 || lastHight != preY) {
            System.out.println("RESULT " + preX + " " + preY);
            List<Integer> resultSet = new ArrayList<>();
            resultSet.add(preX);
            resultSet.add(preY);
            result.add(resultSet);

            lastHight = preY;
          }
        }

        preY = calcY(map);
        preX = p.x;

        if (preY == 0 && i == data.size() - 1 ) {
          System.out.println("RESULT get to 0 " + preX + " " + preY);
          List<Integer> resultSet = new ArrayList<>();
          resultSet.add(preX);
          resultSet.add(preY);
          result.add(resultSet);
        }
      }
    }
    return result;
  }

  private int calcY(TreeMap<Integer, Integer> map) {
    int result = 0;
    int previousCutPoint = -1;
    int count = 0;

    for (Map.Entry<Integer, Integer> e : map.entrySet()) {
      if (previousCutPoint >= 0 && count > 0) {
        result += e.getKey() - previousCutPoint;
      }
      count += e.getValue();
      previousCutPoint = e.getKey();
    }
    return result;
  }
}
```