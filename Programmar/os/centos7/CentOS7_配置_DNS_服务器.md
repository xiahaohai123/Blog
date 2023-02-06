# CentOS7 配置 DNS 服务器

## 正文

修改 resolv.conf 文件

```shell
vim /etc/resolv.conf
```

添加

```shell
#主DNS服务器
nameserver 8.8.8.8

#备DNS服务器
nameserver 8.8.8.8
```

重启 NetworkManager

```shell
systemctl restart NetworkManager
```

## 参考文献

1. [Centos7配置IP地址和DNS](https://blog.csdn.net/teisite/article/details/81196244)