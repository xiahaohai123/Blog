# Jenkins构建升级包demo的设计与实践

## 前言

本篇为第二个demo的设计与实践。

Q：为什么会有第二个demo？

A：因为第一个demo的架构设计比较复杂，实现起来需要时间，而笔者在打包地狱中最缺的就是时间。所以在第一个demo完成前，先设计与实践第二个demo，这个demo可以立即投入使用，减少笔者在打包工作上的投入，该demo的要求比第一个要简单的多，功能和第一个demo的侧重点不同，为升级包的构建与部署。两个demo合并后可以成为自动构建系统的原型。

## demo要求

1. 升级包的构建为手动构建
2. 升级包的构建逻辑依赖于Jenkinsfile
3. rpm构建使用公司的jenkins进行构建
4. rpm版本号修改脚本化（优先级低）

## 升级包目录设计

```
|-d- bak
|-d- check_scripts
   |-f- prebackup.sh
   |-f- precheck.sh
|-d- degrade
   |-d- official
      |-d- rpm
      |-d- script
   |-d- lib
      |-d- rpm
   |-f- md5
|-d- upgrade
   |-d- official
      |-d- rpm
      |-d- script
   |-d- lib
      |-d- rpm
|-f- upgrade
|-f- upgrade.log
```

## 构建流程设计

1. 准备构建
    1. 确定需要构建哪些安装包
    2. 确定安装包对应的旧版安装包
2. 构建RPM
    1. 使用API构建RPM(parallel实现)
    2. 将RPM下载到本地工作空间中的第一方RPM目录(upgrade/official/rpm)
3. 旧版RPM获取
    1. 根据**准备构建**步骤准备的信息，从最新版本开始下载升级包，解压并查找对应的RPM
    2. 将旧版本RPM放入本地工作空间的第一方RPM目录(degrade/official/rpm)
4. 获取新脚本文件(低优先级)
5. 获取旧脚本文件(低优先级)
6. 第三方依赖(低优先级)
7. 组装升级包
    1. demo打通过程中，暂时使用过去的upgrade脚本模板进行升级。
    2. 该demo打通后的第二阶段，upgrade应当来自于各个信息的组装。
8. 自动部署到目标机器(低优先级)

## 问题列表

将目前能考虑的问题拆分列于此处，构成一个有方向的开发文档。

1. jenkins的docker容器内没有wget命令，需要一个下载通道，下载公司jenkins构建出来的rpm包。
    1. 鉴于docker容器内有curl命令，使用curl命令替代。
2. jenkins该如何计算出对应的旧版本rpm包并获取呢？
    1. 维护一个存放过去版本升级包的仓库，如果没有升级包则下载，不删除，通过缓存复用来减少下载时间对构建的影响。
    2. 定义一个升级包顺序表，从最新的版本往旧版本遍历，如果找到了对应的旧版本rpm安装包则取用。

## 参考文档

1. [Jenkins自动化流水线demo的设计与实践](./build_upgrade_package_automatically_design.md)