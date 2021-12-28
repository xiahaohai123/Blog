---
title: CentOS7 服务方式启动java程序
date: 2021-04-12 00:25:53.372
updated: 2021-06-21 14:04:25.274
url: http://summersea.top:8090/archives/centos7fu-wu-fang-shi-qi-dong-java-cheng-xu
categories: 
tags: Linux | 操作系统
---

service文件一般存储在/etc/systemd/system目录下
![image.png](http://summersea.top:8090/upload/2021/04/image-f1f7ef32fa1c41bd801f6156488263aa.png)

写一个自己的service
```sh
vim keygenerator.service
```
```sh
[Unit]

Description=key generator server

After=syslog.target

[Service]

ExecStart=/usr/bin/java -jar -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 /opt/keygeneratorserver.jar

[Install]

WantedBy=multi-user.target
```
ExecStart表示启动服务实际执行的命令，好像只接受绝对路径，<font color="red">之后看看有没有办法使用相对路径</font>