---
layout: post
title: Qt应用打包发布流程Windows && Linux
date: 2019-01-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - Qt
---

在Qt的使用过程中，我们会发现Qt的的执行文件在Qt的IDE中是可以直接Build出来并执行产生效果的，但是如果直接在编译文件夹下找到该执行文件运行或是要提供给第三方人员使用的时候，执行文件却不能直接使用，会有各式各样的库文件缺失等问题，以下是Qt在Windows与linux环境下的执行文件打包流程。
## Windows
### 工具需要
1 windeployqt(Qt自带，无需安装)
2 [Enigma Virtual Box](https://enigmaprotector.com/en/downloads.html)（需下载安装）
---
### 处理流程
1 通过Qt Creator编译出执行文件
2 在执行路径下找到编译出的执行文件
![](https://upload-images.jianshu.io/upload_images/19406162-53606bb90debea18.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3 将执行文件拷贝到一个空文件夹中
![](https://upload-images.jianshu.io/upload_images/19406162-9f82f573c5a358aa.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4 通过Windows开始菜单找到Qt5.x.x for DeskTop(MinGw)并运行
![](https://upload-images.jianshu.io/upload_images/19406162-3f90e3aeedcf95af.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19406162-a17dabe7e8634191.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

5 在Qt的命令行中跳转至之前exe拷贝到的文件夹路径
![](https://upload-images.jianshu.io/upload_images/19406162-b1a314a273f60701.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
6 输入命令 ：`windeployqt 程序名`
-  至此，经过`windeployqt`将执行文件所需的所有库文件都拷贝至执行文件路径，执行文件已经可以正常运行，接下来只要将库文件包括执行文件一起封装打包即可。
![](https://upload-images.jianshu.io/upload_images/19406162-8ab1c946623d9d68.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/19406162-cbb2a9b8535a2a2f.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
7 运行`Enigma Virtual Box`并选择执行文件导入
![](https://upload-images.jianshu.io/upload_images/19406162-fec8e787eb15a098.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
8 将库文件添加到打包文件中
![](https://upload-images.jianshu.io/upload_images/19406162-94ebbe0a151b1711.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
9 点击`Process`按钮
![](https://upload-images.jianshu.io/upload_images/19406162-3f1a0e546a4e9d0f.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
10 `run`测试运行打包出来的执行文件，`close`关闭
![](https://upload-images.jianshu.io/upload_images/19406162-5396999c6973dac8.PNG?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
11 至此，打包出的执行文件就是可以完全脱离环境依赖运行的程序，可以发给第三方人员使用

---
## Linux
### 工具需要
#### shell脚本收集执行文件相关库文件：
```
#!/bin/bash
/*asa server为目标执行文件*/
LibDir=$PWD"/lib"
$(mkdir $LibDir)
lib_list=`ldd asaServer | awk '{print $3}'`
for var in $lib_list
do
    cp $var $LibDir
done
```
### 处理流程
与Windows基本一致