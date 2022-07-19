---
title: CentOS 7 用户/用户组的添加与删除
date: 2021-06-20 17:54:56.011
updated: 2021-06-20 18:16:03.996
url: http://summersea.top:8090/archives/centos7用户用户组的添加与删除
categories: 
tags: CentOS 7 | 用户管理
---

**注意：要在root用户下使用**
1. 新建用户 
```bash
adduser testuser # 新建testuser 用户 
passwd testuser  # 给testuser 用户设置密码
```
2. 新建工作组 
```bash
groupadd testgroup # 新建test工作组
```
3. 新建用户同时增加工作组 
```bash
useradd -g testgroup testuser # 新建testuser用户并增加到testgroup工作组
```

**注：**-g 所属组 -d 家目录 -s 所用的SHELL

4. 给已有的用户增加工作组 
```bash
usermod -G groupname username
```
5. 临时关闭 
在/etc/shadow文件中找到目标用户的行的第二个字段（密码），前面加上`!`就可以了。想恢复该用户，去掉即可
推荐如下命令关闭用户账号（本质和上方操作一样）： 
```bash
passwd testuser –l 
```
重新释放： 
```bash
passwd testuser –u
```

6. 永久性删除用户账号/用户组 
```bash
userdel testuser 
groupdel testgroup 
usermod –G testgroup testuser #（强制删除该用户的主目录和主目录下的所有文件和子目录）
```

7. 显示用户信息
```bash
id user 
cat /etc/passwd
```
`/etc/passwd`部分段落
![image.png](http://summersea.top:8090/upload/2021/06/image-c056394746b64b0a87df477c56cd14b5.png)
> 从上面的例子我们可以看到，/etc/passwd中一行记录对应着一个用户，每行记录又被冒号(:)分隔为7个字段，其格式和具体含义如下：
**用户名:口令:用户标识号:组标识号:注释性描述:主目录:登录Shell**

**补充**：查看用户和用户组的方法 
用户列表文件：`/etc/passwd`
用户组列表文件：`/etc/group`
查看系统中有哪些用户：`cut -d : -f 1 /etc/passwd`
查看可以登录系统的用户：`cat /etc/passwd | grep -v /sbin/nologin | cut -d : -f 1`
查看用户操作：`w`命令(需要root权限) 
查看某一用户：`w 用户名` 
查看登录用户：`who` 
查看用户登录历史记录：`last`

### 参考文献
[CentOS7添加/删除用户和用户组](https://www.cnblogs.com/gucb/p/11843319.html)
博客都抄来抄去的(￣_￣|||)，还都不看。
[在Linux系统里如何禁用一个用户帐号](https://www.xp.cn/b.php/4610.html)
[Linux 下/etc/passwd文件详解](https://blog.csdn.net/a1154490629/article/details/52190801)