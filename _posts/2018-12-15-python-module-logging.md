---
layout: post
title: Python的logging模块使用
date: 2018-12-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - Python
---

#### 因为工作需要，这段时间在做一些python相关的东西，在此记录一下学习使用python过程中关于logging模块的一些知识点，本人之前是做C/C++开发的，有一定的知识基础，因此本文不做基础部分的讲解，仅记录在实际工作中直接性的使用操作
### logging模块
logging模块是python本身自带的一个提供日志输出记录的模块，使用简单方便，相比之下，C/C++针对log操作的需要开发者自己设计新建/打开文件，写入日志，日志分级等等操作，python在log处理方面确实避免了我们再造轮子的辛苦
### 日志分级
logging模块中的日志级别从低到高分为一下五个级别：
```
debug 调试信息
info 常规信息
warning 警告信息
error 错误信息
critical 严重错误信息
```

### logging模块使用
#### 导入模块
```
import logging
```
#### 新建Logger类
```
class Logger:
    def __init__(self,name = "Asa"):
        print("init class ball by %s"%name)
        self.log_file = name

        console_handle = logging.StreamHandler()
        console_format = logging.Formatter("%(message)s")
        console_handle.setFormatter(console_format)
        console_handle.setLevel(logging.INFO)

        file_handle = logging.FileHandler(self.log_file)
        file_format = logging.Formatter("%(asctime)s / %(name)s / %(levelname)s / %(message)s")
        file_handle.setFormatter(file_format)

        self.logger = logging.getLogger(self.log_file)
        self.logger.addHandler(console_handle)
        self.logger.addHandler(file_handle)
        self.logger.setLevel(logging.DEBUG)

    def debug(self, message):
        self.logger.debug(message)

    def info(self, message):
        self.logger.info(message)

    def war(self, message):
        self.logger.warning(message)

    def error(self, message):
        self.logger.error(message)

    def cri(self, message):
        self.logger.critical(message)
```
#### 调用
```
def main():
    a = Logger()
    a.debug("this is debug") //debug 信息输出
    a.info("this is info") //info信息输出
    a.war("this is warning") //warning信息输出
    a.error("this is error") //error信息输出
    a.cri("this is critical") //critical信息输出
```
#### 输出结果
* console
![](/img/2018-12-15-python-module-logging/1.png)

 * file
![](/img/2018-12-15-python-module-logging/2.png)

### 知识点
#### logger概述
* 一个正常工作的logger类一般具有`console`以及`file`两个`handle`，分别向`控制台`以及`文件`输出log信息，两者可以分别设置相关信息

#### handle使用
* `console_handle`的产生来自与`logging.StreamHandler()`，`file_handle`的产生来自于`logging.FileHandler()`
* `logging.FileHandler()`的参数应该传入log文件的路径，如果文件不存在则添加新建字段
```
Eg:
console_handle = logging.StreamHandler()
file_handle = logging.FileHandler(self.log_file,'w')
```

#### Level设置
* logging的日志分级在文章开头就已经说明，接下来重点讲的是`console_handle`与`file_handle`以及`logger`分别设置log的输出等级以及互相之间的影响
* logging只能输出高于设置级别的信息
* `console_handle`与`file_handle`可以各自设置各自的log输出级别，但是如果他们的级别低于logger的设置级别，那么低于logger设置级别的信息不会被输出
```
Eg:
console_handle.setLevel(logging.INFO)
file_handle.setLevel(logging.INFO)
self.logger.setLevel(logging.DEBUG)
```

#### 输出信息格式设置
* `console_handle`与`file_handle`可以设置各自的输出信息格式，两者之间并无联系
* 输出信息格式的可用全部字段如下
```
%(name)s：Logger的名字，并非用户名，详细查看
%(levelno)s：数字形式的日志级别
%(levelname)s：文本形式的日志级别
%(pathname)s：调用日志输出函数的模块的完整路径名，可能没有
%(filename)s：调用日志输出函数的模块的文件名
%(module)s：调用日志输出函数的模块名
%(funcName)s：调用日志输出函数的函数名
%(lineno)d：调用日志输出函数的语句所在的代码行
%(created)f：当前时间，用UNIX标准的表示时间的浮 点数表示
%(relativeCreated)d：输出日志信息时的，自Logger创建以 来的毫秒数
%(asctime)s：字符串形式的当前时间。默认格式是 “2003-07-08 16:49:45,896”。逗号后面的是毫秒
%(thread)d：线程ID。可能没有
%(threadName)s：线程名。可能没有
%(process)d：进程ID。可能没有
%(message)s：用户输出的消息
```
```
Eg:
console_format = logging.Formatter(" %(module)s")
console_handle.setFormatter(console_format)
file_format = logging.Formatter("%(asctime)s / %(name)s / %(levelname)s / %(message)s")
file_handle.setFormatter(file_format)
```

#### logger
* logger是logging产生的log输出端口，其本身使用时需将句柄添加进去，在通过logger输出信息时log便会通过logger将信息输出到其所包含的所有句柄中
```
Eg:
self.logger = logging.getLogger(self.log_file)
self.logger.addHandler(console_handle)
self.logger.addHandler(file_handle)
```

### Note
* logging本身还有通过配置文件直接做好logger全部设置的方法，但一般使用通过上述已经足够，本文就不再赘述，有机会再做延伸


