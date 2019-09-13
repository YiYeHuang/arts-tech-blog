---
layout: post
title:  "Dynamic sliding questions - part 2"
date:   2019-08-21 15:59:53 -0700
categories: algorithm
tag: [leetcode, priority queue, slidingWindow]
---

[Continue from part 1:](/algorithm/2019/08/18/sliding-questions1)
## Algorithm

Max heap和Min heap解体系列（土味名：大顶堆，小顶堆）
max heap与 min heap的构成使用了heap sort, 进推出推的速度都能达到O(logN)适用面非常广
第K大，第K小，第K频繁，第K近, 第K XXX, Merge K list都可以使用。

本篇博客主要分析heap在sliding题中的应用。

```test
 239. Sliding Window Maximum

 Given an array nums, there is a sliding window of size k which is moving from the very left of the array to the very right. You can only see the k numbers in the window. Each time the sliding window moves right by one position. Return the max sliding window.

 Example:

 Input: nums = [1,3,-1,-3,5,3,6,7], and k = 3
 Output: [3,3,5,5,6,7]
 Explanation:

 Window position                Max
 ---------------               -----
 [1  3  -1] -3  5  3  6  7       3
 1 [3  -1  -3] 5  3  6  7        3
 1  3 [-1  -3  5] 3  6  7        5
 1  3  -1 [-3  5  3] 6  7        5
 1  3  -1  -3 [5  3  6] 7        6
 1  3  -1  -3  5 [3  6  7]       7
 Note:
 You may assume k is always valid, 1 ≤ k ≤ input array's size for non-empty array.

 Follow up:
 Could you solve it in linear time?
 ```

分析：
一旦满足size K以后，便需要不停记录此时window中最大值。因为并不需要知道第二大或者第三大的数字，所以不需要进行局部sorting。
符合时刻保持最大值的数据结构，max heap当仁不让。

要点：
- 没有特别思考的难点，但是需要处理好index。依旧例遍整个数组，但是答案的大小是 n - k + 1.

代码
```java
public int[] maxSlidingWindowPQ(int[] nums, int k) {
    PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>((a, b)-> b - a);
    int n = nums.length;
    if (n <= 0) return new int[0];
    int[] result = new int[n - k + 1];

    for (int i = 0; i < nums.length; i++){
        maxHeap.offer(nums[i]);
        if (i - k >= 0) {
            maxHeap.remove(nums[i - k]);
        }

        // deal with the extra position
        if (i - k + 1 >= 0) {
            result[i - k + 1] = maxHeap.peek();
        }
    }

    return result;
}
```

```text
 480. Sliding Window Median
 
 Median is the middle value in an ordered integer list. If the size of the list is even, there is no middle value. So the median is the mean of the two middle value.

 Examples:
 [2,3,4] , the median is 3

 [2,3], the median is (2 + 3) / 2 = 2.5

 Given an array nums, there is a sliding window of size k which is moving from the very left of the array to the very right. You can only see the k numbers in the window. Each time the sliding window moves right by one position. Your job is to output the median array for each window in the original array.

 For example,
 Given nums = [1,3,-1,-3,5,3,6,7], and k = 3.

 Window position                Median
 ---------------               -----
 [1  3  -1] -3  5  3  6  7       1
 1 [3  -1  -3] 5  3  6  7       -1
 1  3 [-1  -3  5] 3  6  7       -1
 1  3  -1 [-3  5  3] 6  7       3
 1  3  -1  -3 [5  3  6] 7       5
 1  3  -1  -3  5 [3  6  7]      6
 Therefore, return the median sliding window as [1,-1,-1,3,5,6].

 Note:
 You may assume k is always valid, ie: k is always smaller than input array's size for non-empty array.
```

找median难度上了一整个台阶，map不好用，max heap也不好用。

分析：
- max heap 保持最大值在最顶
- min heap 保持最小值在最顶
- 找median需要数列保持sorted状态，如果是单数数列取中间，偶数数列中间两个值

如果能保持max heap和min heap的大小相差最大为一，那么就完全符合以上条件。
单数：max heap or min heap pop
偶数：(mean heap pop + min heap pop) / 2

主要设计：

```java

// length always n/2
private PriorityQueue<Integer> minHeap = new PriorityQueue<Integer>((a, b)-> a - b);
// length always n/2 or n/2 + 1
private PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>((a, b)-> b - a);

public void addNum(int num) {
    // for the very first time, insert into min heap first, no rebalance
    if (minHeap.isEmpty()) {
        minHeap.offer(num);
        return;
    } else if (minHeap.peek() <= num) {
        minHeap.offer(num);
    } else {
        maxHeap.offer(num);
    }
    rebalance();
}

public void remove(int n) {
    // min heap, min is at top, n is in min heap
    if (minHeap.peek() <= n) {
        // means max part needs to cut of one
        minHeap.remove(n);
    } else {
        maxHeap.remove(n);
    }
    rebalance();
}

public void rebalance() {
    if (minHeap.size() > maxHeap.size()) {
        maxHeap.offer(minHeap.poll());
    } else if (maxHeap.size() > minHeap.size()) {
        minHeap.offer(maxHeap.poll());
    }
}

```

- min heap 起手push
- 第二个push
    - 小于第一个值，进max heap, 不需rebalance.
    - 大于第一个值，进min heap, rebalance以后进max heap.
- 第三个push
    - 小于min heap顶，进max heap, rebalance后进min heap.
    - 大于min heap顶，进min heap, rebalance以后进max heap.
- 第nnnnnn个以后

如此就保持了两个heap的平衡, 最后，选median就很简单了。

代码
```java

private PriorityQueue<Integer> minHeap = new PriorityQueue<Integer>((a, b)-> a - b);
private PriorityQueue<Integer> maxHeap = new PriorityQueue<Integer>((a, b)-> b - a);

public double[] medianSlidingWindow(int[] nums, int k) {
    int n = nums.length;
    if (n <= 0) return new double[0];
    double[] result = new double[n - k + 1];

    for (int i = 0; i < nums.length; i++){
        addNum(nums[i]);
        if (i - k >= 0) {
            remove(nums[i - k]);
        }

        // deal with the extra position
        if (i - k + 1 >= 0) {
            result[i - k + 1] = findMedian();
        }
    }

    return result;
}

public void addNum(int num) {
    if (minHeap.isEmpty()) {
        minHeap.offer(num);
        return;
    } else if (minHeap.peek() <= num) {
        minHeap.offer(num);
    } else {
        maxHeap.offer(num);
    }
    rebalance();
}

public double findMedian() {
    if (minHeap.size() == maxHeap.size())
        return (minHeap.peek()/ 2.0 + maxHeap.peek()/ 2.0) ;
    else
        return minHeap.peek();
}

public void remove(int n) {
    // min heap, min is at top, n is in min heap
    if (minHeap.peek() <= n) {
        // means max part needs to cut of one
        minHeap.remove(n);
    } else {
        maxHeap.remove(n);
    }
    rebalance();
}

public void rebalance() {
    if (minHeap.size() > maxHeap.size()) {
        maxHeap.offer(minHeap.poll());
    } else if (maxHeap.size() > minHeap.size()) {
        minHeap.offer(maxHeap.poll());
    }
}
```
