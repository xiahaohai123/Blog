---
title: java.net.SocketException: Permission denied的一种情况
date: 2021-06-20 22:01:39.838
updated: 2021-06-20 22:01:39.838
url: http://summersea.top:8090/archives/javanetsocketexceptionpermissiondenied-de-yi-zhong-qing-kuang
categories: 
tags: Linux | 权限
---

### 背景
在使用非root用户启动服务时，发现服务启动失败，但是使用root用户启动就可以成功启动。
环境是CentOS 7。

### 原因
在[stackoverflow的一篇问答](https://stackoverflow.com/questions/46316198/java-net-socketexception-permission-denied-connect-starting-tomcat-7)中找到了这个情况的解：
>You'll need to run Tomcat as root for it to be able to bind to port 80. All ports below 1024 require superuser permissions for binding.
>
>Just run your command with SUDO

大意是端口号小于1024的需要超级管理员的权限来绑定，而遇到问题的人刚好在绑定80端口来启动Tomcat。

### 验证
我试了试修改程序绑定的端口为8443，成功以非root用户启动，说明确实是端口号的问题，那么如果需要用443端口启动，需要用root用户或者一些手段将443端口的数据转发到高端口。
转发手段有nginx反向代理，也有路由转发。