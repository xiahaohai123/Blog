---
title: CentOS 7 防火墙操作
date: 2021-04-02 17:51:18.426
updated: 2021-08-10 10:12:46.666
url: http://summersea.top:8090/archives/centos7fang-huo-qiang-cao-zuo
categories: 
tags: CentOS 7
---

列出所有已开放端口：
```shell
firewall-cmd --list-ports
```

新增端口 --permanent表示是否永久添加
```shell
firewall-cmd --add-port=8080/tcp [--permanent]
firewall-cmd --add-port=8080/udp [--permanent]
```

删除端口
```shell
firewall-cmd --remove-port=8080/tcp
```
重新载入
```shell
firewall-cmd --reload
```
直接重启防火墙也可以
```shell
systemctl restart firewalld
```
以上是端口配置

接下来是防火墙信任列表 rich rule

列出信任列表
```shell
firewall-cmd --list-rich-rules
```

添加信任列表格式
```shell
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="[源ip]" port port="[端口]" protocol="tcp/udp" accept"
```
举个例子
例子1
```shell
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="[12.12.12.12]" port port="[5633]" protocol="tcp" accept"
```
例子2
```shell
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="[12.12.12.12]" port port="[5633]" accept"
```
例子3
```shell
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="[12.12.12.12]" accept"
```

可以发现添加信任列表的格式是添加一个字符串。

除了rich-rule，还有一种规则叫direct rule。
显示direct rule:
```shell
firewall-cmd --direct --get-all-rules
```
direct规则的定义不会在`--list-all`中显示。