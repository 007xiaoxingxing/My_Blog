---
title: 常见排序算法之冒泡排序
categories:
-  Program
tags:
- 排序
- 冒泡排序
- 算法
date: 2017-05-17 23:54:44
---

冒泡排序，这个就不多说了吧。show me the code!

```python
# -*- coding:utf-8 -*-

A = [3,7,5,2,9,0,1]

def bubble_sort(Array_A):
    for i in range(0,len(Array_A)-1):
        for j in range(len(Array_A)-i-1):
            if Array_A[j] > Array_A[j+1]:
                Array_A[j],Array_A[j+1] = Array_A[j+1],Array_A[j]
bubble_sort(A)
print A
```

<!-- more -->

### 冒泡排序的原理

每一趟只能将一个数归位, 如果有n个数进行排序,只需将n-1个数归位, 也就是说要进行n-1趟操作(已经归位的数不用再比较)

### 冒泡排序的缺点

效率特别低，时间复杂度O(n2)

### 冒泡排序的算法稳定性

冒泡排序就是把小的元素往前调或者把大的元素往后调。比较是相邻的两个元素比较，交换也发生在这两个元素之间。所以，如果两个元素相等，我想你是不会再无聊地把他们俩交换一下的；如果两个相等的元素没有相邻，那么即使通过前面的两两交换把两个相邻起来，这时候也不会交换，所以相同元素的前后顺序并没有改变，所以冒泡排序是一种稳定排序算法。

