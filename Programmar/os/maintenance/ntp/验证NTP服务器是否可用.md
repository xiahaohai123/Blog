# 验证NTP服务器是否可用

## 前言

遇到一个场景，配置 NTP 服务器时需要先验证服务器是否可用，该如何完成验证这个步骤？

## ntpdate -d

`ntpdate` 命令本身用以从目标 NTP 服务器同步时间。

而 `ntpdate -d` 命令可以以 debug 模式运行一遍，而不修改本地时间。

> -d    
> Enable the debugging mode, in which ntpdate will go through all the  steps, but
> not  adjust  the local clock. Information useful for general debugging will also
> be printed.

这意味着 `ntpdate -d` 可以完成一整套同步动作，并给出反馈，好让我们确定目标服务器是否可以正常提供服务，且不修改本地时钟。完美契合我们的场景，问题就是机器上是否一定存在 `ntpdate` 命令。


### 如何判断
这里由于笔者的研发环境无法复制文字出来，所以使用其他人的执行结果。

```shell
[root@centos6 ~]# ntpdate -d ntp1.aliyun.com
27 Mar 22:04:38 ntpdate[1049]: ntpdate 4.2.6p5@1.2349-o Wed Dec 19 20:22:35 UTC 2018 (1)
Looking for host ntp1.aliyun.com and service ntp
host found : 120.25.115.20
transmit(120.25.115.20)
receive(120.25.115.20)
transmit(120.25.115.20)
receive(120.25.115.20)
transmit(120.25.115.20)
receive(120.25.115.20)
transmit(120.25.115.20)
receive(120.25.115.20)
server 120.25.115.20, port 123
stratum 2, precision -25, leap 00, trust 000
refid [120.25.115.20], delay 0.07951, dispersion 0.00424
transmitted 4, in filter 4
reference time:    e046016e.965c7e00  Wed, Mar 27 2019 22:04:30.587
originate timestamp: e0460177.12bce60a  Wed, Mar 27 2019 22:04:39.073
transmit timestamp:  e0460177.04f47dc5  Wed, Mar 27 2019 22:04:39.019
filter delay:  0.09312  0.07951  0.08475  0.09390 
         0.00000  0.00000  0.00000  0.00000 
filter offset: 0.004349 0.010768 0.007720 0.019682
         0.000000 0.000000 0.000000 0.000000
delay 0.07951, dispersion 0.00424
offset 0.010768
27 Mar 22:04:39 ntpdate[1049]: adjust time server 120.25.115.20 offset 0.010768 sec
```

而无法提供服务的服务器在 debug 时没有 receive 关键字，我们可以判断是否存在 receive 关键字来判断目标服务器是否可以提供服务。

具体命令如下:

```shell
ntpdate -d -p 1 10.10.10.10 | grep receive | wc -l
```

当返回结果大于 0 时，我们可以认为目标服务器可以正常提供服务。

-p 1 表示仅收发 1 个包，以此加快验证速度，默认值是 4，不适合验证场景。

## 参考文献

1. [Linux环境下如何验证提供时间校准的NTP服务器是否可用](http://blog.itpub.net/2317695/viewspace-2639523)