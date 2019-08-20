---
layout: post
title:  "Dynamic sliding questions - part 1"
date:   2019-08-18 15:59:53 -0700
categories: algorithm
tag: [leetcode, queue, priority queue, slidingWindow, hard]
---

## Algorithm
没有刷题经验的时候碰到类似需要滑动窗口解题思路的时候，一紧张很容易被吓的用DP来解题，碰到Moving Maximum这样的题甚至会被按在地上摩擦。

Sliding Windown的题目特征是会给一个很明显的K区间，然后围绕这个区间搞事情。
解题主要数据结构有一下：
- 通解map: 记录value和position
- queue: 如果数据不需要经常做全局计算
- Max/Min Heap: 处理动态+需要实时更新的计算

### Queue解题
```test
346 Moving Average from Data Stream

Given a stream of integers and a window size, calculate the moving average of
all integers in the sliding window.

For example, MovingAverage
m = new MovingAverage(3);
m.next(1) = 1
m.next(10)= (1 + 10) / 2
m.next(3) = (1 + 10 + 3) / 3
m.next(5) = (10 + 3 + 5) / 3
```

典型的无法再典型的队列结构应用
- 在K区间随时保持进出控制
- 需要一个global sum记录目前的和的更新。

```java
public class MovingAverageFromDataStream
{
    Queue<Integer> cache = new LinkedList<>();

    int maxSize = 0;
    int currentSum = 0;
    int currentCount = 0;

    /** Initialize your data structure here. */
    public MovingAverageFromDataStream(int size)
    {
        maxSize = size;
    }

    public double next(int val)
    {
        if (currentCount < maxSize)
        {
            cache.add(val);
            currentCount++;
            currentSum += val;
            return (double)(currentSum) / (double)(currentCount);
        }
        else
        {
            currentSum -= cache.poll();
            currentSum += val;
            cache.add(val);
            return (double)(currentSum) / (double)(currentCount);
        }
    }
}
```

### map解题

```test
159. Longest Substring with At Most Two Distinct Characters

Given a string, find the length of the longest substring T that contains at most 2 distinct characters.
For example, Given s = “eceba”,
T is "ece" which its length is 3.

or 变形题

904. Fruit Into Baskets

In a row of trees, the i-th tree produces fruit with type tree[i].
You start at any tree of your choice, then repeatedly perform the following steps:

Add one piece of fruit from this tree to your baskets.If you cannot, stop.
Move to the next tree to the right of the current tree.If there is no tree to the right, stop.
Note that you do not have any choice after the initial choice of starting tree: you must perform step 1, then step 2, then back to step 1, then step 2, and so on until you stop.

You have two baskets, and each basket can carry any quantity of fruit, but you want each basket to only carry one type of fruit each.

What is the total amount of fruit you can collect with this procedure?

Example 1:

Input: [1,2,1]
Output: 3
Explanation: We can collect [1,2,1].
Example 2:

Input: [0,1,2,2]
Output: 3
Explanation: We can collect [1,2,2].
If we started at the first tree, we would only collect [0, 1].
Example 3:

Input: [1,2,3,2,2]
Output: 4
Explanation: We can collect [2,3,2,2].
If we started at the first tree, we would only collect [1, 2].
Example 4:

Input: [3,3,3,1,2,1,1,2,3,3,4]
Output: 5
Explanation: We can collect [1,2,1,1,2].
If we started at the first tree or the eighth tree, we would only collect 4 fruits.

Note:

1 <= tree.length <= 40000
0 <= tree[i] < tree.length
```
LC 159十分简洁明了，但是1046新题用一种非常膈应的表达重新解释了一遍`159. Longest Substring with At Most Two Distinct Characters`, 初见解题会非常卡。

从159的解释上解题，求解最长的只含有两种字母的字符串，这题可以通解延伸至k种字母的字符串。
即存在 end - start, 使得end - start符合只含有两种字母的字符串并且end - start值最大。

解题思路
- 用Map处理sliding window的问题
- 适用two pointer
- 需要几个cache的值保存先前的结果

使用例子`abbaaceebd`
- 以为适用的two pointer, map用来保存字符和出现频率，不需要保存index，检测到`abbaa`的时候，map达到最大size临界点2，end = 4, start = 0， 当前最大result为5， map不做任何剔除。
- 检测到`abbaac`的时候，map size突破二，需要触发新剔除程序，使的map size重新回到2，便缩进start
- 当剔除到`aac`的时候，map内部为 a(2), b(0), c(1), 需要剔除b, 新的result大小为3，3<5不做更新。
- 以此类推，走完整个array

核心逻辑
```java
while (count.size() > 2) {
    count.put(tree[head], count.get(tree[head]) - 1);
    if (count.get(tree[head]) == 0) {
        count.remove(tree[head]);
    }
    head++;
}
```

完整解题
```java
public static int solution(int[] list) {
    Map<Integer, Integer> count = new HashMap<>();
    int res = 0, head = 0;
    for (int end = 0; end < list.length; ++end) {
        count.put(list[end], count.getOrDefault(list[end], 0) + 1);
        while (count.size() > 2) {
            count.put(list[head], count.get(list[head]) - 1);
            if (count.get(trlistee[head]) == 0) {
                count.remove(list[head]);
            }
            head++;
        }
        res = Math.max(res, end - head + 1);
    }
    return res;
}
```