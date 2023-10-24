---
layout: post
title: 【Frida】JavaScript读取并解析json文件的一种方式
# author: 焜_8899
date: 2023-07-05 18:00:00 +0800
tags: [Frida, JavaScript, json]
toc:  true
---

## 0. 背景

希望 frida 在 hook 时，js 脚本能读取并解析 json 文件。此脚本运行在 Linux 系统上。

## 1. 代码

```JavaScript
const openPtr = Module.getExportByName(null, "open");  // 获取系统 open 函数的地址
const open = new NativeFunction(openPtr, "int", ["pointer", "int"]);  // 创建新的函数对象以调用上一行获取的 open 函数
const fd = open(Memory.allocUtf8String("sample.json"), 0);  // 获取所需要解析的文件的文件描述符
if (fd > 0) {
    const input = new UnixInputStream(fd);  // 通过文件描述符创建输入流
    const promise = input.read(2048);  // 从输入流中读取文件内容，read 函数会返回一个 Promise 对象，该对象会接收一个 ArrayBuffer，其中包含读取到的数据
    promise
        .then(
            (arraybuffer) => {
                const content = arraybuffer.unwrap().readUtf8String();  // 将 ArrayBuffer 中的内容转为字符串
                const jsonObj = JSON.parse(content);  // 将字符串解析为 JSON 对象
                send(jsonObj["key"]);  // 通过键获取 JSON 对象中的值，并发送回 hook
                send("stop");
            }
        )
        .catch(
            (error) => {
                send(error.toString());
                send("stop");
            }
        );
}
```

## 2. 思路

要完成所需功能，思路主要分为两部分：读取文件和解析 json。解析 json 的部分比较简单，使用 `JSON.parse()` 即可将一个 json 格式的字符串转换为 JavaScript 对象。而从文件中读出所需字符串的实现方式则要复杂一些。

### 2.1 解析 json

解析 json 的方法比较简单，`var obj = JSON.parse(text);` 即可。其中 `text` 是要解析的 json 格式字符串，`obj` 是解出来的 JavaScript 对象。

更详细的说明可参考[这篇文章](https://www.runoob.com/js/js-json.html)和[这篇文章](https://www.runoob.com/json/json-parse.html)，在此不再赘述。

### 2.2 读取文件

根据 [frida 的文档](https://frida.re/docs/javascript-api/#unixinputstream)，`new UnixInputStream(fd[, options])` 可以从指定文件中读取内容，其中 `fd` 是要读的文件的文件描述符。

#### 2.2.1 获取文件描述符

要获取文件描述符，可以调用 Linux 的系统函数 [`int open(const char *pathname, int flags);`](https://www.man7.org/linux/man-pages/man2/open.2.html)：
 - `pathname` 是表示文件路径的字符串；
 - `flags` 是代表打开方式的整数：
   - 可选的打开方式有只读（`O_RDONLY`）、只写（`O_WRONLY`）和读写（`O_RDWR`），分别对应的值为 `0`、`1` 和 `2`；
 - 函数返回的整数就是文件描述符。

要调用系统的 `open` 函数，可以先用 frida 的 [`Module.getExportByName(moduleName|null, exportName)`](https://frida.re/docs/javascript-api/#module) 函数来通过函数名获取其地址，再将地址传入 [`new NativeFunction(address, returnType, argTypes[, abi])`](https://frida.re/docs/javascript-api/#nativefunction) 函数来创建新的函数对象以便调用。
 - `Module.getExportByName` 函数可以从指定模块中导出函数的地址：
   - `moduleName` 是模块名称：
     - 如果模块名称未知，可以传入 `null` 作为模块名称；
   - 参数 `exportName` 是要导出的函数的名称；
   - 函数的返回值就是导出函数的地址。
 - `NativeFunction` 用于创建一个新的函数对象，它可以调用指定地址处的函数：
   - `address` 是要调用的函数的地址；
   - `returnType` 是所调函数的返回值类型；
   - `argTypes` 是一个数组，其内容是被调用函数的参数列表中每个参数对应的类型；
   - 以 `1. 代码` 中的程序第 2 行为例：
     - 要调用的函数是 `open`，其地址已通过 `Module.getExportByName(null, "open")` 获得；
     - `open` 的返回类型为整数，因此 `returnType` 为 `"int"`；
     - `open` 所需的参数是一个指向 C 字符串的指针和一个整数，所以 `argTypes` 处应填入 `["pointer", "int"]`。


上文提到，`open` 的第一个参数是个指向 C 字符串的指针。然而，JavaScript 字符串没法直接传给 `open` 函数。因此需要借助 [`Memory.allocUtf8String(str)`](https://frida.re/docs/javascript-api/#memory) 函数来分配一块空间并写入所需字符串，该函数的返回值便是指向字符串的指针。这样就可以将其传递给 `open` 函数了。

#### 2.2.2 创建文件输入流

获取到文件描述符之后，[`UnixInputStream`](https://frida.re/docs/javascript-api/#unixinputstream) 函数便可以通过该描述符来构造输入流对象。它的 [`read(size)`](https://frida.re/docs/javascript-api/#inputstream) 函数会返回一个 [`Promise`](https://www.runoob.com/js/js-promise.html) 对象。该对象会接收一个 [`ArrayBuffer`](https://frida.re/docs/javascript-api/#arraybuffer)，其中包含读取到的数据。

要从 `ArrayBuffer` 解出所需的数据，可以调用它的 `unwrap()` 函数。该函数返回一个 [`NativePointer`](https://frida.re/docs/javascript-api/#nativepointer)。再调用 `NativePointer` 的 `readUtf8String([size = -1])` 函数即可读出其中的字符串。
