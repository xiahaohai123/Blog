# 用 svg 做一个圆形进度条

## 前言

在自己的项目中，传输文件时需要一个圆形进度条，找了很多实现方式，最终选择了 svg 作为接下来要用的方案。

## 为什么选择 svg

我看到的其他方案都是基于一些基础形状**拼装裁剪**而成的，没有一种浑然天成的感觉，在不断地寻找中，终于碰见了 svg。并且搜索后发现 svg 能做动画。

svg 的方案与其他我见到的方案存在的不同点在于，svg 是从 0 开始绘画，这也符合我的期望，成品**浑然天成**。

为什么我会拘泥于**浑然天成**，因为我认为基于**直线思维**构建的代码阅读门槛会低一些，也就是说这样会易于维护。当然这是忽略了 svg 上手门槛的情况下。但是没有关系，因为这个项目是我的个人项目，没有学习门槛的问题。

## 参考文献

1. [SVG 路径动画-圆形高光进度条](https://www.jianshu.com/p/73dd45f04779)
2. [SVG 路径动画](https://blog.csdn.net/weixin_43866528/article/details/120224383)
3. [SVG 教程-菜鸟教程](https://www.runoob.com/svg/svg-tutorial.html)
4. [SVG 路径-菜鸟教程](https://www.runoob.com/svg/svg-path.html)
5. [SVG：可缩放矢量图形-MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG)
6. [SVG 教程-MDN](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Tutorial)