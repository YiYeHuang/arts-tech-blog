---
layout: post
title:  "Fenwick Tree"
date:   2019-09-27 15:59:53 -0700
categories: algorithm
tag: [leetcode, binary-index-tree,hard]
---

## Algorithm
一整个月没有写博客，~~绝对不是因为懒~~, 主要是因为在憋Binary Search的大招，无奈敌人实在太强大，这里预留一个传送门位置，等题做完了写。Binary Search做着做着就心灰意冷，然后做着做着，LeetCode题就开始做歪了。
不过说到歪的树，这篇就总结一下Fenwick Tree的题，也就是Binary Index Tree


为Binary Index Tree量身定做的题是`LeetCode 307. Range Sum Query - Mutable`。不过在分析这题之前先看:

```text
 303. Range Sum Query - Immutable

 Given an integer array nums, find the sum of the elements between indices i and j (i ≤ j), inclusive.

 Example:
 Given nums = [-2, 0, 3, -5, 2, -1]

 sumRange(0, 2) -> 1
 sumRange(2, 5) -> -1
 sumRange(0, 5) -> -3
 Note:
 You may assume that the array does not change.
 There are many calls to sumRange function.
 ```

个人推荐，做题不需要怵暴力解题，会暴力解了能知道那里慢，需要怎么优化，所谓暴力一直爽，一直暴力一直爽。单次call sum很简单，移动到low index开始，一路加到high index. 难点坐落于这个sum需要call很多次，单次的worst case都能到O(n)。

大佬这个时候已经坐不住了：一看到需要用所有解的问题就是DP啊。。。嗯。。。。没错
先来证明，

e.g. [2, 5]

需要 sum 3, -5, 2, -1, 这个时候需要引入 prefixSum的概念

`sum[0, 5] ==> -2, 0, 3, -5, 2, -1`  
`sum[0, 2] ==> -2, 0, 3`  
`sum[2, 5] == sum[0, 5] - sum[0, 1]`

得出，prefixSum[i, j] where i<=j is sum[j] - sum[i - 1]

解题：
```java
public class RangeSumQueryImmutable {

    int[] result;
    public RangeSumQueryImmutable(int[] nums) {

        for(int i = 1; i < nums.length; i++) {
            nums[i] += nums[i - 1];
        }

        result = nums;
    }

    public int sumRange(int i, int j) {
        if(i == 0) {
            return result[j];
        }

        return result[j] - result[i - 1];
    }
}
```

看307正题
 ```test
 307. Range Sum Query - Mutable

Given an integer array nums, find the sum of the elements between indices i and j (i ≤ j), inclusive.

The update(i, val) function modifies nums by updating the element at index i to val.

Example:

Given nums = [1, 3, 5]

sumRange(0, 2) -> 9
update(1, 2)
sumRange(0, 2) -> 8
Note:

The array is only modifiable by the update function.
You may assume the number of calls to update and sumRange function is distributed evenly.
 ```

比较经典的LeetCode作妖方式，不动的input变为可动的input。
不改数据结构是绝对怎么做都会超时的。 这题有两种数据结构可以做: Binary Index Tree 和 Segement Tree.

这里讲Binary Index Tree, 也就是 Fenwick Tree。
可变的数据的痛点在于initial result construct比较耗时，有n次call会让效率变成 O(n^2)。

Fenwick tree其实不是tree，有一些想heapifer的算法，用array表示树的结构，而且这颗树歪的很。核心思想是delay update和delay query. 

