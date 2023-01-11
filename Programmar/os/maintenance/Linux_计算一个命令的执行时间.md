# Linux_计算一个命令的执行时间

## 正文

使用 `time` 命令可以计算一个命令的执行时间

```shell
$ time du -a / 2> /dev/null | wc -l
```

可以获得响应:

```shell
$ real    0m5.00s
$ user    0m0.00s
$ sys     0m0.00s
```

- `real` 表示总占用时间
- `user` 表示用户占用时间
- `sys` 表示系统占用时间