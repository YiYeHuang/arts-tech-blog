---
layout: post
title:  "All the sums"
date:   2019-09-01 15:59:53 -0700
categories: algorithm
tag: [leetcode, two-pointer, hashtable]
---

## Algorithm

作为题号为1的题，two sum一直是LeetCode看门人，但凡刷过题的人一般都是秒做，没刷过的秒死。
但是要掌握所有的Sum题，并不是这么容易的。

### 1. Two Sum
```text
 Given an array of integers, 
 return indices of the two numbers such that they add up to a specific target.

 You may assume that each input would have exactly one solution, 
 and you may not use the same element twice.

 Example:

 Given nums = [2, 7, 11, 15], target = 9,

 Because nums[0] + nums[1] = 2 + 7 = 9,
 return [0, 1].
```

Two Sum本体：
- 只有一个正确答案
- 不能使用重复数字
- return index

解法很简单， 结束一个hashmap存储对应的值和坐标，时间复杂度 O(n), 空间复杂度O(n)。
HashMap的速度最优为O(1)，解决了暴力解法的问题。

```java
public static int[] twoSum(int[] numbers, int target){
    int[] result = new int[2];
    // <value, index>
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < numbers.length; i++)
    {
        if (map.containsKey(target - numbers[i]))
        {
            result[1] = i + 1;
            result[0] = map.get(target - numbers[i]);
            return result;
        }
        map.put(numbers[i], i + 1);
    }
    return result;
}
```
### 167. Two Sum II - Input array is sorted
```text
Given an array of integers that is already sorted in ascending order, find two numbers such that they add up to a specific target number.

The function twoSum should return indices of the two numbers such that they add up to the target, where index1 must be less than index2.

Note:

Your returned answers (both index1 and index2) are not zero-based.
You may assume that each input would have exactly one solution and you may not use the same element twice.
Example:

Input: numbers = [2,7,11,15], target = 9
Output: [1,2]
Explanation: The sum of 2 and 7 is 9. Therefore index1 = 1, index2 = 2.
```

Two sum的一个小小变形题，也是重点题，从这道题的技巧是最后three sum和four sum的基础。

变形在input的数列是已经sorted, leetcode用hashmap依旧能过这道题，但是本意是要使用二分法的一些思想。

因为数列已经排序，所以可以用两个指针往中间靠拢的办法找解。
- 相加的答案大于target, 尾数过大
- 相加的答案小于target, 头部数过小

```java
public static int[] twoSum(int[] numbers, int target) {
    int low = 0;
    int high = numbers.length - 1;
    int[] result = new int[2];

    while (low < high) {

        if (numbers[low] + numbers[high] < target) {
            low++;
        } else if (numbers[low] + numbers[high] > target) {
            high--;
        } else {
            return new int[]{low + 1, high + 1};
        }
    }

    return result;
}
```

### 15. 3Sum
```text

Given an array nums of n integers, are there elements a, b, c in nums 
such that a + b + c = 0? 
Find all unique triplets in the array which gives the sum of zero.

Note:

The solution set must not contain duplicate triplets.

Example:

Given array nums = [-1, 0, 1, 2, -1, -4],

A solution set is:
  [
    [-1, 0, 1],
    [-1, -1, 2]
  ]
```
进入正题，3sum 做了简化，不需要求得target K, 只要相加为0就好。
并且要求得出所有解。

解读
- 数列非排序
- 有负数
- 要求所有解

无排序外加所有解这一点使得hashmap在这里没有这么好用，

有了two sum sorted的思维，可以把这题的某一个解解读为: 
```三个数字第一个数字为0， 求剩余数列中相加为0的两个数字。```
这么就把3 sum的题化解为2 sum sorted的解法。所以算法的第一步就是把整个数列sort一次。

剩下的解法就变成：
- 锁定头
- 剩余的list用2 sum sorted办法解决
- 需要解决答案唯一性的问题

```java
public static List<List<Integer>> threeSum(int[] nums) {
    Arrays.sort(nums);
    List<List<Integer>> res = new ArrayList<>();
    for (int i = 0; i < nums.length-2; i++) {
        if (i == 0 ||  nums[i] != nums[i-1]) {
            int lo = i+1, hi = nums.length-1, sum = 0 - nums[i];
            while (lo < hi) {
                // match case
                if (nums[lo] + nums[hi] == sum) {
                    res.add(Arrays.asList(nums[i], nums[lo], nums[hi]));

                    //duplicate check
                    while (lo < hi && nums[lo] == nums[lo+1]){
                        lo++;
                    }
                    //duplicate check
                    while (lo < hi && nums[hi] == nums[hi-1]) {
                        hi--;
                    }
                    lo++;
                    hi--;
                } else if (nums[lo] + nums[hi] < sum) {
                    // need a bigger number
                    lo++;
                } else {
                    // need a smaller number
                    hi--;
                }
            }
        }
    }
    return res;
}
```

