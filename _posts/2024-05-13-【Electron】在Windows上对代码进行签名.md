---
layout: post
title: 【Electron】在Windows上对代码进行签名
# author: 焜_8899
date: 2024-05-13
tags: [Electron]
toc:  true
---

## 0. 背景

Electron 的[这篇文档](https://www.electronjs.org/zh/docs/latest/tutorial/%E6%89%93%E5%8C%85%E6%95%99%E7%A8%8B#%E9%87%8D%E8%A6%81%E6%8F%90%E7%A4%BA%E5%AF%B9%E4%BB%A3%E7%A0%81%E8%BF%9B%E8%A1%8C%E7%AD%BE%E5%90%8D)讲解了有关代码签名相关的事项，然而并没有说明如何获取其例子中所用到的 `cert.pfx` 文件。

## 1. 方法

[《如何创建应用包签名证书》](https://learn.microsoft.com/zh-cn/windows/win32/appxpkg/how-to-create-a-package-signing-certificate)这篇指南讲解了签名的步骤，然而文中还是有些小问题。

### 1.1 问题一：MakeCert.exe 已弃用

> MakeCert.exe 已弃用。有关创建证书的最新指南，请参阅[为包签名创建证书](https://learn.microsoft.com/zh-cn/windows/msix/package/create-certificate-package-signing)。

尽管 MakeCert.exe 已弃用，依然可以使用文件搜索工具在系统中搜到此文件，可拷贝出来正常使用。

用同样的方式也可以找到指南中所需要用到的 Pvk2Pfx.exe 文件。

### 1.2 问题二：命令无法使用

上述指南所提供的有关 MakeCert 工具的命令无法直接使用。

MakeCert 工具的用法可参考：[MakeCert - Windows drivers \| Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-hardware/drivers/devtest/makecert)。

Pvk2Pfx 工具的用法可参考：[Pvk2Pfx - Windows drivers \| Microsoft Learn](https://learn.microsoft.com/zh-cn/windows-hardware/drivers/devtest/pvk2pfx)。

### 1.3 问题三：缺少相关文件

使用 MakeCert 指南中的示例命令，仅会生成一个 `.cer` 文件。然而，使用 Pvk2Pfx 工具还需要一个 `.pvk` 文件。

该文件可以在使用 MakeCert 工具时添加 `-sv` 参数并指定文件名来生成。

添加此参数后，将会出现两个需要操作的弹窗。分别输入所需的密码即可。

![创建私钥密码](/assets/images/illustrations/20240513/创建私钥密码.png)

![输入私钥密码](/assets/images/illustrations/20240513/输入私钥密码.png)

## 2. 总结

简而言之，要对 Electron 应用进行代码签名，可以使用 MakeCert 和 Pvk2Pfx 两个工具。

首先，可参考以下命令使用 MakeCert 工具得到 `.pvk` 和 `.cer` 文件：
```
MakeCert -r -pe -ss PrivateCertStore -n "CN=Contoso.com(Test)" -sv MyKey.pvk testcert.cer
```

然后，可参考以下命令使用 Pvk2Pfx 工具，即可得到代码签名证书文件 `cert.pfx`：
```
pvk2pfx -pvk MyKey.pvk -pi sample_password -spc testcert.cer -pfx cert.pfx -f
```
