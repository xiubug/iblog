---
title: vscode 使用汇总
tags:
  - vscode
categories:
  - vscode
date: 2018-05-05 22:44:10
---
## 基本使用

### 常用命名
ctrl + k 清空终端内容

### 设置

#### 设置tab的空格数

方案一：文件 -> 首选项 -> 设置。将 editor.tabSize 改为 2。注意：复制到右侧修改。

## 插件

### Setting Sync 插件
>使用插件将目前配置保存到GitHub上，以后只需要从GitHub上获取，就可以一次性安装插件配置信息。

#### 适用于

电脑更换时还要一个个去搜索插件安装，公司电脑和个人电脑同步。
* 注册申请github账号
* 安装了git

#### 步骤如下

##### 首先在VSCode里面搜索Setting Sync插件，安装好后重新加载激活

* Upload Key : Shift + Alt + U
* Download Key : Shift + Alt + D
![vscode-tutorial1.png](/images/vscode-tutorial/img1.png)

##### Shift + Alt + U

* 这一步好像在有文件打开时才有用
![vscode-tutorial2.png](/images/vscode-tutorial/img2.png)

* 在跳出来的页面点击 generate new tooken
![vscode-tutorial3.png](/images/vscode-tutorial/img3.png)

* 将生成的key输入vscode命令框里
![vscode-tutorial4.png](/images/vscode-tutorial/img4.png)

##### 上传完成后会生成一个ID，要记下来

ID和key不同

##### 使用Shift + Alt + D，输入ID，即可开始同步配置

## 常见问题
### ES6 及以上
#### vscode中使用Experimental Decorators报错

在vscode中使用es7的新语法decorator会报错：

```javascript
对修饰器的实验支持是一项将在将来版本中更改的功能。设置 "experimentalDecorators" 选项以删除此警告
```

这个错误来自与vscode的JS support，只要在项目根目录下创建一个jsconfig.json文件，添加如下内容：
```javascript
{
    "compilerOptions": {
        "experimentalDecorators": true
    }
}
```
添加好了有可能需要重启才能生效。
### css 相关
#### vscode 使用 postcss 语法并且对其支持emmet
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

### vue 相关
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