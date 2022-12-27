# Gradle 构建中文输出乱码

## 解决方法

1. 调整 gradle 输出编码为 utf-8，在 build.gralde 内加入如下信息
```groovy
tasks.withType(JavaCompile) {
    options.encoding = 'UTF-8'
}
```

2. 调整 idea 输出为 utf-8

Help --> Edit Custom VM Options

加入如下信息

```properties
-Dfile.encoding=UTF-8
```

3. 重新启动 idea

## 参考资料

1. [gradle项目控制台中文乱码](https://blog.csdn.net/weixin_46196153/article/details/125907615)