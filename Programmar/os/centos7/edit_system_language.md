# CentOS7如何修改系统语言

`vim /etc/locale.conf`

修改`LANG`字段对应的值
1. 中文: zh_CN.UTF-8
2. 英文: en_US.UTF-8


## 实际上修改了什么？

此番修改实际上修改了系统向用户终端输出字符时使用的字符编码，如果用户终端不支持系统发送的编码，则会由于编码不对应出现乱码。

修改完之后reboot一下系统以生效