[原理演示](https://visualgo.net/en/fenwicktree)

主要原理是利用了一个`lowBit` function
```text
Lowbit(x)=x&-x;
```
含义是，每次index递增它最后出现的一位1所在位置的值

Update Sum:

index 1 -> 01, 下一位更新位置 > 01 + 01 = 10, 再下一位 10 + 10 = 100, 跳过了index 3,  
index 3 -> 101, 下一位更新位置 > 101 + 01 = 110, 再下一位 110 + 10 = 1000

Query:

Query Sum的时候则是在target index反向减去lowBit, 然后顺路加上之前没有累加的Sum。

这样使得update和Query的performance都达到了O(logN)


Implementation:

Java的Implementation可以利用Integer自带的lowestOneBit实现`Lowbit(x)=x&-x;`
```java
public class BinaryIndexTree {
    private int[] sums;
    private int size;

    public BinaryIndexTree(int n) {
        size = n + 1;
        sums = new int[size];
    }

    public void update(int i, int val) {
        // skip the initial 0
        ++i;
        while (i < size) {
            counts[i] += val;
            i += Integer.lowestOneBit(i);
        }
    }

    public int sum(int i) {
        // skip the initial 0
        ++i;
        int ans = 0;
        while (i > 0) {
            ans += counts[i];
            i -= Integer.lowestOneBit(i);
        }
        return ans;
    }

    public int query(int i, int j) {
        return sum(j) - sum(i - 1);
    }
}
```

307. 解题也就引刃而解

引用上面的Class
```java
public class NumArray {

    BinaryIndexTree bitTree;

    public NumArray(int[] nums) {
        bitTree = new BinaryIndexTree(nums.length);

        for(int i = 0; i < size; i++){
            bitTree.updateTree(i, nums[i]);
        }
    }

    public void update(int i, int val) {
        bitTree.updateTree(i, val - nums[i]);
    }

    public int sumRange(int i, int j) {
        return bitTree.sumRange(i, j);
    }
}
```

做完了这题，难免心潮澎湃，以为得到了刷题初期的dfs模版，感觉又可以连干10题，没想到一点开Binary Index Tree的tag就特么5题，全是Hard，还一题比一题难，(╯°□°)╯︵ ┻━┻

继续跪着做题。。。┬─┬ノ( º _ ºノ)

```text
327. Count of Range Sum

Given an integer array nums, return the number of range sums that lie in [lower, upper] inclusive.
Range sum S(i, j) is defined as the sum of the elements in nums between indices i and j (i ≤ j), inclusive.

Note:
A naive algorithm of O(n2) is trivial. You MUST do better than that.

Example:

Input: nums = [-2,5,-1], lower = -2, upper = 2,
Output: 3 
Explanation: The three ranges are : [0,0], [2,2], [0,2] and their respective sums are: -2, -1, 2.
```

Hard题一定是变形题，绝对不直接用概念。

先暴力解：
```java
public class Solution {
    public int countRangeSum(int[] nums, int lower, int upper) {
        if(nums.length == 0) return 0;
        long[] sum = new long[nums.length + 1];
        int sum = 0;
        for (int i = 0; i < nums.length - 1; i++) {
            sum[i + 1] = sum + nums[i];
            sum = sum[i + 1];
        }
 
        int answer = 0;
        for (int i = 0; i < nums.length; i++) {
            for (int j = i + 1; j <= nums.length; j++) {
                if(lower  <= sum[j] - sum[i] && sum[j] - sum[i] <= upper) {
                    ans++;
                }
            }
        }
        return answer;
    }
}
```
简单易懂，妥妥的O(n^2)。总结暴力解:

需要得到所有符合个数  

`lower bound < prefixSum(i, j) < upper bound`

-> `lower < sum[j] - sum[i - 1] < upper`  
-> `lower + sum[i] < sum[j] < upper bound + sum[i]`  
or -> `sum[j] - lower < sum[i - 1] < sum[j] - upper` where i<=j  

可以翻译成

求`sum[i]` 之内， 符合 `[sum[i-1] - lower, sum[i-1] - upper]` 区间内条件的所有解。
即

区间和[0, i]，[1, i]... [i - 1, i]在范围[lower, upper]中的个数

用例子枚举：[1, -1, 3, 2], 区间为 [2, 4]; 

此例子中，求 [0, 3], [1, 3] [2, 3], [3, 3] index区间内的presumfix在[low, high]中的个数。  
除去重复解, 并且累加之前的结果。因此这道题的Binary Index Tree并不是用来记prefixSum的，而是用来记结果的。

(；¬д¬) ？？？？？ 嗯，所以这题只是可以用BIT解，但并不是最优处理办法，divide and conquer的办法代码更简便，但极度烧脑。

Implementation
- 需要一个数据结构记录所有的sum, sum - low, sum - high,
- 因为需要在这个数据结构里搜索，可以选择
    - sort + binary search
    - 或者push进map做搜索
- 以下选择sort + binary search
- build BIT, 和第一个数据结构同size, 走一遍input number list, 建立prefixSum
- 一边建立建立prefixSum一边在第一个数据结构里寻找prefixSum， 然后取得index build BIT
- 第二次过input number list
- 在BIT里query Sum(i) + Upperbound的index
- 减去在BIT里query Sum(i) + lowerbound的index
- 这一系列操作的意义是因为第一个数据结构被sort过了，而且input number list是正负随机的，而low一定小于high
- 所以sum(i) + upperbound很有可能小于，Sum(i) + lowerbound， 这样就没有解

让人精分的implementation
```java
public int countRangeSum(int[] nums, int lower, int upper) {
    long[] sum = new long[nums.length + 1];
    long[] range = new long[3 * sum.length + 1];
    int index = 0;
    range[index++] = sum[0];
    range[index++] = lower + sum[0] - 1;
    range[index++] = upper + sum[0];

    for (int i = 1; i < sum.length; i++) {
        sum[i] = sum[i - 1] + nums[i - 1];
        range[index++] = sum[i];
        range[index++] = lower + sum[i] - 1;
        range[index++] = upper + sum[i];
    }

    // avoid getting root of the binary indexed tree when doing binary search
    range[index] = Long.MIN_VALUE; 
    Arrays.sort(range);

    BinaryIndexTree bitTree = new BinaryIndexTree(range.length);

    // build up the binary indexed tree
    for (int i = 0; i < sum.length; i++) {
        bitTree.update(Arrays.binarySearch(range, sum[i]), 1);
    }

    int count = 0;

    for (int i = 0; i < sum.length; i++) {
        // get rid of visited elements by adding -1 to the corresponding tree nodes
        addValue(bit, Arrays.binarySearch(range, sum[i]), -1);

        // valid elements with upper bound (upper + sum[i]) - lower bound (lower + sum[i] - 1)
        count += bitTree.query(
            Arrays.binarySearch(range, lower + sum[i] - 1), 
            Arrays.binarySearch(range, upper + sum[i]));
    }

    return count;
}
```
题里用long是为了过一些恶心的test case


跪着长了膝盖疼。。。
 ```text
 315. Count of Smaller Numbers After Self

You are given an integer array nums and you have to return a new counts array. The counts array has the property where counts[i] is the number of smaller elements to the right of nums[i].

Example:

Input: [5,2,6,1]
Output: [2,1,1,0] 
Explanation:
To the right of 5 there are 2 smaller elements (2 and 1).
To the right of 2 there is only 1 smaller element (1).
To the right of 6 there is 1 smaller element (1).
To the right of 1 there is 0 smaller element.
 ```

继续暴力
```java
class Solution {
    public List<Integer> countSmaller(int[] nums) {
        List<Integer> list = new ArrayList<>();

        for (int i = 0; i < nums.length; i++) {
            int counter = 0;
            for (int j = i + 1; j < nums.length; j++) {
                if (nums[j] < nums[i]) {
                    counter++;
                }
            }
            list.add(counter);
        }

        return list;
    }
}
```

每次暴力完了都觉得自己能站起来了，一分析又跪下了。

先分析理想状态，如果要比大小最好是能有序数列，但是又要注意比较的数字的原始index。
此外，以[5,2,6,1]的例子来看，比2小的有1个数字，如果5知道比2小的数字，那只用记录2比5小就可以，其余的答案继续加上。

得出
- 需要排序， 去重
- 需要记录原始顺序，
- 应该逆向比较
- 用BIT记录靠前的index的答案

一写代码又跪下了。

```java
class Solution {
    public List<Integer> countSmaller(int[] nums) {
        
        int[] sortedList = Arrays.copyOf(nums, nums.length);
        Arrays.sort(sorted);

        Map<Integer, Integer> ranks = new HashMap<>();
        int rank = 0;
        for (int i = 0; i < sortedList.length; i++) {
            if (i == 0 || sortedList[i] != sortedList[i - 1]) {
                ranks.put(sortedList[i], ++rank);
            }
        }

        BinaryIndexTree tree = new BinaryIndexTree(ranks.size());
        List<Integer> result = new ArrayList<Integer>();

        // build BIT while go through the list in reverse order
        for (int i = nums.length - 1; i >= 0; i++) {
            int sum = tree.query(ranks.get(nums[i]) - 1);      
            ans.add(tree.query(ranks.get(nums[i]) - 1));
            tree.update(ranks.get(nums[i]), 1);
        }

        Collections.reverse(ans);
        return ans;
    }
}
```


总结：
- BIT tree主要处理需要累加之前解的问题，DP一般能处理，但是题目可能会有performance的需求
- BIT内存实用很实惠。
- T_T 实用范围不大，而且难，能用BIT解决的题segment tree, Binary search tree或者别的办法也能解决