# 安装 docker-compose

## 正文

[https://github.com/docker/compose/releases/](https://github.com/docker/compose/releases/) 下有工具的发布信息。

我们可以在自己的服务器上执行 `uname -s` 和 `uname -m` 获取操作系统和架构信息，来选择 `docker-compose` 的类型。

将工具下载到服务器上,保存为 `/usr/local/bin/docker-compose` 以保证这个工具可以在任意目录中被调用，注意赋予权限。

运行 `docker-compose version` 可以查看版本号，以及验证安装是否成功。

之后使用命令 `docker-compose up -d` 以启动容器，并让容器在后台工作。