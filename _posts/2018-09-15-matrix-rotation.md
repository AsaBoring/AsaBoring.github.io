---
layout: post
title: C++实现二维数组(二维矩阵)旋转(多角度)
date: 2018-09-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - C++
---

代码如下:
```
    int xy = 3;    //二维数组元素个数，此处设定xy轴元素个数一致，即一级数组与二级数组元素个数一致
    int n = xy-1;  //数组索引极限值，用之进行索引运算操作
    int src[xy][xy],dst[xy][xy],dst1[xy][xy],dst2[xy][xy];

    uniform_int_distribution<unsigned> u(0,20);  //随机数分布类型
    default_random_engine e;                     //随机数引擎

    /*为source数组充实数据*/
    for(int i = 0 ; i < xy ; ++i)
    {
        for(int j = 0 ; j < xy ; ++j)
        {
            src[i][j] = u(e);
            cout<<src[i][j]<<" ";
        }
        cout<<endl;
    }

    cout<<endl;

    /*转180*/
    for(int i = 0 ; i < xy ; ++i)
    {
        for(int j = 0 ; j < xy ; ++j)
        {
            dst[i][j] = src[n-i][n-j];
            cout<<dst[i][j]<<" ";
        }
        cout<<endl;
    }
    cout<<endl;

    /*顺时针转90*/
    for(int i = 0 ; i < xy ; ++i)
    {
        for(int j = 0 ; j < xy ; ++j)
        {
            dst1[i][j] = src[n-j][i];
            cout<<dst1[i][j]<<" ";
        }
        cout<<endl;
    }
    cout<<endl;

    /*逆时针转90*/
    for(int i = 0 ; i < xy ; ++i)
    {
        for(int j = 0 ; j < xy ; ++j)
        {
            dst2[i][j] = src[j][n-i];
            cout<<dst2[i][j]<<" ";
        }
        cout<<endl;
    }
```