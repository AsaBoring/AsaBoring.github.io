---
layout: post
title: 将Byte转化为合适的Kb/MB/GB等其他单位(C++)
date: 2019-02-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - C++
---

####    在开发项目的过程中需要有一个将字节数目转化为相对应的合适的`MB`或者`GB`的需求，在网上看到的许多实现实际上都有字节大小的限制，超出一定范围就会输出乱码，这实际上是由于`long`以及`double`的存储长度的限制，于是自己使用C++做了一个实现，在此记录
### 代码如下
```
#include <list>
#include <math.h>
#include <string>
#include <vector>
#include <iostream>

#define uint64_asa unsigned __int64

using namespace std;

int main()
{
    uint64_asa bytesEg = 2631111221212111;

    vector<string> vec;
    vec.push_back("B");
    vec.push_back("KB");
    vec.push_back("MB");
    vec.push_back("GB");
    vec.push_back("TB");
    vec.push_back("PB");

    uint64_asa nRemainder = 0;
    double nKeyValue  = 1024.0;
    uint64_asa nByteSize = bytesEg;
    int i = 0;

    while (nByteSize >= nKeyValue && i < vec.size())
    {
        nRemainder = (uint64_asa)nByteSize % (uint64_asa)nKeyValue;
        nByteSize = nByteSize/nKeyValue;   //nByteSize /= nKeyValue
        i++;
    }

    nByteSize = floor(nByteSize);
    string strResult = to_string((uint64_asa)nByteSize);
    if(nRemainder > 0)
    {
       strResult += "." + to_string(nRemainder).substr(0,2);
    }

    strResult += vec.at(i);
    cout << "strResult is: " << strResult<<endl;

    return 0;
}
```

#### 经过验证，确实可以实现极大字节数的正常转换
