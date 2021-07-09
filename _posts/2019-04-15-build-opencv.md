---
layout: post
title: Ubuntu16.04编译OpenCV(CMake),以及Qt的加载
date: 2019-04-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - Linux
    - OpenCv
    - CMake
---

在开发环境的选择上，很多开发者都觉得Linux的自由度要比Windows强很多，在此记录一下`OpenCv`在ubuntu下是如何编译并使用的。

* [先放个我编译好的OpenCV的lib](https://github.com/AsaBoring/OpenC-4.3_release)

### 工具准备

#### Qt
* 这里就放一个Qt的下载[Link](http://download.qt.io/archive/qt/)

####  OpenCv源码

* 网上有很多`OpenCv`的下载地址，不过国内的话我还是推荐`Gitee`，网速稳健[https://gitee.com/mirrors/opencv?_from=gitee_search](https://gitee.com/mirrors/opencv?_from=gitee_search)

#### Cmake

* `cmake`在Linux上的安装通过以下这条命令就可以了
`sudo apt-get install cmake`

### 编译过程

* 将下载好的`OpenCv`源码解压
![解压opencv](/img/2019-05-15-build-opencv-contrib/opencv.jpg)
* 在`src`文件夹下新建一个自己的`build文件夹`用来`执行cmake`
![在Src路径下新建build文件夹](/img/2019-05-15-build-opencv-contrib/mkdir_build.png)
![通过命令行进入建好的文件夹](/img/2019-05-15-build-opencv-contrib/insert_folder.png)
* 执行命令
`cmake -D WITH_TBB=ON -D BUILD_NEW_PYTHON_SUPPORT=ON -D WITH_V4L=ON -D INSTALL_C_EXAMPLES=ON -D BUILD_EXAMPLES=ON -D WITH_OPENGL=ON -D WITH_EIGEN=OFF -D BUILD_opencv_world=ON -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local/OpenCV/Release ..`
![进行make file的生成](/img/2019-05-15-build-opencv-contrib/make_file.png)
![make file生成成功](/img/2019-05-15-build-opencv-contrib/make_success.png)
* Note:
1 `CMAKE_INSTALL_PREFIX`后面的参数`/usr/local/OpenCV/Release`是库文件最终install完毕后的位置，可自行设定
2 `BUILD_opencv_world=ON`是将所有的库文件编译为一个文件
3 `CMAKE_BUILD_TYPE`控制编译为`Release`版本还是`Debug`版本
* 执行命令
`make -j2`
![执行make](/img/2019-05-15-build-opencv-contrib/execute_make.png)
![make完毕](/img/2019-05-15-build-opencv-contrib/make_success.png)
* 执行命令
`sudo make install`
![make install](/img/2019-05-15-build-opencv-contrib/make_install.png)
* 到路径`/usr/local/OpenCV/Release/lib`下可查看编译结果
![lib位置](/img/2019-05-15-build-opencv-contrib/lib_location.png)

### Qt Demo

* `Pro`文件添加`OpenCV`库`lib`与`include`

```
INCLUDEPATH += /usr/local/OpenCV/Release/include/opencv4 \
               /usr/local/OpenCV/Release/include/opencv4/opencv2

LIBS += -L/usr/local/OpenCV/Release/lib \
        -lopencv_world
```

* main.cpp

```
#include <iostream>
#include <QCoreApplication>
#include <opencv2/opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc, char *argv[])
{
    QCoreApplication a(argc, argv);

    Mat srcImage = imread("/home/asa/桌面/Asa/qt_opencv_demo/opticalfb.jpg",0);
    if(srcImage.empty())
    {
        cout<<"load pic failed"<<endl;
        return -1;
    }
    imshow("demo",srcImage);

    return a.exec();
}
```


