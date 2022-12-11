# Docker 网络

## 前言

有时候我们会遇到这么个场景:

两个 Docker 容器之间需要通信，比如 Nginx 容器需要将流量转发到另一个 Web 容器。

这个时候就需要使用 Docker 网络了。

## Docker 网络的使用

### 命令帮助

```bash
$ docker network help
```

该命令可以查看 Docker 网络相关命令。

### 列出已存在的 Docker 网络列表

```bash
$ docker network list/ls
```

这个命令可以列出已存在的 Docker 网络，方便运维。

### 创建 Docker 网络

```bash
# 创建一个名为 "myNetwork" 的 Docker 网络
$ docker network create myNetwork
```

### 容器加入到 Docker 网络

```bash
# 将容器 "container1" 和 "container2" 连接到 "myNetwork" 网络上
$ docker network connect myNetwork container1
$ docker network connect myNetwork container2
```

### 查看某个网络的详情

```bash
# 查看 "myNetwork" 网络的详细信息
$ docker network inspect myNetwork
```

这个命令可以查看网络内的详情，也可以查看这个网络被多少个容器连接上了。

## 连接两个容器

### 原始 bridge 网络

所有的 docker 容器被创建时都会默认连接到一个名为 bridge 的网桥。

这个时候不同的容器之间其实就已经可以通信了，但只能使用 IP 进行通信。

### 新建网络

我们创建一个新的网络并将容器连接上该网络之后，容器之间可以使用容器名作为域名来通信。
