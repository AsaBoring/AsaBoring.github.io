---
layout: post
title: C++实现二分查找
date: 2018-07-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - C++
---

#### 递归实现
```
int View::binarySearchRecursively(int arr[] , int low , int high , int nKey)
{
    if(low <= high)
    {
        int mid = (low + high)/2;
        if(arr[mid] == nKey)
        {
            return mid;
        }else if(arr[mid] < nKey)
        {
            return binarySearchRecursively(arr,mid+1,high,nKey);
        }else if(arr[mid] > nKey)
        {
            return binarySearchRecursively(arr,low,mid-1,nKey);
        }
    }
}

```

#### 循环实现
```
int View::binarySearch(int arr[], int low, int high, int nKey)
{
    /*临时标记值*/
    int mid = 0;

    while (low <= high) {
        mid = (low + high)/2;
        if(arr[mid] == nKey)
        {
            break;
        }else if(arr[mid] < nKey)
        {
            low = mid + 1;
        }else{
            high = mid - 1;
        }
    }

    return mid;
}
```
