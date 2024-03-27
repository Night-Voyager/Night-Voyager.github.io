---
layout: post
title: 【Electron + Vue】使用log4js
# author: 焜_8899
date: 2024-03-27 18:33:58 +0800
tags: [Electron, Vue]
toc:  true
---

## 0. 背景

Electron 的结构中[包含两种类型的进程：**主进程**和**渲染器进程**](https://www.electronjs.org/zh/docs/latest/tutorial/process-model)。通过源码运行项目时，在主进程中使用 `console.log()` 打印的信息会被输出到运行项目的终端窗口中，而渲染器进程的内容则会显示在网页开发者工具的控制台里。希望能统一到一个地方来显示日志。

而且，有时候用 `console.log()` 会用得比较随意。所以还是想有一个专门的日志工具，使用它提供的功能。

## 1. 安装

可以使用 `yarn` 安装：

```console
yarn add log4js
```

## 2. 引入

无论是在主进程还是渲染器进程使用 log4js，都希望日志能显示到同一个地方。这可以通过[进程间通信（IPC）](https://www.electronjs.org/zh/docs/latest/tutorial/ipc)来实现，需要用到[预加载（preload）脚本](https://www.electronjs.org/zh/docs/latest/tutorial/process-model#preload-%E8%84%9A%E6%9C%AC)。

首先，可以先为 log4js 创建一个专门的文件 `src/utils/log.js`，设计相关的接口：

```javascript
const log4js = require('log4js');

const getLogger = (category, level = 'all') => {
    const logger = log4js.getLogger(category);
    logger.level = level;
    return logger;
};

export const trace = (event, category, message) => {
    getLogger(category).trace(message);
};


export const debug = (event, category, message) => {
    getLogger(category).debug(message);
};


export const info = (event, category, message) => {
    getLogger(category).info(message);
};


export const warn = (event, category, message) => {
    getLogger(category).warn(message);
};


export const error = (event, category, message) => {
    getLogger(category).error(message);
};


export const fatal = (event, category, message) => {
    getLogger(category).fatal(message);
};


export const mark = (event, category, message) => {
    getLogger(category).mark(message);
};

```

然后，在 `src/main.js` 文件中引入这些接口：

```diff
+import {debug, error, fatal, info, mark, trace, warn} from "./utils/log";
 
 // ...
 
 // This method will be called when Electron has finished
 // initialization and is ready to create browser windows.
 // Some APIs can only be used after this event occurs.
-app.on('ready', createWindow);
+app.on('ready', () => {
+  ipcMain.on('log4js:trace', trace);
+  ipcMain.on('log4js:debug', debug);
+  ipcMain.on('log4js:info', info);
+  ipcMain.on('log4js:warn', warn);
+  ipcMain.on('log4js:error', error);
+  ipcMain.on('log4js:fatal', fatal);
+  ipcMain.on('log4js:mark', mark);
+
+  createWindow();
+});

```

之后便可通过预加载脚本 `src/preload.js` 将接口暴露给渲染器进程：

```javascript
// See the Electron documentation for details on how to use preload scripts:
// https://www.electronjs.org/docs/latest/tutorial/process-model#preload-scripts
const { contextBridge, ipcRenderer } = require('electron');

contextBridge.exposeInMainWorld('getLogger', {
    trace: (category, message) => ipcRenderer.send('log4js:trace', category, message),
    debug: (category, message) => ipcRenderer.send('log4js:debug', category, message),
    info: (category, message) => ipcRenderer.send('log4js:info', category, message),
    warn: (category, message) => ipcRenderer.send('log4js:warn', category, message),
    error: (category, message) => ipcRenderer.send('log4js:error', category, message),
    fatal: (category, message) => ipcRenderer.send('log4js:fatal', category, message),
    mark: (category, message) => ipcRenderer.send('log4js:mark', category, message),
});

```

这样在主进程和渲染器进程中就都可以调用同样的日志接口了。

## 3. 使用

### 3.1 主进程

例如有一个被主进程使用的 `src/utils/syscall.js` 文件，其中有个 `ping()` 方法，调用时想要记录一下日志：

```diff
+import {info} from "./log";
+
 const util = require('util');
 const exec = util.promisify(require('child_process').exec);
 
 export const ping = (event, target) => {
     const ipv4Pattern = /((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})(\.((2(5[0-5]|[0-4]\d))|[0-1]?\d{1,2})){3}/;  // IPv4
     const domainNamePattern = /[a-zA-Z0-9][-a-zA-Z0-9]{0,62}(\.[a-zA-Z0-9][-a-zA-Z0-9]{0,62})+\.?/;       // 域名
 
     if ((!ipv4Pattern.test(target)) && (!domainNamePattern.test(target))) {
         return Promise.reject(new Error(
             target.toString() + ' is neither a valid IPv4 address nor a valid domain name.'
         ));
     }
 
+    info(null, 'syscall.js', `ping ${target}`);
     return exec('ping ' + target);
};

```

控制台中会打印出：

```
[2024-03-26T11:55:46.850] [INFO] syscall.js - ping 127.0.0.1
```

### 3.2 渲染器进程

假设有个 `src/components/SampleComponent.vue` 文件要记录日志：

```diff
 await window.syscall.ping(form.ip).then(() => {  // ping 目标地址
+    window.getLogger.info('SampleComponent.vue', `ping ${form.ip} success`);
     // ...
 }).catch((error) => {
+    window.getLogger.error('SampleComponent.vue', error.message);
     // ...
 });

```

控制台会显示：

```
[2024-03-26T11:55:49.957] [INFO] SampleComponent.vue - ping 127.0.0.1 success
[2024-03-26T11:57:22.315] [ERROR] SampleComponent.vue - Error invoking remote method 'ping': Error: Command failed: ping 0.0.0.0
```

## 4. 配置

要修改 log4js 的配置，可以修改 `src/utils/log.js` 文件中 `getLogger()` 方法的内容。例如在将日志输出到控制台同时保存到文件。

```diff
-const getLogger = (category, level = 'all') => {
-    const logger = log4js.getLogger(category);
-    logger.level = level;
-    return logger;
+const getLogger = (category, level = 'info') => {
+    log4js.configure({
+        appenders: {
+            console: { type: 'console' },
+            dateFile: {
+                type: 'dateFile',
+                filename: 'logs/log.log'
+            }
+        },
+        categories: {
+            default: { appenders: ['console', 'dateFile'], level: level }
+        }
+    });
+    return log4js.getLogger(category);
 };
```
