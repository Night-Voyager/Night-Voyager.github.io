---
layout: post
title: 【Ant Design Vue】Statistic 统计数值参数valueStyle的用法
# author: 焜_8899
date: 2021-02-24 13:44:00 +0800
last_modified_at: 2021-02-24 13:51:49 +0800
tags: [Ant Design Vue]
toc:  true
---

## 1. 遇到的问题

在使用Ant Design Vue的[Statistic组件](https://www.antdv.com/components/statistic-cn/)时，希望通过其valueStyle参数来设置数值的样式，没有达到预期效果。要么数值的样式没有变化，要么页面无法正常渲染。

## 2. [官方文档](https://www.antdv.com/docs/vue/introduce-cn/)存在的问题

[API部分](https://www.antdv.com/components/statistic-cn/#API)的内容如下
> |参数|说明|类型|默认值|
> |:---:|:---:|:---:|:---:|
> |valueStyle|设置数值的样式|`style`|-|

参数名为`valueStyle`。

然而，[代码演示-在卡片中使用](https://www.antdv.com/components/statistic-cn/#%E5%9C%A8%E5%8D%A1%E7%89%87%E4%B8%AD%E4%BD%BF%E7%94%A8)中的代码却是
>```html
><a-statistic
>    title="Feedback"
>    :value="11.28"
>    :precision="2"
>    suffix="%"
>    :value-style="{ color: '#3f8600' }"
>    style="margin-right: 50px"
>>
>```

参数名为`:value-style`，参数值看上去是CSS。

## 3. 试验可行的用法

1. 参数名应为`:value-style`，参数值应为**JSON格式**的CSS。
2. 若CSS属性包含连字符“-”，则属性应当在引号中书写。
3. 若有多个属性，则使用逗号分隔。

以下是一个例子
```html
<a-statistic
    title="ABC"
    :value="123"
    :value-style="{ color: '#cf1322', 'text-decoration':'underline' }"
></a-statistic>
```
