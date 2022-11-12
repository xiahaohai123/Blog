# 使用nginx快速搭建一个http文件服务器

## 前言

在有些场景下我们需要将文件发布出去，给其他人使用，这个时候将文件放在文件服务器上让人们进行下载是一个很好的方案，所以这里记录了怎么快速搭建一个http文件服务器。

## 正文

1. 首先安装nginx，笔者使用了docker。
2. 创建文件仓库目录`/home/user/filerepos`
3. 配置/etc/nginx/nginx.conf
```
http {
    # ignore other config
    
    server {
        listen 80;
        server_name localhost;
            location / {
                root /home/user/filerepos; #指定哪个目录作为Http文件服务器的根目录
                autoindex on; #设置允许列出整个目录
                autoindex_exact_size off; #默认为on，显示出文件的确切大小，单位是bytes。改为off后，显示出文件的大概大小，单位是kB或者MB或者GB
                autoindex_localtime on; #默认为off，显示的文件时间为GMT时间。改为on后，显示的文件时间为文件的服务器时间
                charset utf-8; #防止文件乱码显示, 如果用utf-8还是乱码，就改成gbk试试
            }
    }
    # include /etc/nginx/conf.d/*.conf;
}
```
4. 重启或重新刷新配置
```shell
nginx -s reload
```

## 参考文献

[使用Nginx搭建Http文件服务器](https://blog.csdn.net/witty_ming/article/details/124282231)