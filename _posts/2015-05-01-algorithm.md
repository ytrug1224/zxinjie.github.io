---
layout: post
title: 时间复杂度为n方的基础算法实现
categories: [算法]
description: 算法
keywords: 算法
---

**插入排序**
适用于近似有序的序列，时间复杂度是 O(n*n)

```
void insertSort(int arr[]){
    for (int i = 0; i < arr.length; i ++){
        int temp = arr[i];
        for(int j = i; j > 0 && arr[j-1] < temp; j--) {
            arr[j]=arr[j-1];  
        }
        arr[j] = temp;
    }
}
```
**选择排序**
时间复杂度是 O(n*n), 先确定位置，再选择最小值。
```
void selectSort(int arr[]){
    for(int i = 0; i < arr.length; i){
        int minIndex = i;
        for( int j = i + 1; j < n; j ++ )
            if( arr[j] < arr[minIndex] )
                minIndex = j;
        
        swap(arr[minIndex], arr[i]);
    }
}
```
**冒泡排序**
时间复杂度是 O(n*n)， 最大值沉底
```
void bubbleSort(int[] arr) {
    int len = arr.length;
    for (int i = 0; i < len - 1; i++) {
        for (int j = 0; j < len - i - 1; j++) {
            if (arr[j] < arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
        len--;
    }
}
```

**希尔排序**

```
 public static void shellSort(int[] arr) {
        //增量gap，并逐步缩小增量
        for (int gap = arr.length / 2; gap > 0; gap /= 2) {
            //从第gap个元素，逐个对其所在组进行直接插入排序操作
            for (int i = gap; i < arr.length; i++) {
                int j = i;
                int temp = arr[j];
                if (arr[j] < arr[j - gap]) {
                    while (j - gap >= 0 && temp < arr[j - gap]) {
                        //移动法
                        arr[j] = arr[j - gap];
                        j -= gap;
                    }
                    arr[j] = temp;
                }
            }
        }
    }
```
