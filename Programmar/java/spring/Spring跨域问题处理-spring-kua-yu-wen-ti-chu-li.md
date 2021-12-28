---
title: Spring跨域问题处理
date: 2021-11-19 17:57:48.441
updated: 2021-11-19 17:57:48.441
url: http://summersea.top:8090/archives/spring-kua-yu-wen-ti-chu-li
categories: 
tags: 互联网
---

### 参考资料
[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)


### 背景

在工作中遇到了跨域请求问题，为了解决这个问题，按照同事的说法使用了`@CrossOrigin`注解，解决问题后又遇到了新的跨域问题，花费了较长时间才解决，所以在此记录一下。

## 业务场景

整个产品作为集群部署的时候，用户想要下载文件可能处于其他节点上，此时需要在用户访问的节点网页向目标节点发起下载请求以下载文件。

## 故事过程

一开始采用的下载方式是`window.location.href`，但是后来发现这种下载方式无法进行错误处理，当目标节点上也不存在文件的时候，会返回http code 404，最终导致页面空白，用户体验极差，所以想要尝试使用`ajax`进行错误处理。

使用`ajax`请求后就遭遇了跨域问题，按照同事的解决办法，在后端对应的请求方法上新增了`@CrossOrigin`注解，问题解决。

但是新的问题出现了，刚才发现解决问题的原因是，仅测试了404的场景，但文件可以下载的场景仍然有跨域问题。

思来想去最终想起来，下载文件的过程直接使用了`HttpServletReponse`，使用这个类后`spring`是无法控制的，所以下载文件的场景仅仅是`@CrossOrigin`注解是不够的。

最终的解决办法是能下载文件时手动增加允许跨域的`header`。

![企业微信截图_16373157761950.png](http://summersea.top:8090/upload/2021/11/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16373157761950-8df812005b574667a02aa5341ead3390.png)

![企业微信截图_16373158036201.png](http://summersea.top:8090/upload/2021/11/%E4%BC%81%E4%B8%9A%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_16373158036201-26d411abaa584cada8b88fb795dc1424.png)