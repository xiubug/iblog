---
title: nginx查看配置文件nginx.conf路径
tags:
  - path
categories:
  - nginx
date: 2018-03-31 22:56:28
---

当你执行 nginx -t 得时候，nginx会去测试你得配置文件得语法，并告诉你配置文件是否写得正确，同时也告诉了你配置文件得路径：

```bash
# nginx -t
nginx: the configuration file /usr/local/etc/nginx/nginx.conf syntax is ok
nginx: configuration file /usr/local/etc/nginx/nginx.conf test is successful
```

```bash
$ ps  -ef | grep nginx # 确定Nginx是以那个config文件启动的，也可以查看配置文件nginx.conf路径
```
