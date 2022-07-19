---
title: java heap dump
date: 2022-03-15 16:59:14.297
updated: 2022-03-15 16:59:32.018
url: http://summersea.top:8090/archives/javaheapdump
categories: 
tags: 
---



# 背景

工作中遭遇了现场出现`OOM`的问题，所以记录一下`OOM`问题的排查流程。


# 总体流程

1. 获取java程序堆镜像文件，也就是以`.hprof`后缀的文件。
2. 把堆镜像文件塞入分析程序，看着分析程序分析。


# 搞到堆镜像文件


## 1. 找出对应java程序的进程号`pid`

### JPS

命令行输入`jps`可以查询java相关的进程号，`jps -l`可以查询更详细的信息。

### ps

在`linux`环境下，`ps auxf | grep xxx`可以快速找出你需要锁定的应用程序，并从中获取进程号。

## 2. 转储堆内存镜像

转储堆内存镜像的办法还挺多的，这里一一列出。

### jmap

执行如下命令即可获得堆内存镜像。
`jmap -dump:live,format=b,file=/home/xx/heap.hprof <pid>`

- 注意，执行这个命令的时候，需要以目标应用程序的启动者用户的身份来执行，例如：

`sudo -u testuser jmap -dump:live,format=b,file=/home/xx/heap.hprof <pid>`

### jstack

`jstack <pid>`

同理，也需要以应用程序启动者的身份来执行该命令。

### 重点：JVM参数

一般情况下，发生`OOM`了以后，我们再去手动转储时，已经存在了部分的时间差，这样转储出来的镜像价值并没有出现`OOM`的瞬间转储出来的镜像价值高。

Java也为我们准备了这么一组JVM参数，以确保`OOM`发生时立即转储堆内存镜像供我们分析：
`-XX:+HeapDumpOnOutOfMemoryError`
另外，我们可以重定向转储的镜像文件的位置:
`-XX:HeapDumpPath=/home/test`
这样，当OOM发生时，JVM就会在/home/test目录下生成一份`heap dump`文件，例如`java_pid2108.hprof`


# 分析Heap Dump

转储出来的.hprof文件可以送进特定的应用程序进行分析，以下是阶段常用的两个软件：

- MAT(Memory Analyzer)
免费，JDK11
- Jprofiler
收费

