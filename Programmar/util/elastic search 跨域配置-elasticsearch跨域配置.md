---
title: elastic search 跨域配置
date: 2022-04-24 14:45:58.274
updated: 2022-04-24 14:46:08.793
url: http://summersea.top:8090/archives/elasticsearch跨域配置
categories: 
tags: 
---


# 背景

研发中经常需要使用第三方工具快速查询`es`内的数据，常用的插件有`es-head`。每次使用该插件都要求`es`允许跨域请求，每次为此查询有些浪费时间，在此记录。

# 操作

1. 在文件`/etc/elasticsearch/elasticsearch.yml`中添加以下信息：
```yml
http.cors.enabled: true
http.cors.allow-origin: "*"
```

2. 重启es