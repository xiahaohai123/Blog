# docker 构建镜像指定运行平台

## 前言

笔者自己的一个项目，一直在 windows 上构建镜像然后上传到 docker hub，直到笔者用 Macbook 构建镜像时，发现在服务器上跑不起来了。

一看 docker logs，报出了一个不明所以的原因:

```shell
standard_init_linux.go:178: exec user process caused "exec format error"
```

本文就该问题写一份文档。

## 问题原因

笔者的 Macbook 使用的芯片是 M1，会被 docker 认定为 arm64 架构。

docker 构建镜像时，如果不指定镜像的运行平台，就会默认使用构建机器的平台环境作为平台。

所以 docker 在 windows 上会构建出 amd64 平台运行的镜像，而在 Macbook M1 上会构建出 arm64 平台运行的镜像。

而笔者的服务器，是 x86_64 架构，对应 amd64 平台:

```shell
[root@VM-0-6-centos ~]# uname -a
Linux VM-0-6-centos 3.10.0-1160.45.1.el7.x86_64 #1 SMP Wed Oct 13 17:20:51 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
```

## 解决问题

在构建镜像时，指定运行平台即可

```shell
docker build --platform=linux/amd64 -f <Dockerfile> -t <tag> .
```

## 参考文献

1. [standard_init_linux.go:178: exec user process caused "exec format error"](https://stackoverflow.com/questions/42494853/standard-init-linux-go178-exec-user-process-caused-exec-format-error#comment120165301_64215125)