---
title: apache httpclient请求url经过url编码后出现broken pipe或Timeout waiting for connection from pool异常
date: 2021-09-26 16:01:45.296
updated: 2021-09-26 16:01:45.296
url: http://summersea.top:8090/archives/apachehttpclient-qing-qiu-url-jing-guo-url-bian-ma-hou-chu-xian-brokenpipe-huo-timeoutwaitingforconnectionfrompool-yi-chang
categories: 
tags: Java
---

由于工作匆忙所以此文只是简单记录。

根因是初始化httpclient的方式不对。

解决了问题的初始化方式
![企业微信截图_16326432236819.png](http://summersea.top:8090/upload/2021/09/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16326432236819-2308e09c2c2149f98bd50474634e8781.png)

出问题的初始化方式：
![企业微信截图_16326432713149.png](http://summersea.top:8090/upload/2021/09/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16326432713149-5c19dcb3a8844d689d3f53008a68a331.png)