# TypeScript 前端项目搭建

## 前言

本文记录搭建 TypeScript 前端项目流程，作为记录为后续其他项目搭建铺路。

## 正文

### Node.js

node.js 可以去 [官方网站下载](https://nodejs.org/zh-cn/) 。下载后安装即可。

### 项目搭建

#### 1. 创建项目根目录

```shell
mkdir learnfp
```

#### 2. 初始化 npm 项目

进入刚刚创建的目录后，执行命令以初始化 npm 项目。

```shell
cd learnfp
npm init
```

执行命令后会让我们填写相关信息，根据需求填写即可，也可以一路回车。

最终输入 yes 以完成初始化。

然后就可以使用 IDE 打开这个项目了。

#### 3. 项目引入 typescript

```shell
npm install --save-dev typescript @types/node
```

#### 4. 新建一个tsconfig.json

项目根目录新建一个文件 tsconfig.json

```json5
{
  "compilerOptions": {
    //假定运行环境的API有哪些
    "lib": ["es2015"],
    //把代码编程成哪个模块系统
    "module": "commonjs",
    //生成的JavaScript代码存放到哪个文件夹中
    "outDir": "dist",
    //开启sourcemap方便调试
    "sourceMap": true,
    //检查无效代码，强制所有代码都应该正确声明了类型
    "strict": true,
    //把代码编译成哪个JavaScript版本
    "target": "es2015"
  },
  //  在哪个文件夹寻找typescript文件
  "include": [
    "src"
  ]
}
```

#### 5. 新建一个 index.ts

在项目根目录新建一个代码资源目录 src

再在 src 目录下新建一个 index.ts 文件

文件中敲入一些代码

```typescript
console.log('Hello TypeScript!');
```

这样最基础的 typescript 项目就建好了。

#### 6. 运行项目

webstorm 编译一下 index.ts，编译完成后可以在项目根目录的 dist 目录下看到 index.js 文件。

进入 index.js 文件，右键运行文件即可。