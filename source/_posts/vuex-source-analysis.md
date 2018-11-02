---
title: vuex 源码全方位剖析
tags:
  - source
categories:
  - vuex
date: 2018-06-29 20:11:43
---

## 背景
Vue 火不火？我们看 github stars 就知道，现在已超越 React 了，国民的力量还是很强大的。我们在使用 Vue.js 开发应用时，经常会遇到多个组件共享同一个状态，也或者多个组件去更新同一个状态。对于简单的应用，我们可以
