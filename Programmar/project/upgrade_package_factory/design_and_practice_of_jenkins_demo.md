# Jenkins自动化流水线demo的设计与实践

## 前言
本文档派生自[自动打升级包系统设计](./build_upgrade_package_automatically_design.md)，因为用jenkins来落地看起来比较现实，所以决定学习jenkins，通过实践一个demo让我进行成长，并积累经验，开拓视野，以尽量完美的品质完成自动打升级包系统落地的目标。

## demo设计
既然学习jenkins是来源于一个最终目标，那么demo设计成与目标高度相关的产品也比较好。一来这样可以节省一部分从头开工的时间，二来这样实践下来更容易找出目前的方案哪里还有问题，需要怎么解决。

所以从顶层设计或功能范围来说，这个demo需要完成以下目标：
1. 完成流水线管道`pipeline-build-rpm`，用以构建第一方rpm
    1. 这个rpm包只需要参照公司已有的rpm构建方式进行构建。
    2. 不要求对rpm包进行签名。
    3. 完成至少两个管道。
    4. 对对应的代码库具备监测更新并触发自动构建能力。
2. 完成流水线管道`pipeline-build-upgrade`: 使用pipeline-build-rpm的产品组件出一个升级包
    1. 不需要满足可回滚特特性。
    2. 不需要满足增量升级特性。
    3. 构建产品需要满足一键安装要求。
3. 任意一条`pipeline-build-rpm`构建完成后需要触发`pipeline-build-upgrade`。