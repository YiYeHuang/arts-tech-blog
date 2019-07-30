---
layout: post
title:  "Basic Calculator"
date:   2019-07-23 15:59:53 -0700
categories: algorithm
tag: [leetcode, stack, median]
---

## Algorithm

### 224. Basic Calculator

```text
  227. Basic Calculator II
  Medium

  Share
  Implement a basic calculator to evaluate a simple expression string.

  The expression string contains only non-negative integers, +, -, , / operators and empty spaces . The integer division should truncate toward zero.

  Example 1:

  Input: "3+22"
  Output: 7
  Example 2:

  Input: " 3/2 "
  Output: 1
  Example 3:

  Input: " 3+5 / 2 "
  Output: 5

```

224 Basic Calculator 的延伸， 这题标榜median, 不过在我看来这两题都是hard不足，median有余。
如果事前做了224的话这题应该不难。224先入为主，依旧用stack来解题，了解清楚几个出栈的条件这题就没有什么问题了。

#### 要点:
- input永远是valid的，这点很重要(leetcode的仁慈？？)
- 乘法除法需要优先计算
- 需要处理个位数以上的运算
- 使用stack的问题

输入实例 3*4+4/2-9/3 这个其实可以看作

3*4一组，4/2一组，9/3一组。 这题因为没有括号，正序反序遍历都一样，所以解题要点是
- 第一个数字不重要，第二个数字出现的时候才开始计算
- 需要cache之前的运算符号
- +号和-号和224的括号一样，是入栈条件
- 为了让第一个数字入栈，initial operator为+

#### 代码：
需要处理以下
- + 数字入栈 cache清零
- - 数字加'-'入栈 cache清零
- 数字: 处理进制的问题
- ' ': 继续前进
- *: pop和现在cache的数字计算，然后push入栈
- /: pop和现在cache的数字计算，然后push入栈


```java
    public static int calculate(String s) {
        if (s == null || s.length() == 0) return 0;
        Stack<Integer> stack = new Stack<>();
        s += '+';
        char operator = '+';
        for (int i = 0, n = 0; i < s.length(); i++) {
            char ch = s.charAt(i);

            if (ch >= '0' && ch <= '9') {
                n = n * 10 + ch - '0';
                continue;
            }
            if (ch == ' ') continue;
            if (operator == '+') {
                stack.push(n);
            } else if (operator == '-') {
                stack.push(-n);
            } else if (operator == '*') {
                stack.push(stack.pop()*n);
            } else if (operator == '/') {
                stack.push(stack.pop()/n);
            }
            operator = ch;
            n = 0;
        }

        int total = 0;
        while (!stack.isEmpty()) {
            total += stack.pop();
        }
        return total;
    }
```