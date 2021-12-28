---
title: CentOS7 编写服务
date: 2021-06-18 20:58:35.149
updated: 2021-06-20 21:12:12.432
url: http://summersea.top:8090/archives/centos7编写服务
categories: 
tags: CentOS 7 | 服务
---

在CentOS 7中，服务文件的存放位置为：
- 系统服务，开机不需要登录就能运行的程序
/usr/lib/systemd/system

```sh
[Unit]
Description=halo blog  # 服务描述
After=network.target   # 在什么服务后启动

[Service]
User=XXX               # 以XXX用户启动此服务
ExecStart=/usr/bin/java -jar /opt/bin/halo/halo-1.4.8.jar & #启动命令

[Install]
WantedBy=multi-user.target
```
WantedBy字段：表示该服务所在的Target。

Target的含义是服务组，表示一组服务。WantedBy=multi-user.target指的是，oracle所在的Target是multi-user.target（多用户模式）。
<font color=red>具体含义还不明确，有待解析。</font>

写完后需要执行以下命令来重新加载服务
```bash
systemctl daemon-reload
```

如果启动服务出错可以通过以下命令查看服务启动日志：
```bash
journalctl -u xxx.service
```
