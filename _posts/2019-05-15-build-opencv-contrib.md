---
layout: post
title: windows10通过mingw(Qt5.14)+CMake编译opencv(4.3)+contrib(4.3),以及相关问题解决
date: 2019-01-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - Windows
    - Qt
    - OpenCv
    - Contrib
---

####    有关`OpenCv`在windows下的编译网上教程纷杂，但在我研究的过程中发现如果只是编译`OpenCv`本身的话还好，一旦添加`Contrib`之后就会有各种稀奇古怪的问题出现，在这里出一版经过本人研究确定可行的方案，以及相关的问题的解决

---
### 工具准备
#### Qt
* 这里就放一个Qt的下载Link,`Mingw`使用Qt自带的版本就可以
http://download.qt.io/archive/qt/

#### Cmake
* [https://cmake.org/download/](https://cmake.org/download/)

####  OpenCv源码
* [https://github.com/opencv/opencv/releases](https://github.com/opencv/opencv/releases)

####  OpenCv_Contrib源码
* [https://github.com/opencv/opencv_contrib/releases](https://github.com/opencv/opencv_contrib/releases)

> * Note：OpenCv与OpenCv_Contrib的版本一定要保持一致

---
### 编译过程
* 安装Qt
* 安装CMake

* Note:Qt与cmkae的安装较为简单，点击安装包直接安装即可，需要注意的是Qt安装的时候选择Mingw编译套件

* 安装完毕后将`CMake`以及`Mingw`的安装路径添加到系统路径中
![sys path配置](/img/2019-05-15-build-opencv-contrib/sys_path.png)

* 将OpenCv源码以及OpenCv_Contrib源码解压至同一个文件夹，我这里用的都是当前最新的4.3版本
![解压源码](/img/2019-05-15-build-opencv-contrib/decompress_code.png)
* 新建一个自己的build文件夹
![新建build文件夹](/img/2019-05-15-build-opencv-contrib/new_folder_build.png)
* 启动CMake的GUI
![启动cmake](/img/2019-05-15-build-opencv-contrib/startup_cmake.png)
* `Browse Source`选择`OpenCv4.3`
![选择opencv4.3源文件](/img/2019-05-15-build-opencv-contrib/chose_opencv_file.png)
* `Browse Build`选择新建好的编译文件夹
![选择编译文件夹](/img/2019-05-15-build-opencv-contrib/chose_build_file.png)
* 点击`Configure`，选择`MinGW Makefiles`以及`Specify native compliers`，点击`Next`
![配置MinGW编译](/img/2019-05-15-build-opencv-contrib/setup_mingw.png)
* 选择`gcc`以及`g++`的编译器的位置，然后选择`Finish`即可
![配置编译器](/img/2019-05-15-build-opencv-contrib/setup_complier.png)

> * Note:前面我们以及安装过Qt，编译器在安装路径的...Tools\mingw730_64\bin下

* 等待`Configure`完成
![等待Configure完成](/img/2019-05-15-build-opencv-contrib/wait_config_done.png)
![Configure完成](/img/2019-05-15-build-opencv-contrib/config_done.png)
* 在`Search`中搜索`OPENCV_ENABLE_ALLOCATOR_STATS`取消选中，搜索`BUILD_opencv_world`选中，重新`Configure`
![OPENCV_ENABLE_ALLOCATOR_STATS取消选中](/img/2019-05-15-build-opencv-contrib/search_item.png)
![BUILD_opencv_world选中](/img/2019-05-15-build-opencv-contrib/unchose_item.png)
![再次configure](/img/2019-05-15-build-opencv-contrib/config_again.png)
* 点击`Generate`进行Makefile的生成
![Generate](/img/2019-05-15-build-opencv-contrib/generate.png)
* 以管理员权限打开命令行并跳转到新建文件夹，执行命令`mingw32-make -j8`
![跳转执行命令](/img/2019-05-15-build-opencv-contrib/execute_cmd.png)
![开始编译](/img/2019-05-15-build-opencv-contrib/start_build.png)
* 编译完毕，执行`mingw32-make install`
![编译成功](/img/2019-05-15-build-opencv-contrib/build_success.png)
![执行install](/img/2019-05-15-build-opencv-contrib/run_install.png)
* install完毕后即可在编译文件夹下的此路径`...\build_opencv\Asa_build\install\x64\mingw\bin`找到编译好的lib文件，include文件在编译文件夹下的`...\install`中，将`include`以及`lib`提出来便是编译好的完整的库文件以及包含文件了
sn
---

### 过程中的问题
#### 缺失`ffmpeg.dll` `ffmpeg_64.dll`文件
* 下载链接https://github.com/AsaBoring/opencv_ffmpeg_file，将下载好的文件放在`opencv-4.3.0/3rdparty/ffmpeg`路径下

#### `boostdesc_bgm.i: No such file or directory`没有`boostdesc_bgm`等文件
* [从这里下载](https://github.com/AsaBoring/boostdesc_bgm-files-build-opencv-contrib-needed-
)`boostdesc_bgm.i等文件`的压缩包后将文件解压至`opencv_contrib-4.3/modules/xfeatures2d/src`中即可

#### `features2d/test/test_detectors_regression.impl.hpp: 没有那个文件或目录`  
* 1.将`opencv-4.3/modules/features2d/test/`文件夹下的
`test_descriptors_regression.impl.hpp`  
`test_detectors_regression.impl.hpp`
`test/test_detectors_invariance.impl.hpp`
`test_descriptors_invariance.impl.hpp`
`test_invariance_utils.hpp`
拷贝到`opencv_contrib-4.3/modules/xfeatures2d/test/`路径下
* 2将`opencv_contrib-4.3/modules/xfeatures2d/test/test_features2d.cpp`文件中的代码：
`#include features2d/test/test_detectors_regression.impl.hpp`
`#include features2d/test/test_descriptors_regression.impl.hpp`
改为：
`#include "test_detectors_regression.impl.hpp"`
`#include "test_descriptors_regression.impl.hpp"
`
* 3将`opencv_contrib-4.3/modules/xfeatures2d/test/test_rotation_and_scale_invariance.cpp`文件中的代码:
`#include "features2d/test/test_detectors_invariance.impl.hpp" 
`
`#include "features2d/test/test_descriptors_invariance.impl.hpp"
`改为：
`#include "test_detectors_invariance.impl.hpp"`
`#include "test_descriptors_invariance.impl.hpp"`

### 库文件链接
* 老规矩，这里放个本人编译好的库链接
[https://github.com/AsaBoring/OpenCV_4.3_release_lib/blob/master/OpenCV4.3_contrib_release_windows.zip](https://github.com/AsaBoring/OpenCV_4.3_release_lib/blob/master/OpenCV4.3_contrib_release_windows.zip)




