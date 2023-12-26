# Vue安装VueI18n

1. 安装 Vue I18n
   ```shell
   npm install vue-i18n@9
   ```
2. 在 src 目录下创建 locales
3. main.js 内加入以下代码，不要一次粘贴，分散加入
   ```typescript
   import { createI18n } from "vue-i18n";
   import zhCN from "./locales/zh-CN.json";
   import enUS from "./locales/en-US.json";
   const locales = {
   "zh-CN": zhCN,
   "en-US": enUS,
   };
   const i18n = createI18n({
   locale: navigator.language,
   fallbackLocale: "en-US",
   messages: locales,
   });
   app.use(i18n);
   app.provide("i18n", i18n);
   ```
   

## 参考资料

1. [Vue I18n Installation](https://vue-i18n.intlify.dev/guide/installation.html)
2. [Vue I18n TypeScript Support](https://vue-i18n.intlify.dev/guide/advanced/typescript)