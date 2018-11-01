---
title: git clone 太慢的解决方式
date: 2018-08-01 22:30:39
tags:
    - github
categories:
    - 项目管理
---

众所周知，虽然github没有在国内被墙，但是依然访问很慢。在 clone 项目到本地时，如果项目很大（比如我clone antd的时候估算了一下大概有一百多兆），那就很麻烦了。这里介绍一些比较通用的解决方案。在我机器上 clone 速度从二十几k提高到二百多k，瞬间拽的不要不要的。

## 一、查找域名对应的ip地址，并修改hosts文件

```bash
151.101.76.249 github.global.ssl.fastly.net 
192.30.253.112 github.com
```
