---
layout: post
title: Qt应用打包发布流程Windows && Linux
date: 2019-01-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - Qt
---

#### 在Qt的使用过程中，我们会发现Qt的的执行文件在Qt的IDE中是可以直接Build出来并执行产生效果的，但是如果直接在编译文件夹下找到该执行文件运行或是要提供给第三方人员使用的时候，执行文件却不能直接使用，会有各式各样的库文件缺失等问题，以下是Qt在Windows与linux环境下的执行文件打包流程。
## Windows
### 工具需要
* windeployqt(Qt自带，无需安装)
* [Enigma Virtual Box](https://enigmaprotector.com/en/downloads.html)（需下载安装）

---

### 处理流程
* 通过Qt Creator编译出执行文件
* 在执行路径下找到编译出的执行文件
![](/img/2019-01-15-qt-packge-app/1.png)
* 将执行文件拷贝到一个空文件夹中
![](/img/2019-01-15-qt-packge-app/2.png)
* 通过Windows开始菜单找到Qt5.x.x for DeskTop(MinGw)并运行
![](/img/2019-01-15-qt-packge-app/3.png)
![](/img/2019-01-15-qt-packge-app/4.png)

* 在Qt的命令行中跳转至之前exe拷贝到的文件夹路径
![](/img/2019-01-15-qt-packge-app/5.png)
* 输入命令 ：`windeployqt 程序名`
    - 至此，经过`windeployqt`将执行文件所需的所有库文件都拷贝至执行文件路径，执行文件已经可以正常运行，接下来只要将库文件包括执行文件一起封装打包即可。
![](/img/2019-01-15-qt-packge-app/6.png)
![](/img/2019-01-15-qt-packge-app/7.png)
* 运行`Enigma Virtual Box`并选择执行文件导入
![](/img/2019-01-15-qt-packge-app/8.png)
* 将库文件添加到打包文件中
![](/img/2019-01-15-qt-packge-app/9.png)
* 点击`Process`按钮
![](/img/2019-01-15-qt-packge-app/10.png)
* `run`测试运行打包出来的执行文件，`close`关闭
![](/img/2019-01-15-qt-packge-app/11.png)
* 至此，打包出的执行文件就是可以完全脱离环境依赖运行的程序，可以发给第三方人员使用

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