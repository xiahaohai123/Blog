---
title: JVM时区与操作系统时区的关系
date: 2021-11-02 16:53:32.273
updated: 2021-11-02 16:54:40.443
url: http://summersea.top:8090/archives/jvm时区与操作系统时区的关系
categories: 
tags: 操作系统 | Java
---

## 现象
当Linux系统使用通常手段修改时区后，JVM时区并不会跟着改变，即使修改时区后再启动Java程序。

## 解决方案
如果只是临时想要确定JVM的时区，可以在JVM参数中增加参数以指定时区，比如中国的所在的东8区:
```
-Duser.timezone=Asia/Shanghai
```