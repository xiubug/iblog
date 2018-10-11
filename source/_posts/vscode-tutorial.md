---
title: vscode 使用指南
tags:
  - vscode
categories:
  - vscode
date: 2018-05-05 22:44:10
---
## css
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

## vue
### 使用vscode，新建vue模板
文件 --> 首选项 --> 用户代码片段 --> 输入vue，选择vue -->复制 以下内容保存：
``` json
{    
  "Print to console": {
    "prefix": "vue",
    "body": [
        "<template>",
        "   <div class=\"\">\n",
        "   </div>",
        "</template>\n",
        "<script type=\"text/ecmascript-6\">",
        "export default {",
        "   name: '',",
        "   data() {",
        "       return {}",
        "   },",
        "  components: {}",
        "}",
        "</script>\n",
        "<style scoped lang=\"stylus\">",
        "</style>",
        "$2"
    ],
    "description": "Log output to console"
  }
}
```
新建vue文件，输入vue，按下tab键即可。