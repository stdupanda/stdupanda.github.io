---
layout: post
title: 常见排序算法
categories: 算法
description: 整理常见排序算法
keywords: java, algorithm, sort, 排序
---

简单整理常见的排序算法，以供查阅。

# `bubble sort` 冒泡排序

其原理类似水里的泡泡, 故而成为 `冒泡排序`.

> 重复地走访过要排序的数列, 一次比较两个元素, 如果他们的顺序错误就把他们交换过来. 走访数列的工作是重复地进行直到没有再需要交换, 也就是说该数列已经排序完成.

算法复杂度为 O(n^2).

```java
public class BubbleSort {
    public void sort(int[] a) {
        int temp = 0;
        for (int i = a.length - 1; i > 0; --i) {
            for (int j = 0; j < i; ++j) {
                if (a[j + 1] < a[j]) {
                    temp = a[j];
                    a[j] = a[j + 1];
                    a[j + 1] = temp;
                }
            }
        }
    }
}
```

# `quick sort` 快速排序

对 `冒泡排序` 的一种改进.

> 基本思想是：通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。


# `select sort` 选择排序

> 工作原理是每一次从待排序的数据元素中选出最小（或最大）的一个元素，存放在序列的起始位置，直到全部待排序的数据元素排完。 选择排序是 **不稳定** 的排序方法（比如序列[5， 5， 3]第一次就将第一个[5]与[3]交换，导致第一个5挪动到第二个5后面）。

```java
public static void selectSort(int[] a) {
    int minIndex = 0;
    int temp = 0;
    if((a == null) || (a.length == 0))
        return;
    for(int i = 0; i < a.length - 1; i++) {
        minIndex = i;//无序区的最小数据数组下标
        for(intj = i + 1; j < a.length; j++) {
            //在无序区中找到最小数据并保存其数组下标
            if(a[j] < a[minIndex]) {
                minIndex = j;
            }
        }
        if(minIndex!=i) {
            //如果不是无序区的最小值位置不是默认的第一个数据，则交换之。
            temp = a[i];
            a[i] = a[minIndex];
            a[minIndex] = temp;
        }
    }
}
```
