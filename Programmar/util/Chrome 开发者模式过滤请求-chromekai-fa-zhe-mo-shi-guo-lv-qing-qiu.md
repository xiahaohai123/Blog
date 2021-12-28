---
title: Chrome 开发者模式过滤请求
date: 2021-06-22 10:22:49.996
updated: 2021-06-22 10:26:24.372
url: http://summersea.top:8090/archives/chromekai-fa-zhe-mo-shi-guo-lv-qing-qiu
categories: 
tags: 调试 | Chrome | 请求
---

### 原因
当我们初次接触一个网站系统且用开发者模式跟踪网络请求信息时，经常会发现有大量.png/.css/.js请求。通常情况下我们更需要关注前后端交互的请求，然而这些少数的请求往往会被淹没在资源请求里，所以我们需要用过滤器过滤掉不想要的请求。

### 直接过滤
1. 可以在输入框里直接输入文字来过滤，此时过滤基于文字与请求信息的匹配（不仅仅匹配名字）
![image.png](http://summersea.top:8090/upload/2021/06/image-311e9fa062d04d6d90d813991e2e6f0e.png)
2. 输入框右边列出了大多数请求类型，可以通过点击该类型来过滤出对应类型的请求。
![image.png](http://summersea.top:8090/upload/2021/06/image-fc1a0fa77ec44282b1061f9a399d8c17.png)

### 条件过滤
 1. 在输入框内输入`-`，Chrome会弹出各种过滤条件，我们可以按这些条件进行过滤。
![image.png](http://summersea.top:8090/upload/2021/06/image-f6aca683395842f88aad9d018bfa11ff.png)
在过滤方法后面输入`:`后Chrome也会弹出输入建议，最终可以按条件得出过滤结果。
![image.png](http://summersea.top:8090/upload/2021/06/image-5b6060867dbf46569637ff2ca806fcbb.png)

### 反向过滤
- 有时候会想要过滤掉不想看见的请求，这个时候只要输入`-`加要匹配的文字即可，此时匹配上的请求都会被过滤掉。
![image.png](http://summersea.top:8090/upload/2021/06/image-75801be1e5ad4e1184e2fc3803b5443d.png)
- 可以多条过滤，过滤条件中间加个空格分隔即可，比如`-.js -.css`。