### 18. 4Sum
```text
18. 4Sum

Given an array nums of n integers and an integer target, are there elements a, b, c,
and d in nums such that a + b + c + d = target? Find all unique quadruplets in the array which gives the sum of target.

Note:

The solution set must not contain duplicate quadruplets.

Example:

Given array nums = [1, 0, -1, 0, -2, 2], and target = 0.

A solution set is:
[
  [-1,  0, 0, 1],
  [-2, -1, 1, 2],
  [-2,  0, 0, 2]
]
```
由3 sum的思路拓展基础上，4sum的一个解就是
```四个数字第一个数字为0， 求剩余数列中相加为0的三个数字。如果第二个数字也为0的时候，求剩余数列中相加为0的两个数字```
由此可以退出n sum的公式，便是不停将算法向2 sum靠拢

核心
```java 
public void twoSum(
    int[] nums, int target,
    int low, int high,
    List<List<Integer>> sumList,
    int res1, int res2) {

    if (low >= high) return;

    // end early
    if (2 * nums[low] > target || 2 * nums[high] < target) return;

    int i = low;
    int j = high;
    int sum;
    int checkDup;

    while( i < j) {
        sum = nums[i] + nums[j];
        if (sum == target) {
            sumList.add(Arrays.asList(res1, res2, nums[i], nums[j]));

            // cut duplicate
            checkDup = nums[i];
            while (i < j && checkDup == nums[i]) {
                i++;
            }
            checkDup = nums[j];
            while (i < j && checkDup == nums[j]) {
                j--;
            }
        }

        if (sum < target) i++;
        if (sum > target) j--;
    }
}
```

优化思路：
n sum的算法并不快，最坏可达到O(n^3), 并且要求全答案，所以很多数据结构也用不上。但是可以加一些优化条件，过滤掉一些答案。

假设题是K sum, 数列sorting完毕以后，
- 如果目前第n步的累计的答案 result + (k - n)* currentMin > target, 此时数列之后都不会有解了
- 如果目前第n步的累计的答案 result + (k - n)* currentMax <> target, 此时数列之后都不会有解了


```java
public List<List<Integer>> fourSum(int[] nums, int target) {
    List<List<Integer>> res = new ArrayList<>();

    int len = nums.length;
    Arrays.sort(nums);
    int max = nums[len - 1];
    if (4 * nums[0] > target || 4 * max < target) return res;

    for (int i = 0; i < len; i++) {
        int piece1 = nums[i];
        // avoid duplicate
        if (i > 0 && piece1 == nums[i - 1]) continue;
        // current piece too small
        if (piece1 + 3* max < target) continue;
        // current piece too large
        if (4 * piece1 > target) break;

        threeSum(nums, target - piece1, i + 1, len -1 , res, piece1);
    }

    return res;
}

public void threeSum(
    int[] nums, int target, 
    int low, int high, 
    List<List<Integer>> sumList, int res1) {

    if (low + 1 >= high) return;

    int max = nums[high];

    if (3 * nums[low] > target || 3 * max < target) return;

    for (int i = low; i < high - 1; i++) {
        int piece2 = nums[i];

        // avoid duplicate
        if (i > low && piece2 == nums[i - 1]) continue;
        // current piece too small
        if (piece2 + 2* max < target) continue;
        // current piece too large
        if (3 * piece2 > target) break;

        if (3 * piece2 == target) {
            // z is the boundary
            if (i + 1 < high && nums[i + 2] == piece2)
                sumList.add(Arrays.asList(res1, piece2,piece2,piece2));
            break;
        }

        twoSum(nums, target - piece2, i+1, high, sumList, res1, piece2);
    }
}


public void twoSum(
    int[] nums, int target, 
    int low, int high, 
    List<List<Integer>> sumList, int res1, int res2) {

    if (low >= high) return;

    // end early
    if (2 * nums[low] > target || 2 * nums[high] < target) return;

    int i = low;
    int j = high;
    int sum;
    int checkDup;

    while( i < j) {
        sum = nums[i] + nums[j];
        if (sum == target) {
            sumList.add(Arrays.asList(res1, res2, nums[i], nums[j]));

            // cut duplicate
            checkDup = nums[i];
            while (i < j && checkDup == nums[i]) {
                i++;
            }
            checkDup = nums[j];
            while (i < j && checkDup == nums[j]) {
                j--;
            }
        }

        if (sum < target) i++;
        if (sum > target) j--;
    }
}
```

### 一些变形
```text
16. 3Sum Closest

Given an array nums of n integers and an integer target, find three integers in nums such that the sum is closest to target. Return the sum of the three integers. You may assume that each input would have exactly one solution.

Example:

Given array nums = [-1, 2, 1, -4], and target = 1.

The sum that is closest to the target is 2. (-1 + 2 + 1 = 2).
```

3 sum的一道变形题，思路和3 sum一样，锁定头，然后用two pointer找值，区别是需要一个local的result不停和新的sum作比较，取Math.min做更新。

```java
public int threeSumClosest(int[] num, int target) {
    int result = num[0] + num[1] + num[num.length - 1];
    Arrays.sort(num);
    for (int i = 0; i < num.length - 2; i++) {
        int start = i + 1, end = num.length - 1;
        while (start < end) {
            int sum = num[i] + num[start] + num[end];
            if (sum > target) {
                end--;
            } else {
                start++;
            }
            if (Math.abs(sum - target) < Math.abs(result - target)) {
                result = sum;
            }
        }
    }
    return result;
}
```