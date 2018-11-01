---
title: danger 入门教程
date: 2018-07-14 22:57:12
tags:
    - danger
categories:
    - 前端构建
---

## 背景

在 Code Review 时，我们可能经常要去检查各种事情，比如 pr 是否提到了 develop 分支、commit 中是否有毒（存在 merge commit）、禁止某些文件在 pr 中有修改、pr 的描述是否正常等等各种事情。有时我们会忘记检查这些事情，merge 之后才发现，这个就非常尴尬了。

## 快速入门指南

### 获取
我们建议通过 Yarn 安装 Danger，当然我们也可以使用 npm CLI。

### 安装
我们这边以 Yarn 为例：
``` bash
yarn add danger --dev
```

### 创建 Danger 配置文件 dangerfile.js 或 dangerfile.ts
