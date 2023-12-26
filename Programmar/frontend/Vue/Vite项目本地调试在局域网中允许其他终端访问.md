# Vite项目本地调试在局域网中允许其他终端访问

## 正文

在配置文件 `vite.config.ts` 中增加配置项 `server`

```typescript
export default defineConfig({
    plugins: [
        // ...
    ],
    resolve: {
        // ...
    },
    build: {
        // ...
    },
    server: {
        host: "0.0.0.0",
    },
});
```

## 参考文献

1. [Vue 在vite中运行无法使用外部ip访问](https://www.jianshu.com/p/3b0444e75ce0)