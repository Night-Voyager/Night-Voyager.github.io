---
layout: post
title: 【Electron + Vue】数据流转的几种方式
# author: 焜_8899
date: 2024-02-17 18:00:00 +0800
tags: [Electron, Vue]
toc:  true
---

## 1. Electron 与 Vue 之间的数据交互

## 2. Vue 内部的数据交互

### 2.1 父组件向子组件传递数据：props 和 attributes

传递 props 和透传 attributes 是两种常见的将数据传入组件的方式。

> 传入一个属性或者是事件，如果这个属性、事件没有在组件中定义，那么它依然是透传属性，因为没有东西接受它。
> 
> 如果是事先定义了 `defineEmits` 或 `defineProps` 来接受它，那么它就是 props 属性或自定义事件，不再是透传属性。
> 
> <p align="right">——[《一文搞懂Vue3中的透传属性》](https://juejin.cn/post/7086724982486597668)</p>

#### 2.1.1 Props

有关 props 的用法可参考[《组件基础·传递 props》](https://cn.vuejs.org/guide/essentials/component-basics.html#passing-props) 和 [《Props》](https://cn.vuejs.org/guide/components/props.html)。

另外，[Vue 的代码规范](https://cn.vuejs.org/guide/scaling-up/tooling.html#linting)强烈建议[在声明 props 的时候加上类型定义](https://eslint.vuejs.org/rules/require-prop-types.html)。

#### 2.1.2 Attributes

如果传递给一个组件的属性没有被该组件声明为 props 或者 emits，这个属性便是“[透传 attribute](https://cn.vuejs.org/guide/components/attrs.html)”。不过通常情况下还是会使用 props 向子组件传递数据。

### 2.2 祖先组件向后代组件传递数据：依赖注入

若是在深层次的组件树结构中通过 props 逐级传递数据，会使代码变得繁琐冗长，导致 [prop 逐级透传问题](https://cn.vuejs.org/guide/components/provide-inject.html#prop-drilling)。[依赖注入](https://cn.vuejs.org/guide/components/provide-inject.html)便可以解决这个问题。

### 2.3 多个组件共享数据：状态管理

有时候，可能会有多个组件依赖于或者要修改同一个数据。这些组件可能位于组件树的不同子树上。此时可以将这个数据提取出来，放到一个全局的单例中来管理。这样，任何位置上的组件都可以访问或更新此数据。

[Vue 有关状态管理的文档](https://cn.vuejs.org/guide/scaling-up/state-management.html)给出了其由来与方法，并推荐了 Vue 的专属状态管理库 [Pinia](https://pinia.vuejs.org/zh/)。
