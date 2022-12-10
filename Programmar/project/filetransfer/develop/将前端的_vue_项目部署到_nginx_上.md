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

## Nginx 部署

为了确保以后都能快速重新部署，这里采用 docker 的部署形式。

1. `docker pull nginx` 以下载 Nginx 的镜像。
2. 准备自己的 [nginx.conf](nginx.conf) 文件
3. `docker run --name some-nginx -v /some/content:/usr/share/nginx/html:ro -v /etc/nginx/nginx.conf:/etc/nginx/nginx.conf:ro -d -p 80:80 nginx`
    - 注意，执行这个步骤前需要先保证 `nginx.conf` 文件已经准备好了。
4. 重启 docker 容器或者在容器内执行 `nginx -s reload`

这样我们就可以直接访问我们的页面了。

## 排错之路

### 资源串文件 404

由于前端的代码设计中，资源串文件是动态加载的，仍然会以打包前的目录去下载资源串，但打包完毕后对应路径下没有资源串文件，导致了404。

1. 需要一个方案，打包的时候把资源串文件拷贝到包内，让客户端能下载到资源串文件。
    - 但是手动将资源文件放到包内后，发现不起作用，能下载，但是不能加载。
2. 那么换一个思路，当前的动态加载实现方式是动态生成资源路径再加载，所以编译器没有识别这部分文件，那么将资源路径换成静态的，不就可以编译进去了么？
    - 原始代码
        ```typescript
        (async function () {
          const localeResourcePath: string = getLocaleResource();
          const localeResModule = await import(localeResourcePath);
          app.use(i18nPlugins, localeResModule.resource);
        })();
        
        function getLocaleResource(): string {
          const defaultLocale = "en";
          let locale = localStorage.locale;
          if (!locale) {
            locale = defaultLocale;
          }
          return "./locales/" + locale;
        }
        ```
    - 新代码
         ```typescript
         loadLocaleResource().catch(err => console.error(err));
      
         async function loadLocaleResource() {
           const defaultLocale = "en";
           let locale = localStorage.locale;
           if (!locale) {
             locale = defaultLocale;
           }
           let localeResModule;
           switch (locale) {
           case "zh":
             localeResModule = await import("./locales/zh");
             break;
           case "company":
             localeResModule = await import("./locales/company");
             break;
           default:
             localeResModule = await import("./locales/en");
           }
           app.use(i18nPlugins, localeResModule.resource);
         }
         ```

这么做确实能将资源串文件编译进去问题解决。

### vue 页面刷新出现 404

由于我的项目路由使用的是 history 模式，所以必定会出现这个致命问题。

#### 原因

由于 Vue 项目是单页面应用，所以在刷新页面时，请求的 URI 很可能不存在，此时 Nginx 服务器就会返回 404 错误。

比如当我直接请求 `domain/transfer` 这个路径时，Nginx 匹配路径 `/transfer`，但是服务器上 `nginx.conf` 并没有配置针对这个路径的处理方式，所以 Nginx 就返回了 404 错误。

从 `domain` 进入 `domain/transfer` 不会出现问题，因为 Vue 项目是单页面应用，跳转路由的时候不会请求 `domain/transfer`，而是下载对应的资源文件，并修改地址栏。

#### 解决方案

网上主流有两种解决方案:

1. 路由模式从 history 模式改为 hash 模式，这样虽然解决了问题，但是地址栏的访问路径上会有一个符号 `#`。
2. 变更文件 `nginx.conf` 的配置，增加一个 `try_files` 配置以解决问题。

我选择修改 `nginx.conf`。修改成如下配置后重新加载 Nginx 问题便解决了。注意确保配置文件的修改刷新进入了 docker 容器内。

```
location / {
    root   /usr/share/nginx/html;
    index  index.html index.htm;
    try_files $uri $uri/ /index.html;
}
```

这么做后，当请求 URI 为 `domain/transfer`，Nginx 在找不到匹配路径时，会依次尝试以下文件或目录:

1. `/usr/share/nginx/html/transfer` (配置中变量 $uri 的值为 transfer)
2. `/usr/share/nginx/html/transfer/`
3. `/usr/share/nginx/html/index.html`

这样，浏览器向服务器请求资源的时候，都会返回 index.html 文件，让 vue 能够正常从 index.html 开始渲染页面，并能正确使用路由。

## 参考文献

1. [nginx-docker_hub](https://hub.docker.com/_/nginx)