---
layout: post
title:  "Basic Calculator"
date:   2019-07-16 15:59:53 -0700
categories: algorithm
tag: [leetcode, stack, hard]
---

## Algorithm

### 224. Basic Calculator

```text
  Implement a basic calculator to evaluate a simple expression string.

  The expression string may contain open ( and closing parentheses ), 
  the plus + or minus sign -, non-negative integers and empty spaces .

  Example 1:

  Input: "1 + 1"
  Output: 2
  Example 2:

  Input: " 2-1 + 2 "
  Output: 3
  Example 3:

  Input: "(1+(4+5+2)-3)+(6+8)"
  Output: 23
  Note:
  You may assume that the given expression is always valid.
  Do not use the eval built-in library function.
```

比较经典的一道LeetCode的题。略有刷题经验的leetcoder一看题就知道使用需要用stack来解题，但是这题标榜为hard，真正代码写起来，很容易让自己钻进去。

#### 要点:
- input永远是valid的，这点很重要
- 需要处理括号的优先级运算
- 需要处理个位数以上的运算
- 使用stack的问题

首先按常规的人类数学思想来看，是把符号从第一位开始push进stack，然后做计算。但是看以下例子

计算：`(7-(8+9))` ----> 进stack以后再pop会成为

(9 + 8 ) - 7) ----> 结果变成 17 - 7 = 10， 与正确答案8相差很远。但这个问题解决起来很容易，把string从尾到头push就可以了。

#### 代码：
需要处理以下
- '(': push即可
- ')': 其实不需要处理，闭括弧代表一组计算的结束，以此触发stack的计算。把结果继续push进stack
- 数字*: 数字比较难处理, 需要一个local variable处理进制的问题
- ' ': 无视，继续前进。


```java
计算栈内内容
    public static int calculateStack(Stack<Object> stack) {

        int res = 0;

        if (!stack.empty()) {
            res = (int) stack.pop();
        }

        // Evaluate the expression till we get corresponding ')'
        while (!stack.empty() && !((char) stack.peek() == ')')) {

            char sign = (char) stack.pop();

            if (sign == '+') {
                res += (int) stack.pop();
            } else {
                res -= (int) stack.pop();
            }
        }
        return res;
    }
```

```java
主体：按照逆向要点分析推论，'('的操作变为')' '('的操作变为')'
    public static int calculate(String s) {

        int operand = 0;
        int n = 0;
        Stack<Object> stack = new Stack<Object>();

        for (int i = s.length() - 1; i >= 0; i--) {

            char ch = s.charAt(i);

            if (Character.isDigit(ch)) {

                // Forming the operand - in reverse order.
                operand = (int) Math.pow(10, n) * (int) (ch - '0') + operand;
                n += 1;

            } else if (ch != ' ') {
                if (n != 0) {

                    // Save the operand on the stack
                    // As we encounter some non-digit.
                    stack.push(operand);
                    n = 0;
                    operand = 0;

                }
                if (ch == '(') {

                    int res = calculateStack(stack);
                    stack.pop();

                    // Append the evaluated result to the stack.
                    // This result could be of a sub-expression within the parenthesis.
                    stack.push(res);

                } else {
                    // For other non-digits just push onto the stack.
                    stack.push(ch);
                }
            }
        }

        //Push the last operand to stack, if any.
        if (n != 0) {
            stack.push(operand);
        }

        // Evaluate any left overs in the stack.
        return calculateStack(stack);
    }
```

#### 变形:
有时候，题会有变形，将正序运算变成prefix operator. e.g: (-7(+8 9))
这个时候需要略做改动，要点还是一样，抓住input永远是valid的这一点，不用做过多的case考虑，

将之前的
pop-> result1, pop->sign, pop->result2 变成 pop->sign, pop-> result1, pop->result2

``` java
    public static int calculateStackPre(Stack<Object> stack) {

        int res1 = 0;
        int res2 = 0;

        if (stack.size() == 1) {
            return (int) stack.pop();
        }
        if (!stack.empty()) {

            char sign;
            sign = (char) stack.pop();
            res1 = (int) stack.pop();
            res2 = (int) stack.pop();

            if (sign == '+') {
                res1 += res2;
            } else {
                res1 -= res2;
            }
        }

        return res1;
    }
```
又因为现在的输入主体的数字之间有空格，需要额外处理在看到' '的时候主动push进数字，并且位数计算清零.