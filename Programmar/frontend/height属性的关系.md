---
title: HTML5前端canvas标签的行内属性width/height与CSS样式width/height属性的关系
date: 2021-06-22 15:25:52.068
updated: 2021-06-28 20:27:55.219
url: http://summersea.top:8090/archives/html5-qian-duan-canvas-biao-qian-de-xing-nei-shu-xing-widthheight-yu-css-yang-shi-widthheight-shu-xing-de-guan-xi
categories: 
tags: HTML5
---

### 图解
**注意，该图解仅为笔者的理解，非官方图解**
![aPage3 1.png](http://summersea.top:8090/upload/2021/06/a-Page-3%20(1)-ff7783ee8029488f9104a60007549f8e.png)!
基本可以理解为：
1. canvas先根据行内属性的width（默认300）和height（默认150）的参数确定了坐标系范围。
2. 前端使用context.fillRect(x,y,width,height)在坐标系上进行绘画（此处省略绘画前的准备步骤）。
3. 画好的图根据CSS样式的width和height参数进行等比例缩放。
4. 得到最终成像。

### 参考文献
[canvas 的行内属性width、height与css的width、height属性差别](https://blog.csdn.net/xiongshiyuan/article/details/85223867)