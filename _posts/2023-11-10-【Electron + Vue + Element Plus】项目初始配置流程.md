---
layout: post
title: 【Electron + Vue + Element Plus】项目初始配置流程
# author: 焜_8899
date: 2023-11-10 00:00:01 +0800
tags: [Electron, Vue, Element Plus]
toc:  true
---

## 1. 新建项目

根据 [Electron Forge 的文档](https://www.electronforge.io/guides/framework-integration/vue-3)，可以在要创建项目的路径下，控制台执行以下两条命令之一。命令在执行的过程中会在当前路径下新建一个文件夹，并在其中初始化项目。

###### yarn

```shell
yarn create electron-app my-vue-app --template=vite
```

###### npm

```shell
npm init electron-app@latest my-vue-app -- --template=vite
```

## 2. 添加依赖

进入上述命令执行所生成的文件夹，然后相应地执行以下命令。

###### yarn

```shell
yarn add vue
yarn add --dev @vitejs/plugin-vue
```

###### npm

```shell
npm install vue
npm install --save-dev @vitejs/plugin-vue
```

## 3. 整合 Vue 3 代码

以下展示各文件更改后的内容作为参考。

###### ~/index.html

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello World!</title>

  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/renderer.js"></script>
  </body>
</html>

```

###### ~/src/App.vue（新建文件）

```vue
<template>
    <h1>💖 Hello World!</h1>
    <p>Welcome to your Electron application.</p>
</template>

<script setup>
console.log('👋 This message is being logged by "App.vue", included via Vite');
</script>
```

###### ~/src/renderer.js

```js
import { createApp } from 'vue';
import App from './App.vue';

createApp(App).mount('#app');

```

###### ~/vite.renderer.config.mjs

```js
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';

// https://vitejs.dev/config
export default defineConfig({
    plugins: [vue()]
});

```

此时项目就可以运行了。如果用的 IDE 是 WebStorm，那么在 `package.json` 中 `"start"` 所在行前可看到一个绿色的播放按钮。点击即可运行。

## 4. 安装 Element Plus

根据 [Element Plus 的文档](http://element-plus.org/zh-CN/guide/installation.html)，有以下几种建议的安装方式。

###### yarn

```shell
yarn add element-plus
```

###### npm

```shell
npm install element-plus --save
```

###### pnpm

```shell
pnpm install element-plus
```

## 5. 完整引入

完整导入 Element Plus 之后使用起来会比较方便，不容易出现样式丢失的问题。

用法可参考 [Element Plus 的文档](http://element-plus.org/zh-CN/guide/quickstart.html)，在本项目中应当修改 `src/renderer.js` 文件。

以下是文件修改后的内容。

###### ~/src/renderer.js

```js
import { createApp } from 'vue';
import App from './App.vue';
import ElementPlus from 'element-plus';
import 'element-plus/dist/index.css';

createApp(App).use(ElementPlus).mount('#app');

```