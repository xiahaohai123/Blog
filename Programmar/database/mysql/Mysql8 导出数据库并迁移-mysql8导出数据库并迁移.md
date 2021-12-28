---
title: Mysql8 导出数据库并迁移
date: 2021-06-20 16:18:30.755
updated: 2021-06-20 16:53:52.15
url: http://summersea.top:8090/archives/mysql8导出数据库并迁移
categories: 
tags: MySQL 8
---

### 参考文献
[MySQL之mysqldump的使用](https://www.cnblogs.com/markLogZhu/p/11398028.html)

> ### 1. mysqldump简介 
>`mysqldump` 是 MySQL 自带的逻辑备份工具。
它的备份原理是通过协议连接到 MySQL 数据库，将需要备份的数据查询出来，将查询出的数据转换成对应的`insert`语句，当我们需要还原这些数据时，只要执行这些`insert`语句，即可将对应的数据还原。
> ### 2. 备份命令
> #### 2.1 命令格式
> `mysqldump [选项] 数据库名 [表名] > 脚本文件名`
`mysqldump [选项] --数据库名 [选项 表名] > 脚本名`
备份所有数据库
`mysqldump [选项] --all-databases [选项]  > 脚本名`
> #### 2.2选项说明
|参数名|缩写|含义|
|:---:|:--:|:--:|
|-\-host|-h|服务器IP地址|
|-\-port|-P|服务器端口号|
|-\-user|-u|MySQL 用户名|
|-\-pasword|-p|MySQL 密码|
|-\-databases||指定要备份的数据库|
|-\-all-databases||备份mysql服务器上的所有数据库|
|-\-compact||压缩模式，产生更少的输出|
|-\-comments||添加注释信息|
|-\-complete-insert||输出完成的插入语句|
|-\-lock-tables||备份前，锁定所有数据库表|
|-\-no-create-db/--no-create-info||禁止生成创建数据库语句|
|-\-force||当出现错误时仍然继续备份操作|
|-\-default-character-set||指定默认字符集|
|-\-add-locks||备份数据库表时锁定数据库表|
> ### 3. 还原命令
> #### 3.1 命令行命令
```bash
mysqladmin -uroot -p create db_name 
mysql -uroot -p  db_name < /backup/mysqldump/db_name.db
```
> 注：在导入备份数据库前，db_name如果没有，是需要创建的； 而且与db_name.db中数据库名是一样的才可以导入。
> #### 3.2 mysql内source命令
```sql
mysql > use db_name
mysql > source /backup/mysqldump/db_name.db
```

## 实战
1. 导出备份文件
```bash
mysqldump -uroot -p key_xxxx > /root/key_xxxx.db
```
ll命令查看/root目录下可以发现已经导出了db文件

2. 将该文件转移到
这里使用FileZilla软件帮助文件迁移

3. 复原数据
在新的环境上进入mysql
输入以下命令:
```sql
> CREATE DATABASE IF NOT EXISTS key_xxxx DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
> SOURCE /root/key_xxxx.db
```
至此，数据库迁移成功。
