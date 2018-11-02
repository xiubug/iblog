---
title: markdown表格中转义 " | "符号
date: 2018-08-13 20:09:32
tags:
  - markdown
categories:
  - markdown
---

今天用 markdown 语法写文档时，用到了 table 标签。文档中有一项用到`|`符号，发现`|`无法使用`反斜杠`转义。google一番找到了一个方法：使用[ASCII 字符集](http://www.runoob.com/tags/html-ascii.html)。举个简单的例子：

| 姓名      | 爱好        | 
| ------     | ------     | 
| Weich1     | 篮球 &#124; 游泳 |
| Weich2     | 足球 &#124; 音乐 |
| Weich3     | 爬山 |

注：「篮球 &\#124; 游泳&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;足球 &\#124; 音乐」
