# Nginx 配置 SSL 证书

## 关键字

TLS, SSL, Cert, HTTPS

## 前言

之前自己的项目部署到服务器上了，但是前端使用了 Service Worker 技术，这要求网站连接必须是 HTTPS。

出于安全性考虑，我们也需要一个 SSL 证书来保证通信安全。

## SSL 证书准备

### 花钱

浏览器内置的签发机构签发的证书需要钱来购买，但笔者的环境目前没有必要因为这个作出开销，所以不选择花钱。

### 自签证书

自签证书会变成非受信的证书。

但是笔者想到，都 2022 年了，都开始强制使用 HTTPS 了，难道这世道是强买强卖的不成？

之后 AI 给了个答案，可以用 Let's Encrypt。

### Let's Encrypt

官方推荐 certBot，但是 acme.sh 好像得到了更加广泛的支持，于是笔者选择使用 acme.sh。

#### acme.sh

##### 安装

中国大陆服务器的安装方式稍有些不同，得使用国内的 gitee 库。
使用如下命令进行安装:

```bash
$ git clone https://github.com/acmesh-official/acme.sh.git
$ git clone https://gitee.com/neilpang/acme.sh.git # 中国大陆服务器，上面的命令用不了就用这个
$ cd acme.sh
$ ./acme.sh --install -m my@example.com
```

邮箱可以填写自己的邮箱，如果acme.sh使用zeroSSL CA，则这里的-s参数指定的邮箱可以关联到已有的zeroSSL账号。关联成功后，通过acme.sh生成的zeroSSL证书会在zeroSSL网站的控制面板上显示：

- [Free SSL Certificates and SSL Tools - ZeroSSL](https://zerossl.com/)

##### 域名验证与证书生成

生成证书的过程需要通过 acme 协议对**域名所有权**进行验证。使用如下命令进行验证:

```bash
$ sh acme.sh --issue -d mydomain.com -d www.mydomain.com --webroot /home/wwwroot/mydomain.com/
```

这是 webroot 模式验证，需要服务器上已经运行了 http 服务，并且可以通过公网访问。具体原理是 acme 客户端会在我们的服务器上方一个特殊的验证文件，然后 acme 服务器会通过 http 请求，请求我们的域名来读取文件，以此来判定我们是不是真的掌握域名。

注意切换 ca 机构，笔者做这个事情的时候 zerossl 不工作

```bash
$ ./acme.sh --set-default-ca --server letsencrypt
```

针对 Nginx 服务器，还有一种 Nginx 专用的命令，更加智能。

```bash
$ sh acme.sh --issue -d mydomain.com --nginx
```

**注意事项**: 执行命令时，需要和 Nginx 服务器处于同一空间，例如 Nginx 服务器在 Docker 容器内，那么 acme.sh 也应当在容器内。

##### Docker 内的操作

1. 首先需要将 acme 客户端从宿主机复制到容器内
2. 到对应目录执行安装命令
    1. 笔者的 Nginx 容器生成定时任务会失败，安装说明提示笔者用命令 `acme.sh --cron` 可以更新证书。
3. 执行验证与证书生成命令

```bash
$ docker cp /opt/acme.sh/acme some-nginx:/opt
$ sh acme.sh --install -m my@example.com
$ sh acme.sh --issue -d mydomain.com -d www.mydomain.com --nginx
```

整个运维过程都有些麻烦，而且容器要是删了再新建实例就不好搞了，不建议。

##### standalone 方式

acme 自己运行一个 webserver 来监听 acme 服务器发送的请求，依赖 socat，需要提前安装。

```bash
$ yum install socat
```

使用 standalone 方式来进行域名所有权验证对于 Nginx 环境被高度隔离的环境是比较友好的。

```bash
$ sh acme.sh --issue --standalone -d mydomain.com -d www.mydomain.com --httpport 88
# --httpport 可以指定 webserver 监听的端口，可以不和正在服务的端口起冲突。
```

可惜，一样走不通。赶快调转注意力到 certbot 上。

#### certbot

##### 注册证书

- centos

```shell
$ yum install snapd
$ systemctl start snapd
$ snap install core
$ ln -s /var/lib/snapd/snap /snap
$ snap install --classic certbot
$ ln -s /snap/bin/certbot /usr/bin/certbot
$ certbot certonly --standalone
```

- ubuntu

```shell
apt install snapd
systemctl start snapd
snap install core
snap refresh core
apt-get remove certbot
snap install --classic certbot
ln -s /snap/bin/certbot /usr/bin/certbot
certbot certonly --standalone
```

命令 `certbot certonly --standalone` 会注册证书，后续 `certbot renew --dry-run` 命令会根据该注册信息更新

之后就是照着提示一步一步操作下去，这个方法需要关闭 web 服务器一段时间，因为需要监听 80 端口。

之后会弹出如下信息，就是我们要的证书了

> Certificate is saved at: /etc/letsencrypt/live/domain/fullchain.pem
>
> Key is saved at: /etc/letsencrypt/live/domain/privkey.pem

如果不想在默认目录使用证书，把证书搬到我们要使用的目录即可。


##### 更新证书

```bash
$ certbot renew
```

## SSL 证书配置

nginx.conf 内配置一个 server

```
server {
  listen 443 ssl;
  ssl_certificate /some/ssl/domain/fullchain1.pem;
  ssl_certificate_key /some/ssl/domain/privkey1.pem;
}
```

删除 nginx 容器，以新命令启动，注意映射 443 端口。

大功告成。

## 获取证书有效期

```shell
certbot certificates
```

## 参考文献

1. [Let's Encrypt 安装配置教程，免费的 SSL 证书](https://zhuanlan.zhihu.com/p/196638669)
2. [Let's Encrypt 官网](https://letsencrypt.org/zh-cn/)
3. [acme 说明](https://github.com/acmesh-official/acme.sh/wiki/%E8%AF%B4%E6%98%8E)
4. [acme 中国大陆御用库](https://gitee.com/neilpang/acme.sh)
5. [acme How to install](https://github.com/acmesh-official/acme.sh/wiki/How-to-install)
6. [How to issue a cert](https://github.com/acmesh-official/acme.sh/wiki/How-to-issue-a-cert)
7. [acme.sh简单教程](https://blog.csdn.net/Dancen/article/details/121044863)
8. [certbot instructions other on centos7](https://certbot.eff.org/instructions?ws=other&os=centosrhel7)
