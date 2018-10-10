---
title: vscode 使用指南
tags:
  - vscode
categories:
  - vscode
date: 2018-05-05 22:44:10
---

### vscode 使用 postcss 语法并且对其支持emmet
安装`postcss-sugar-language`插件。
将css文件视作postcss（为了方便.css文件使用postcss语法不被报错），以及让emmet支持postcss文件。
在settings中的配置：
``` js
"files.associations": {
    "*.css": "postcss"
},
"emmet.includeLanguages": {
  "vue-html": "html",
  "javascript": "javascriptreact",
  "postcss": "css"
},
```
