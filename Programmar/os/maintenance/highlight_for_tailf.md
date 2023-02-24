# 高亮tail -f命令输出的关键字内容

## 单个关键词高亮显示

```shell
tail -f 日志文件 | perl -pe 's/(关键词)/\e[1;颜色$1\e[0m/g'
```

```shell
tail -f catalina.out | perl -pe 's/(DEBUG)/\e[1;34m$1\e[0m/g'
```

## 多个关键词高亮显示

```shell
tail -f catalina.out | perl -pe 's/(关键词1)|(关键词2)|(关键词3)/\e[1;颜色1$1\e[0m\e[1;颜色2$2\e[0m\e[1;颜色3$3\e[0m/g' 
tail -f catalina.out | perl -pe 's/(DEBUG)|(INFO)|(ERROR)/\e[1;34m$1\e[0m\e[1;33m$2\e[0m\e[1;31m$3\e[0m/g' 
```

## 字体颜色

30-37 黑、红、绿、黄、蓝、紫、青、白

- 30m：黑
- 31m：红
- 32m：绿
- 33m：黄
- 34m：蓝
- 35m：紫
- 36m：青
- 37m：白

## 背景颜色设置

40-47 黑、红、绿、黄、蓝、紫、青、白

- 40：黑
- 41：红
- 42：绿
- 43：黄
- 44：蓝
- 45：紫
- 46：青
- 47：白

## 其他参数说明

- [1; 设置高亮加粗
- [4; 下划线
- [5; 闪烁

例子： 黄字，高亮加粗显示
[1;33m 红底黄字，高亮加粗显示
[1;41;33m

## 注意

该命令仅针对标准输出流有效，如果需要高亮错误输出流，需要将错误输出流重定向到标准输出流，比如:

```shell
ssh -vvv 10.10.10.10 2>&1 | perl -pe 's/(peer)|(host key)|(ERROR)/\e[1;34m$1\e[0m\e[1;33m$2\e[0m\e[1;31m$3\e[0m/g' 
```

## 参考引用

1. [Linux基础命令之tail动态显示日志文件时关键字有颜色、高亮显示](https://blog.csdn.net/Soinice/article/details/96284534)
1. [【小而优】 如何实现 tail -f 动态显示日志时高亮显示关键字](https://www.cnblogs.com/Detector/p/7246377.html)