---
layout: post
title: Qt+OpenCv实现车牌定位(Windows10)
date: 2019-03-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - Qt
    - OpenCv
---

####     最近这段时间需要进行图片方面的处理，众所周知，`OpenCv`是开源圈里几乎最优秀的计算机视觉处理库，本文记录在研究过程中所做的一个车牌定位的Demo。
####     本文所提供的Demo仅限于车牌定位，定位出的车牌进行识别的话可以使用`TeseRact`进行，本文不做更多描述。

### 工具准备
#### Qt
* Qt的安装网上有大量的教程，或许之后本人会出一个相关的文章，不过在这里就只放一个下载的Link好了
http://download.qt.io/archive/qt/

####  OpenCv库文件
*  `OpenCv`在Windows下配合Qt的使用需要对源码进行重新编译，我所使用的是最新的4.3版本,放个下载连接以及安装编译教程
*  [`OpenCv`下载](https://www.opencv.org/releases.html)
*  [`OpenCv`安装视频教程windows](https://www.bilibili.com/video/BV1Za4y1v7ra/?spm_id_from=333.788.videocard.0)
* 如果各位觉得上述操作太麻烦我这里提供一个我自己编译出来的[OpenCv4.3](https://github.com/AsaBoring/OpenCV_4.3_release_lib/blob/master/OpenCV4.3_release_windows_without_contrib.zip
)的lib

### Demo
1.新建一个Qt的项目
2.Pro文件中需添加`OpenCv`库的`头文件路径`以及`lib路径`[^note1]

```
INCLUDEPATH += C:\Users\houxia2x\Desktop\Asa\opencv\opencv\asa_build\install\include\opencv2\
               C:\Users\houxia2x\Desktop\Asa\opencv\opencv\asa_build\install\include

LIBS += -LC:\Users\houxia2x\Desktop\Asa\opencv\opencv\asa_build\install\x64\mingw\bin\
        -llibopencv_world430
```

3.头文件需包含OpenCv的统筹文件"opencv2/opencv.hpp"
```
#include "opencv2/opencv.hpp"
```
4.加载命名空间
```
using namespace cv;
```
5.加载图片
```
Mat srcImage = imread(path);
if(srcImage.empty())
{
    cout<<"load pic failed";
    return;
}
```
6.对图片进行高斯模糊处理(图像平滑处理)
```
Mat dealImage = srcImage.clone();
GaussianBlur(srcImage,dealImage,Size(3,3),0);
```
7.边缘检查
```
    Mat h_filler,v_filler,abs_h_filler,abs_v_filler;
    Sobel(dealImage,h_filler,CV_16S,1,0,3,1,0,BORDER_DEFAULT);      //横向检查
    convertScaleAbs(h_filler,abs_h_filler);
    Sobel(dealImage,v_filler,CV_16S,0,1,3,1,0,BORDER_DEFAULT);      //纵向检查
    convertScaleAbs(v_filler,abs_v_filler);
    addWeighted(abs_h_filler,0.5,abs_v_filler,0.5,0,dealImage);     //合并检查结果
```
8.图像二值化处理
```
    cvtColor(dealImage,dealImage,COLOR_BGR2GRAY);           //转为灰度图像
    threshold(dealImage,dealImage,95,255,THRESH_BINARY);    //二值化
```
9.图像补全处理
```
    Mat element = getStructuringElement(MORPH_RECT,Size(25,25));    //核阵列
    vector<vector<Point>> vec_count;
    morphologyEx(dealImage,dealImage,MORPH_CLOSE,element);          //闭运算（膨胀--腐蚀）
    morphologyEx(dealImage,dealImage,MORPH_OPEN,element);           //开运算（腐蚀--膨胀）
```
10.获取车牌区域(截取车牌范围的算法可自行判断设计)
```
    vector<vector<Point>> vec_area;
    findContours(dealImage,vec_area,RETR_EXTERNAL,CHAIN_APPROX_NONE);
    vector<Point> max_point_area;
    size_t num_max = 0;
    Point2f tmp_rect[4];
    for(size_t i = 0 ; i < vec_area.size() ; ++i)       //获取最大面积的区域，即车牌区域
    {
        minAreaRect(vec_area[i]).points(tmp_rect);
        double width = powf((tmp_rect[0].x - tmp_rect[1].x), 2) + powf((tmp_rect[0].y - tmp_rect[1].y), 2);
        width = sqrt(width);
        double height = powf((tmp_rect[0].x - tmp_rect[3].x), 2) + powf((tmp_rect[0].y - tmp_rect[3].y), 2);
        height = sqrt(height);
        if(height*width > num_max)
        {
            num_max = height*width;
            max_point_area = vec_area[i];
        }
    }
    if(num_max > 0)         //是否获取到区域
    {
        /*描绘最大区域的轮廓*/
        Mat carPlateAreaImage = srcImage.clone();
        for(size_t i = 0 ; i < max_point_area.size() ; ++i)
        {
            circle(carPlateAreaImage,max_point_area.at(i),2,Scalar(255,0,0));
        }

        /*描绘最大区域的矩形形态*/
        Point2f max_rect_corner_point[4];
        RotatedRect max_rect = minAreaRect(max_point_area);
        max_rect.points(max_rect_corner_point);
        for(size_t j = 0 ; j < 4 ; ++j)
        {
            line(carPlateAreaImage,max_rect_corner_point[j],max_rect_corner_point[(j+1)%4],Scalar(0,0,255),3);
        }
        //        imshow("carPlateAreaImage image",carPlateAreaImage);

        Mat carPlateImage = Mat(srcImage,max_rect.boundingRect());
        imshow("carPlateImage image",carPlateImage);
    }else{
        cout<<"Fail for fetch area of car plate";
    }
```

### Note
> * Demo中所提供的代码是经过本人亲测有效的，不过考虑到汽车图像的各种情况不同，定位效果不做保证，如果希望进行汽车图像车牌的复杂环境多种类型都可以定位的话，需要依靠机器学习的模型来进行，本文只是提供一种车牌定位的实现，后面有机会再分享机器模型的创建以及训练

[^note1]: 我将OpenCv的库文件编译成了一个文件，所以在LIBS中只添加了一个lib，如果不是这种情况，需要将所有lib都添加进来


