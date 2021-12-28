---
title: 使用XmlHttpRequest2下载大文件
date: 2021-11-21 15:12:47.099
updated: 2021-11-22 09:54:24.066
url: http://summersea.top:8090/archives/使用xmlhttprequest2下载大文件
categories: 
tags: JavaScript | 网络
---


## 背景

工作中遇到了一个场景，A国的用户要从B国的服务器下载文件，中间会经过广域网，广域网的网络资源成为了必须要考虑的问题。


## 期望

- 找到一种办法，可以使用`javaScript`下载文件，又可以同时对文件不存在等情况进行错误处理，并告知用户相关信息，以保证用户体验。
- 上述行为要在一个`http`请求内完成，以节省广域网资源。


## 过程

### 各个方案排雷过程
- 最开始参照公司的代码使用了`window.location.href = url`进行下载，一切都运行的很美好——在我提供的完美环境下。直到试了下删除后台的文件，再下载就直接跳转到了404页面。我一脸懵逼，那用户用起来不也得跟傻了似的？这不行，肯定要做**错误处理**。


- 错误处理嘛，简单，让`js`来！由于用了`AngularJS`框架，而且我不会，所以先排除`axios`、`Angular`的内置请求框架方案。

- 刚开始我很自信的使用了`ajax`请求服务端的文件，结果显而易见，`ajax`竟然**不处理流数据**！等到进入了`success回调`后，**数据都已经在内存里了**，虽然我们可以在`success回调`里解析数据并存储成一份文件，但是这个解决方案无法应对巨大文件，我相信一般人的PC是不会有几十个甚至几百个G的内存空间的。

- 即使一直受到挫折打击，但是理论上是一定存在一种办法，**能在一个请求内进行错误处理或文件下载**的。最终，我将注意力集中到更底层的`XmlHttpRequest`上。于是便开始了`XHR`的学习之路。


### 解决过程

- 刚开始动手试了试`XHR`请求，不难，但是下载文件时出现了与使用`ajax`相同的问题，到了`readystate`=4的时候内存已经被大量占用了。

- 后来看到了阮一峰的博客，发现`responseType`是可以指定为`blob`的，试了试，下载2G文件的时候盯着任务管理器疯狂F5刷新（也正是这次我才知道任务管理器可以按F5），发现内存变化不大，这时候问题基本已经宣告解决了，剩下的便是收尾。从任务管理器的信息来看，磁盘读写占用飙升了，相较于内存占用，磁盘读写的占用更经济，另外也能反推出，使用`blob`时**实际上是下载了一份文件到磁盘**，那我们接下来的任务就是把这份文件存成目标文件名即可。

- 存储文件的方式是使用了浏览器下载文件的自带代码，我们只要创建一个链接连向下载文件时受到的`blob`对象，再点击这个链接即可。

### 示例前端代码
``` javascript
var xhr = new XMLHttpRequest()
xhr.open("GET", url, true)
xhr.responseType = "blob"  // 核心
xhr.onload = function() {
    if (xhr.status === 404) {
        // 404处理
    } else if (xhr.status === 200) {
        var blob = xhr.response
        var localDownloadUrl = URL.createObjectURL(blob)
        var link = document.createElement('a')
        link.download = filename // 下载的文件名
        link.href = localDownloadUrl
        link.click()
    }
}
xhr.send()
```

### 遗憾
- 这个解决方式实际上是先将数据下载到磁盘，再将数据从磁盘上的这个文件搬移到目标位置。

- 而最理想的方式应当像`window.location.href = url`的实现一样，直接下载到目标位置。

- 可惜目前没能找到这个完美的办法，而笔者本身的技术栈偏向后端，所以研究到此便停止了（再找下去可能要触发边际递减效应了，对笔者的收益不高）。

- 这个解决办法可能会出现一些用户体验的问题，即用户很久才会看到下方有下载进度框，所以需要在前端耍一些把戏以优化用户体验问题。

## 参考资料
[XMLHttpRequest Level 2 使用指南](https://www.ruanyifeng.com/blog/2012/09/xmlhttprequest_level_2.html)
[原生JS使用Blob导出csv文件](https://www.cnblogs.com/moluy/p/14096490.html)
[URL.createObjectURL()](https://developer.mozilla.org/zh-CN/docs/Web/API/URL/createObjectURL)