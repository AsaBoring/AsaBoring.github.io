---
layout: post
title: Linux文件写入同步
date: 2018-10-15 15:32:24.000000000 +09:00
author:     "Asa"
header-img: "img/post-bg-alibaba.jpg"
tags:
    - Linux++
---

####    之前在一个项目中总是会出现设备突然断电后，某些文件成为0byte或者文件数据只写入一半的情况，经过研究后发现该问题的复现操作为在写入文件时进行突然的断电或将程序kill，再追入其中发现这个问题      实质上与linux的文件系统有关，linux为了提升系统的运行速率设计了虚拟文件系统，该系统实际上并不会 立刻将你通过代码写入文件的数据写入磁盘，而是以一种缓冲区的方式先完成了写入数据的临时性存储， 完成这一步，linux的虚拟文件系统就会返回写入完成，但此时数据并未写入到磁盘，如果此时发生断电等 异常情况，就会出现开篇所述的问题，现将文件数据写入流程以及一些注意点记录如下：

### 文件数据的实际写入以及对应Func如下
```
fileWrite->usrCache: fwrite
usrCache->sysCache: fflush
sysCache->diskCache: fsync
diskCache->disk: barrier
```
* usrCache --> sysCache
    - 应用程序write文件完成之后，
文件数据是暂存于USER CACHE，
利用FFLUSH，将USER CACHE
中的数据刷新到SYS CACHE

* sysCache-diskCache/disk
    - 利用SYNC将SYS CACHE中的数据
刷新到DISK上，如果DISK存在DISK CACHE，那么FSYNC只保证数据刷新到DISK CACHE，FSYNC函数不关心DISK CACHE到DISK的过程，如果无DISK CACHE，那么FSYNC可以保证数据写到了DISK上

* diskCache --> disk
    - 如果数据从DISK CACHE到DISK的过程中断电，那么数据存在丢失风险，解决方案有以下三种：
        - 启用备用电源，保证DISK数据完成传输。
        - 使用内核WRITE BARRIER机制（使IO操作后显式刷新DISK来保证FSYNC后，数据一定写到磁盘上）
        - FSYNC后加入延时，通过预留一定时间来达到数据写入DISK的目标

### Note
* Fopen
    - FILE *fopen(const char *path, const char *mode);
        - mode 设置为 RB+;以读写方式打开文件，文件必须存在，否则报错。文件指针置于文件头。以覆盖的方式去写文件，保留源文件中未覆盖内容。
        - mode 设置为WB+;以读写方式打开文件，文件不存在则创建文件，存在会先清空后写数据。文件指针置于文件头。先清空后写数据的方式写文件。
    - 由RB+和WB+的文件处理逻辑可以得出：
        - RB+方式打开文件，如果出现异常中断，会导致文件损坏，但不会出现0字节现象。
        - WB+方式打开文件，如果出现异常中断，会导致文件损坏，甚至于0字节现象发生。

* O_DIRECT
    - 直接IO操作:Linux允许应用程序在执行磁盘IO时绕过缓冲区高速缓存，从用户空间直接将数据传递到文件或磁盘设备，称为直接IO（direct IO）或者裸IO（raw IO）。
    - O_DIRECT是C语言open函数的flag参数，fopen不支持。为了保证IO同步，通常必须要和O_SYNC参数一同使用。而O_SYNC参数的作用是每写一次文件都会同步到磁盘，在文件整理过程中会循环写文件，如果开启O_SYNC会大大降低性能。
    - 同时使用直接IO需要遵守的一些限制：制：
        - 用于传递数据的缓冲区，其内存边界必须对齐为块大小的整数倍 
        - 数据传输的开始点，即文件和设备的偏移量，必须是块大小的整数倍 
        - 待传递数据的长度必须是块大小的整数倍。 

* SYNC
    - sync命令：sync将内存中缓冲的所有数据写出到磁盘。sync除了执行sync，syncfs，fsync和fdatasync系统调用外什么也不做。
    - sync()函数：将文件系统下的所有文件都同步，它总是返回成功，因而存在数据还未实际写入磁盘就已返回成功的可能，故此sync()函数不可靠。
    - syncfs(fd)：成功返回0,失败返回-1,syncfs()类似于sync()，但是仅同步fd文件。
    - fsync(fd)函数：把文件描述符fd指向的文件缓存在内核中的所有已修改的数据写入文件系统，包含数据与文件元数据（文件大小，文件修改时间等）。但是fsync不会写入对指向文件的目录项的修改，也就是说如果新创建了一个文件，要是确保下次能正确读出的话，就需要把所在目录也fsync一下。阻塞式，直到设备数据传输完成。
    - fdatasync(fd)函数：和fsync作用差不多，但是不会写入对下次正确读取文件作用不大的一些元数据（比如上次访问时间，上次修改时间等），但是大小如果改变了，是会写进去的。阻塞式，只同步文件数据。