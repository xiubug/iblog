---
title: babel 使用指南
date: 2018-04-08 21:00:21
tags:
    - babel
categories:
    - babel
---

## 问题指南

### npm start

**Q：**由于 babel@7 升级导致的 break change，transform-decorators-legacy 无需引入了，如果引入就会报以下错误：
```
$ npm start

Starting the development server...

Failed to compile.

./src/routes/orders/AllPaidOrderListPage.js
Module build failed: TypeError: Property right of AssignmentExpression expected node to be of a type ["Expression"] but instead got null
at Array.map (<anonymous>)
```
**A：**
将`.webpackrc.js`中的`transform-decorators-legacy`进行删除。

[问题详情：](https://github.com/ant-design/ant-design-pro/pull/1345)
![babel-tutorial1.png](/images/babel-tutorial/img1.png)
