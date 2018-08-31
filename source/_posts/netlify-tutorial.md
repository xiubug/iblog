---
title: netlify 入门教程
date: 2018-07-02 10:40:48
tags:
    - netlify
categories:
    - netlify
---

## 快速入门指南

### 部署简单

通过持续部署，您可以通过推送到Git或通过webhook来触发构建。

### 支持HTTPS

使用我们真正的一键式SSL设置保护您的网站或应用。Netlify自动与Let's Encrypt集成，并自动配置，分发和更新您的证书。

### 支持命令行

如果你住在终端，Netlify的命令行工具将是你最好的朋友。您可以直接从终端访问任何Netlify功能。

React 源码 netlify 的配置文件 `netlify.toml`：
``` toml
[build]
  base    = ""
  publish = "fixtures/dom/build"
  command = "yarn build --type=UMD_DEV && cd fixtures/dom/ && yarn && yarn prestart && yarn build"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

![img1.png](netlify-tutorial/img1.png)
