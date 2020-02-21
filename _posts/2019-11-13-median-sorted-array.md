---
layout: post
title:  "Encounter the classic: Median of two sorted array"
date:   2019-11-13 15:59:53 -0700
categories: algorithm
tag: [leetcode, binary-search]
---

## Algorithm

```text
4. Median of Two Sorted Arrays

There are two sorted arrays nums1 and nums2 of size m and n respectively.
Find the median of the two sorted arrays.  
The overall run time complexity should be O(log (m+n)).
You may assume nums1 and nums2 cannot be both empty.

Example 1:
nums1 = [1, 3]
nums2 = [2]

The median is 2.0

Example 2:
nums1 = [1, 2]
nums2 = [3, 4]

The median is (2 + 3)/2 = 2.5
```

按序号刷题的第一题难题，leetcode劝退第一show。  
暴力解题很简单，就是`21. Merge Two Sorted Lists`以后找median。然而题目非常直接，要求O(logn)的效率，赤裸裸的告诉你需要binary search来解题。直接到仿佛丈母娘要求有房有车才能接走女儿一样。

举例子分析：  
最简单的例子就是无交集.  
<pre>
          v
1 2 3 4 5 6  
            7 8 9 10 11 
</pre>
已知list 1 长度是6， list 2 长度是5，一刀砍在index 5上，list 2的头上

有交集

<pre>
              v  v
1 2 3 4 5     9              15 16
          7 8    10 11 12 13       18 19
</pre>
已知list 1 长度是8， list 2 长度是8，一刀砍在index 7和8的中间，分别在array 1的 5/6， array 2的2/3  
所以解题的关键是得知那一刀/两刀分别砍在每个array的哪里  
但是在全局看来，一定是砍在两个array merge以后的中间，所以得

`arrayCut1 + arrayCut2 = (arrayLength1 + arrayLength2 + 1) / 2`  
加一是因为单数int 除以二会有损

去掉空格看分布  
<pre>
1 2 3 4 5 9 | 15 16
        7 8 | 10 11 12 13 18 19
</pre>

看图可得 -> `array1left, array2left  < array1right, array2right`  
已知 array1left.max < array1right.min 因为array已经排序  
已知 array2left.max < array2right.min 因为array已经排序  
需要完成 left < right的条件是  
array1left.max < array2right.min  
array2left.max < array1right.min

如果条件不满足

<pre>
1 2 3 4 5 9 15 | 16
           7 8 | 10 11 12 13 18 19
</pre>
则表示在array1的分区太靠右（偏大）需要往左，疑。。。好像binary search的思路就这么出来了

因为最后计算的时候需要保证 leftPart.Max < righPart.Min
<pre>
1 2 3 4 5 9 | 15 16
        7 8 | 10 11 12 13 18 19
</pre>
所以如果 array1.length + array2.length 是双数时
```
median = 
(max(array1left.rightMost, array2left.rightMost) + min(array1right.leftMost, array2right.leftMost))/2
```
如果是单数
```
median = max(array1left.rightMost, array2left.rightMost)
```

如果到极端的情况下  
<pre>
1 2 3 4 5 6 | MAX
        MIN | 7 8 9 10 11 
</pre>
为了比较，set array2right.min = Integer MAX, array1right.max = Integer MIN

如果到更极端的情况下  
<pre>
    MIN | MAX
    MIN | 7 8 9 10 11 
</pre>
partition A就直接为0， partition B 就直接作为单个数列取median，index为(0 + array B length)/2 

代码:
为了效率，binary search的时候选择短的array做search
```java
public static double findMedianSortedArrays(int[] nums1, int[] nums2) {
    if (nums1.length > nums2.length) {
      return findMedianSortedArrays(nums2, nums1);
    }

    int aLength = nums1.length;
    int bLength = nums2.length;

    int low = 0;
    int high = aLength;

    // we want to search on the shorter list
    while (low <= high) {
      int partitionA = (low + high)/2;
      int partitionB = (aLength + bLength + 1) /2 - partitionA; // plus one for odd number

      int maxLeftA = partitionA == 0 ? Integer.MIN_VALUE : nums1[partitionA - 1];
      int minRightA = partitionA == aLength ? Integer.MAX_VALUE : nums1[partitionA];

      int maxLeftB = partitionB == 0 ? Integer.MIN_VALUE : nums2[partitionB - 1];
      int minRightB = partitionB == bLength ? Integer.MAX_VALUE : nums2[partitionB];

      if (maxLeftA <= minRightB && maxLeftB <= minRightA) {
        // get the point the the left side of the cut are all smaller than the right side of the cut
        if ((aLength + bLength) % 2 == 0) {
          return ((double)Math.max(maxLeftA, maxLeftB) + Math.min(minRightA, minRightB))/2;
        } else {
          return (double)Math.max(maxLeftA, maxLeftB);
        }
      } else if (maxLeftA > minRightB) {
        // means for array we are searching in, the value is too big
        high = partitionA - 1;
      } else {
        low = partitionA + 1;
      }
    }

    return Integer.MIN_VALUE;
}
```