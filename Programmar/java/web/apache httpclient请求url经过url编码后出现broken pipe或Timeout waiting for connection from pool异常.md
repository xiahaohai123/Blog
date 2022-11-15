# apache httpclient请求url经过url编码后出现broken pipe或Timeout waiting for connection from pool异常

由于工作匆忙所以此文只是简单记录。

根因是初始化httpclient的方式不对。

解决了问题的初始化方式

- ![img.png](../assets/screenshot05.png)

出问题的初始化方式：

- ![img.png](../assets/screenshot06.png)