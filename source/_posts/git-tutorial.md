---
title: git 开发汇总
tags:
  - note
categories:
  - git
date: 2018-10-04 20:34:09
---

## 常见问题

### 使用.gitkeep来追踪空的文件夹

Git会忽略空的文件夹。如果你想版本控制包括空文件夹，根据惯例会在空文件夹下放置.gitkeep文件。其实对文件名没有特定的要求。一旦一个空文件夹下有文件后，这个文件夹就会在版本控制范围内。

### 当用git命令拉取最新代码时，有时会遇到如下的提示， Found a swap file by the name “.git/.MERGE_MSG.swp”

在项目根目录（如/StudioProjects/demo/Leave）下，找到.git/.MERGE_MSG.swp这个文件删除即可。 
注：mac 删除命令rm -rf .MERGE_MSG.swp