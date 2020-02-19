---
layout: post
title:  "Evaluate Performance of Divide and Conquer Algorithm"
date:   2019-11-13 15:59:53 -0700
categories: algorithm
tag: [theory, divide-conquer,prof]
---

## Algorithm

For merge sort, we know that the Big O analysis for the algorithm is O(nlog(n)).  
But, how do we prove that?  

The concept of Master mathod is used here:  
let `a` be the number of recursion calls of a algorithm  
`b` be the # of divided sample when the recursion is called
`d` be the Big O performance index of operations down during recursion call.

We have follow
![masterfun](/public/img/masterFun.png)
- `T(n)` as the total number of operations

Let's look at merge sort code
```java
private static void mergeAndCount(int[] arr, int l, int m, int r) {
    // Left subarray
    int[] left = Arrays.copyOfRange(arr, l, m + 1);
    // Right subarray
    int[] right = Arrays.copyOfRange(arr, m + 1, r + 1);

    int i = 0, j = 0, k = l, swaps = 0;
    while (i < left.length && j < right.length) {
        if (left[i] <= right[j])
            arr[k++] = left[i++];
        else {
            arr[k++] = right[j++];
        }
    }

    // Fill from the rest of the left subarray
    while (i < left.length) {
        arr[k++] = left[i++];
    }

    // Fill from the rest of the right subarray
    while (j < right.length) {
        arr[k++] = right[j++];
    }
}

private static int mergeAndSplit(int[] arr, int l, int r) {
    if (l < r) {
        int m = (l + r) / 2;

        mergeAndSplit(arr, l, m);
        mergeAndSplit(arr, m + 1, r);

        mergeAndCount(arr, l, m, r);
    }
}
```

- there are two recursion calls a = 2
- for each call, array get split into 2, b = 2
- the merge operation takes O(n) so d = 1
- a = b^d
- so merge sort algorithm is O(nlog(n))