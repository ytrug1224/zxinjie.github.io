---
layout: post
title: nlogn算法快速排序、归并排序
categories: [算法]
description: 快速排序、归并排序
keywords: 快速排序、归并排序
---

Content here

**归并排序**

归并排序主要选择 跳转的gap 是关键；优化思路是减少数据间的替换。

```
using namespace std;

/**
* 自顶而上的归并实现
*/
template <typename T>
void mergeSortBU(T arr[], int n){
    // Merge Sort Bottom Up 优化
    for( int i = 0 ; i < n ; i += 16 )
        insertionSort(arr,i,min(i+15,n-1));

    for( int sz = 16; sz <= n ; sz += sz )
        for( int i = 0 ; i < n - sz ; i += sz+sz )
            if( arr[i+sz-1] > arr[i+sz] )
                __merge(arr, i, i+sz-1, min(i+sz+sz-1,n-1) );
}

// 将arr[l...mid]和arr[mid+1...r]两部分进行归并
template<typename  T>
void __merge(T arr[], int l, int mid, int r){

    // 经测试,传递aux数组的性能效果并不好
    T aux[r-l+1];
    for( int i = l ; i <= r; i ++ )
        aux[i-l] = arr[i];

    int i = l, j = mid+1;
    for( int k = l ; k <= r; k ++ ){

        if( i > mid )   { arr[k] = aux[j-l]; j ++;}
        else if( j > r ){ arr[k] = aux[i-l]; i ++;}
        else if( aux[i-l] < aux[j-l] ){ arr[k] = aux[i-l]; i ++;}
        else                          { arr[k] = aux[j-l]; j ++;}
    }
}

// 递归使用归并排序,对arr[l...r]的范围进行排序
template<typename T>
void __mergeSort2(T arr[], int l, int r){

    // 对于小规模数组,使用插入排序
    if( r - l <= 15 ){
        insertionSort(arr, l, r);
        return;
    }

    int mid = (l+r)/2;
    __mergeSort2(arr, l, mid);
    __mergeSort2(arr, mid+1, r);
    // 对于arr[mid] <= arr[mid+1]的情况,不进行merge
    // 对于近乎有序的数组非常有效
    if( arr[mid] > arr[mid+1] )
        __merge(arr, l, mid, r);
}

template<typename T>
void mergeSort2(T arr[], int n){
    __mergeSort2( arr , 0 , n-1 );
}

```

**快速排序**

快速排序优化分为二路快排和三路快排；

```
protected static int partition(int arr[],int low,int high) {
        int i=low,j=high;
        int temp=arr[low];
        while(i<j) {
            while (i<j&& temp <= arr[j])
                --j;
            arr[i] = arr[j];
            while(i<j && temp >= arr[i]) 
                ++i;
            arr[j] = arr[i];

        }
        arr[i] = temp;
        return i;
    }
    //递归排序
    public static void quickSort1(int arr[],int low,int high){
        if(low < high) {
            int mid = partition(arr, low, high);
            quickSort1(arr, low, mid-1);
            quickSort1(arr, mid+1, high);
        }
    }

    //非递归实现
    public static void quickSort2(int arr[],int low,int high) {
        Stack<Integer> st = new Stack<Integer>();
        if (low < high) {
            int mid = partition(arr, low, high);
            if (mid-1 > low) {
                st.push(mid-1);
                st.push(low);
            }
            if (mid+1 < high) {
                st.push(high);
                st.push(mid+1);
            }

            while (!st.isEmpty()) {
                int q_low = st.peek();
                st.pop();
                int p_high = st.peek();
                st.pop();
                mid = partition(arr, q_low, p_high);
                if (mid-1 > q_low) {
                    st.push(mid-1);
                    st.push(q_low);
                }
                if (mid+1 < p_high) {
                    st.push(p_high);
                    st.push(mid+1);
                }
            }
        }
    }

```

三路排序
```
template <typename T>
void __quickSort3Ways(T arr[], int l, int r){

    if( r - l <= 15 ){
        insertionSort(arr,l,r);
        return;
    }

    swap( arr[l], arr[rand()%(r-l+1)+l ] );

    T v = arr[l];

    int lt = l;     // arr[l+1...lt] < v
    int gt = r + 1; // arr[gt...r] > v
    int i = l+1;    // arr[lt+1...i) == v
    while( i < gt ){
        if( arr[i] < v ){
            swap( arr[i], arr[lt+1]);
            i ++;
            lt ++;
        }
        else if( arr[i] > v ){
            swap( arr[i], arr[gt-1]);
            gt --;
        }
        else{ // arr[i] == v
            i ++;
        }
    }

    swap( arr[l] , arr[lt] );

    __quickSort3Ways(arr, l, lt-1);
    __quickSort3Ways(arr, gt, r);
}
```