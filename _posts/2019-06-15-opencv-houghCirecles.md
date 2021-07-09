---
layout: post
title: 通过OpenCv的HoughCircles函数获取图片中的圆形
date: 2019-06-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - OpenCv
---

### 话不多说，代码如下：
```
int main(){
    Mat srcImage,grayImage;

    /*加载图片*/
    srcImage = imread(path);

    /*判断加载是否成功*/
    if(srcImage.empty())
    {
        cout<<"load pic fail"<<endl;
    }

    /*开辟dst矩阵，注意需与src矩阵大小类型一致*/
    Mat dstImage(srcImage.size(),srcImage.type());

    /*将src矩阵转化为灰度矩阵*/
    cvtColor(srcImage,grayImage,COLOR_BGR2GRAY);

    /*对灰度矩阵进行图像平滑处理(高斯模糊)*/
    GaussianBlur(grayImage,grayImage,Size(9,9),2,2);

    vector<Vec3f> circles;

    /*获取图片中的圆形*/
    HoughCircles(grayImage,circles,HOUGH_GRADIENT,1.6,10,230,40,12,20);

    /*在dst矩阵中绘制圆心与圆形*/
    for(size_t var = 0 ; var < circles.size() ; ++var)
    {
        Point center(cvRound(circles[var][0]),cvRound(circles[var][1]));
        int radius = cvRound(circles[var][2]);

        circle(dstImage,center,0,Scalar(0,255,0));
        circle(dstImage,center,radius,Scalar(255,0,0));
    }

    /*显示src矩阵与dst矩阵*/
    imshow("src image",srcImage);
    imshow("circle image",dstImage);
}
```
### Note
* `dstImage`的大小以及通道类型需与原图一致
* `HoughCircles`为全文最关键的函数，其作用在于获取图片中所有圆形的`圆心坐标`以及`半径`，调用此函数需细调其参数以确定所要获取的圆形的类型，它的内部参数含义如下：

```
void HoughCircles(InputArray image,OutputArray circles, 
                  int method, double dp, double minDist, 
                  double param1=100,double param2=100, 
                  int minRadius=0, 
                  int maxRadius=0 )
dp:         累加器分辨率与图像分辨率的反比，如果dp=1，累加器的分辨率与输入图像相同。
            如果dp=2，蓄能器有宽度和高度的一半。对于HOUGH_梯度_ALT，建议值为dp=1.5，
image:      8bit，单通道的灰度图数据
method:     检测方法，有HOUGH_GRADIENT以HOUGH_GRADIENT_ALT两种方法选择
param1:     在HOUGH_GRADIENT和HOUGH_GRADIENT_ALT两种模式时，
            它是传递给Canny边缘检测器的两个阈值中较高的一个（较低的阈值小两倍），
            注意HOUGH_GRADIENT_ALT模式使用Scharr 算法，所以阈值通常较高
param2:     该值越小，可以检测到更多根本不存在的圆，该值越大，能通过检测的圆就更加接近完美的圆形
circles:    寻找到的圆形数据的输出向量，每个向量中包含3/4个元素，
            其含义位(x,y,radius)/(x,y,radius,votes)
minDist:    检测到的圆中心之间的最小距离。如果参数是太小，除了一个真实的圆外，
            可能会错误地检测到多个相邻圆。如果是的话太大，可能会漏掉一些圆圈
minRadius:  检测圆形的最小半径
maxRadius:  检测圆形的最大半径
```
