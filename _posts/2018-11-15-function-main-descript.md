---
layout: post
title: main函数参数详解
date: 2018-11-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - Linux
---

### ***概述***
        在示例程序中经常可以看到argc和argv这两个参数 ，在调试代码过程中遇到main函数为int main( int argc, char* argv[] ) 这种类型时往往会报错，或者是运行起来了但命令窗口一闪而过，没有出来结果。网上关于这方面的资料也有点乱，在看浅墨大大一书发现其中有关于这一方面的讲解甚为详细，抑制不住要与大家分享的冲动，现结合网上内容并予以实验，整理如下： 
        带形参的main函数，如 main( int argc, char* argv[], char **env ) ，是UNIX、Linux以及Mac OS操作系统中C/C++的main函数标准写法，并且是血统最纯正的main函数写法。 
        在如今的Visual Studio编译器中，main()函数带有参数argc和argv或者不带，即无论是否在函数体中使用argc和argv，返回值为void或不为void，都是合法的。 
        即至少有如下三种写法合法：
  1 返回值为整型带参的main函数
```
int main( int argc, char** argv )
{ 
     //函数体内使用或不使用argc和argv都可
     ……
     return 0;
}
```
  2 返回值为整型不带参的main函数
```
char    *ptr;
int main( int argc, char** argv )
{ 
     //函数体内使用了argc或argv
     ptr = argv[i];
     ……
     return 0;
}
```
  3 返回值为void且不带参的main函数
```
void main()
{ 
     ……
}
```

---

### ***argc、argv的具体含义***
        argc和argv参数在用命令行编译程序时有用。main( int argc, char* argv[], char **env ) 中
> * 第一个参数，int型的argc，为整型，用来统计程序运行时发送给main函数的命令行参数的个数，在VS中默认值为1。 
> * 第二个参数，：char*型的argv[]，为字符串数组，用来存放指向的字符串参数的指针数组，每一个元素指向一个参数。各成员含义如下
    - [x] argv[0]指向程序运行的全路径名 
    - [x] argv[1]指向在DOS命令行中执行程序名后的第一个字符串 
    - [x] argv[2]指向执行程序名后的第二个字符串 
    - [x] argv[3]指向执行程序名后的第三个字符串 
    - [x] argv[argc]为NULL 
    - [x] 第三个参数，char**型的env，为字符串数组。env[]的每一个元素都包含ENVVAR=value形式的字符串，其中ENVVAR为环境变量，value为其对应的值。平时使用到的比较少