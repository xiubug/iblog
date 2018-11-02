---
title: JAVA_HOME环境变量失效的解决办法
date: 2018-07-09 13:00:01
tags: 
    - JAVA_HOME
categories:
    - java
---

今天团队中一个同事搭建java开发环境，遇到了一个很奇怪的问题。在cmd中输入`java -version`会报如下错误:
``` js
Error:could not open `D:\Java\jre7\lib\amd64\jvm.cfg'
```

原因是因为重装导致的。

解决办法：

Path系统环境变量中，把%JAVA_HOME%\bin调整到最前面即可。