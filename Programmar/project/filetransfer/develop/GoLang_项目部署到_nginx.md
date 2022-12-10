# GoLang 项目部署到 Nginx

## GoLang 项目构建——Docker

### go 命令构建

使用如下命令即可直接构建:

`go build -o <输出文件名> .\cmd\main.go`

然后本地就会有一个文件出现

### DockerFile 构建

为了方便使用和重复构建，以及为未来可能用到的 docker-compose 做准备，笔者打算使用 DockerFile 构建一个镜像。

过程中出现了一些问题。

#### 构建失败如何自己打印一些输出以帮助判定问题原因

我们可以在 DockerFile 内加入一些自己辅助用的命令。例如想要获取目录结构可以使用如下命令:

- `RUN ls -l`

这样，执行 `docker build` 命令时，我们就可以获取到目录信息了。

**注意**: `docker build` 命令会折叠一些输出，所以我们需要加一些参数确保我们的输出不会丢失。

- `docker build -f .\docker\Dockerfile --progress plain --no-cache .`

问题解决后为了加快构建速度，可以删除辅助用的命令:

- `docker build -f .\docker\Dockerfile -t filetransfer:0.0.1-snapshot .`

### 镜像上传与更新

首先使用 `docker tag` 命令为镜像打上标签。

- `docker tag filetransfer:0.0.1-snapshot <myname>/filetransfer:0.0.1-snapshot`

我们也可以在构建阶段直接指定产品的 tag，这么做后做自动化时会简化一个步骤。

- `docker build -f .\docker\Dockerfile -t <myname>/filetransfer:0.0.1-snapshot .`

使用 `docker push` 命令将镜像推送到 Docker Hub。

- `docker push <myname>/filetransfer:0.0.1-snapshot`

这样笔者的镜像就推送到 Docker Hub 上去了，服务器只需要使用 docker pull 和 docker run 命令即可启动镜像。

## 项目部署

要做 2 件事:

1. 让项目在服务器上跑起来
2. 配置 `nginx.conf` 以保证让请求能通过 Http 端口转发到这个应用上。

### 部署并启动项目

这个项目是前后端分离部署的，我们仅需要在服务器上拉取刚刚上传的镜像然后跑起来即可。

```shell
docker pull <myname>/filetransfer:0.0.1-snapshot
docker run -dp 8081:8081 <myname>/filetransfer:0.0.1-snapshot
```

### 配置 nginx.conf

通过如下配置可以将流量转发到我们的后端应用上，只要发送数据到 file 下的接口上就可以。

```
location /file {
    proxy_pass http://127.0.0.1:8081;
}
```