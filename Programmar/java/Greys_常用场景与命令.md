# Greys_常用场景与命令

## 检查方法的入参与出参

1. ```shell
    tt -t *.<className> <methodName>
   ```
   
    这样执行后就会开始监听对应的方法执行过程了，会输出一张表格，每次调用都会输出，每次调用对应一个 id。
2. ```shell
   tt -i <id> -x 2
   ```

    查看之前监听的方法入参出参等其他信息详情
   
## 耗时检测

```shell
    trace *.<className> <methodName>
```

该命令可以检测一个方法调用后使用了多少时间才响应，并会给出子方法的时间。