---
title: 使用lerna优雅地管理多个package
date: 2018-07-21 20:55:06
tags:
    - lerna
categories:
    - lerna
---

## 背景

对于维护过多个package的同学来说，都会遇到一个选择器：**这些package是放在一个仓库里维护还是放在多个仓库里单独维护**，数量较少的时候，多个仓库维护不会有太大问题，但是当package数量逐渐增多时，一些问题逐渐暴露出来：
1. package之间相互依赖，开发人员需要在本地手动执行**npm link**，维护版本号的更替；
2. issue难以统一追踪，管理，因为其分散在独立的repo里；
3. 每一个package都包含独立的node_modules，而且大部分都包含babel,webpack等开发时依赖，安装耗时冗余并且占用过多空间。
