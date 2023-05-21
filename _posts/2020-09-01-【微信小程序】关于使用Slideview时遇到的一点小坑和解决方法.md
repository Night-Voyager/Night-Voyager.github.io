---
layout: post
title: 【微信小程序】关于使用Slideview时遇到的一点小坑和解决方法
# author: 焜_8899
date: 2020-09-01 02:23:02 +0800
last_modified_at: 2020-09-03 00:02:20 +0800
tags: [微信小程序]
toc:  true
---

## 1. 组件的获取与使用

关于WeUI的组件库，[这里](https://developers.weixin.qq.com/miniprogram/dev/extended/weui/)有官方的相关资料可供参考。

要下载包括Slideview在内的组件，请点击[这里](https://developers.weixin.qq.com/miniprogram/dev/extended/weui/download.html)。

关于下载后把文件放哪，其实无所谓。例如我自己的一个项目的目录结构是这样的：
![一个项目的目录结构](\assets\images\illustrations\003.jpg)

然后，在需要使用组件的页面的.json文件里加上：
```
{
  "usingComponents": {
    "mp-cells": "../../components/cells/cells",
    "mp-cell": "../../components/cell/cell",
    "mp-slideview": "../../components/slideview/slideview"
  }
}
```

其它的按照[Slideview的官方文档](https://developers.weixin.qq.com/miniprogram/dev/extended/weui/slideview.html)修改补充就行了。

## 2. Slideview样式缺失

明明官方的长这样子
![官方Slideview示例](\assets\images\illustrations\004.jpg)

而自己做出来的却长这样子
![旧样式与官方的相同，但新样式却不显示](\assets\images\illustrations\005.jpg)

再看一眼官方文档，发现 **没有给wxss！**
![官方示例代码](\assets\images\illustrations\006.jpg)

缺失的内容我找到了，放这儿了
```
.weui-slidecells{
  margin:8px;
}
.weui-slidecell{
  background-color:#FFFFFF;
  border-radius:8px;
  padding:24px;
  line-height:1.4;
  font-size:17px;
  color:rgba(0,0,0,.9);
  /*text-align:right;*/
}
.weui-slidecell__tips{
  color:rgba(0,0,0,.5);
}
```
**来源：**WeUI样式库（可参考我的[这篇文章]({% link _posts/2020-02-09-【微信小程序】使用微信原生样式.md %})）

## 3. Slideview样式缺失

如果做出来的是这个样子（有样式，无功能，既无法滑动也点击不了按钮）
![既无法滑动，也点击不了按钮，定死在这了](\assets\images\illustrations\007.jpg)

那么你的代码应该来自于[Github下载的WeUI样式库](https://github.com/Tencent/weui-wxss/)，而这里面的东西只是用来展示UI的，**没有功能**。

此时，你需要的是从[这里](https://developers.weixin.qq.com/miniprogram/dev/extended/weui/slideview.html)拷代码。

## 4. 左滑按钮没有图标

可以看到，官方文档的示例里面左滑是没有图标的
![官方文档示例没有图标](\assets\images\illustrations\008.jpg)

在WeUI样式库中，图标存放在`example/images/`路径下，复制过来就行了。**记得改代码里的路径。**
![WeUI样式库中的图标路径](\assets\images\illustrations\009.jpg)

## 5. 如何知道左滑后点击的是哪个按键

### 5.1 区分一个Slideview的多个按钮

根据官方文档，左滑的按钮组最多三个按钮，并且在点击时会触发事件并回传数据。其中，index便代表了点击的是第几个按钮。
![三个按钮分别传回的数据](\assets\images\illustrations\010.jpg)

### 5.2 区分有相同按钮的多个Slideview

有时候，需要拉取数据并依此进行列表渲染生成多个Slideview。这些Slideview会具有相同的左滑按钮组，点击时触发相同的事件。此时，可以为拉取的数据中每一个对象增加一个按钮属性，并在按钮的data属性中配上相应的id，之后再进行列表渲染。列表渲染时，Slideview中的buttons属性选用在拉取数据中加入的按钮属性。这样，在触发点击事件时，回传的参数中会带有data参数，是之前设置好的id。

###### JavaScript

```
/*从服务器获取数据列表*/
wx.request({
  url: '服务器API',
  method: 'GET',
  success(res) {
    console.log(res.data);

    /*设置左划按键返回的数据*/
    let items = res.data;
    for (let item of items) {
      item.slideButtons = [{ // 这里我只设置了一个按键
        type: 'warn',
        text: '删除',
        extClass: 'test',
        src: '/pages/images/icon_del.svg', // icon的路径
        data: item.id
      }];
    }

    that.setData({
      list: items.reverse() // 这里反转了列表顺序，使用体验更佳
    })
  }
})
```

###### WXML

```
<view class="weui-slidecells" wx:for="{{list}}">
  <mp-slideview buttons="{{item.slideButtons}}" icon="{{true}}" bindbuttontap="slideButtonTap">
    <view class="weui-slidecell">
      {{item.name}}
    </view>
  </mp-slideview>
</view>
```

关于列表渲染的用法可看[这里](https://developers.weixin.qq.com/miniprogram/dev/reference/wxml/list.html)，不再另行赘述。

## 6. 参考文献

除以下文献外，本文所有参考文献均已以链接的形式标注在了对应位置。

[1] ELEPHANT LEG. 微信小程序 weui slideview 组件点击时传递参数[EB/OL]. (2020-08-06)[2020-09-02]. https://www.sunzhongwei.com/wechat-applet-weui-slideview-component-click-when-passing-parameters.
