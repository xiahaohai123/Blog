# rabbitmq_tracing 插件的使用

## 作用

该插件可以追踪 RabbitMQ 的消息流入流出情况。

## 启用\关闭插件

开启插件

```shell
[root@node3 opt]# rabbitmq-plugins enable rabbitmq_tracing
The following plugins have been enabled:
  rabbitmq_tracing

Applying plugin configuration to rabbit@node3... started 1 plugin.
```

关闭插件
`rabbitmq-plugins disable rabbitmq_tracing`

## 用户入口

1. 进入Web管理页面，进入 Admin 标签，右侧会出现 Tracing 选项。
2. 在 Tracing 页面添加一个新的 trace 后，就会发现对应的日志在搜集消息的流入流出。
3. 点击日志文件即可查看对应日志。

## 参考文献

1. [RabbitMQ消息追踪之rabbitmq_tracing](https://blog.csdn.net/u013256816/article/details/76039201)
