# 执行rabbitmq-diagnostics相关命令出现Authentication failed

## 参考资料
https://www.freesion.com/article/4415632118/

## 现象与解决

现象为 Authentication failed (rejected by the remote node), please check the erlang cookie。

1. 在linux环境中，执行`updatedb`命令后再`locate .erlang.cookie`
2. 在`/root`目录下和`/var/lib/rabbitmq`目录下找到了这两份文件。
3. 使用`md5sum`命令计算两个文件的摘要进行比对，发现不一样。
4. 把`/var/lib/rabbitmq`目录下的文件覆盖拷贝到`/root`目录下。
5. 再次执行相关命令，成功执行。

