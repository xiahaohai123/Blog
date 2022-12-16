# ChaosBlade 中让某个方法的执行产生延迟

## 正文

注意 blade 程序所在位置不对可能会引发奇怪的问题，笔者放在了 `/tmp` 目录下。

创建一个延迟注入

```shell
$ ./balde create jvm delay --classname=com.example.controller.DubboController --methodname=sayHello --time 10000
```

成功执行后可以获取到一个 uid

`{"code":200,"success":true,"result":"<uid>"}`

我们可以用 destroy 命令来取消掉这个注入。

```shell
$ ./blade destroy <uid>
```