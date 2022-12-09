# 前端部署生产环境操作过程记录

## 前端打包

执行 `npm run build` 命令以进行打包。

- 遇到 `TS6504` 问题时可以如何解决:
  > error TS6504: File 'D:/workspace/filetransfer/filetranweb/src/components/PeerItem.vue.jsx' is a JavaScript file. Did you mean to enable the 'allowJs' option? The file is in the program because:
  Matched by include pattern 'src/** / *' in 'tsconfig.vitest.json'

- 进入对应的 `tsconfig.vitest.json` 文件，在 `compilerOptions` 中加入 `"allowJs": true`
  ```json 
  {
    "compilerOptions": {
        "composite": true,
        "lib": [],
        "types": ["node", "jsdom"],
        "allowJs": true 
    } 
  }
  ```
- 这是因为我的某些 vue 组件仍然在使用 js 语法，而 ts 编译过程中没有允许使用 js 语法。

## nginx 部署

为了确保以后都能快速重新部署，这里采用 docker 的部署形式。

1. `docker pull nginx` 以下载 nginx 的镜像。
2. 准备自己的 [nginx.conf](nginx.conf) 文件
3. `docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro -d -p 80:80 nginx`
    - 注意，执行这个步骤前需要先保证 `nginx.conf` 文件已经准备好了。
4. 重启 docker 容器或者在容器内执行 `nginx -s reload`

这样我们就可以直接访问我们的页面了。

## 排错之路

### 资源串文件 404
由于前端的代码设计中，资源串文件是动态加载的，仍然会以打包前的目录去下载资源串，但打包完毕后对应路径下没有资源串文件，导致了404。

需要一个方案，打包的时候把资源串文件拷贝到包内，让客户端能下载到资源串文件。



## 参考文献

1. [nginx-docker_hub](https://hub.docker.com/_/nginx)