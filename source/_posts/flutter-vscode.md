---
title: vscode快速构建Flutter项目
tags:
  - vscode
categories:
  - flutter
date: 2019-01-09 21:46:40
---

### 设置环境变量

由于在国内访问Flutter有时可能会受到限制，Flutter官方为中国开发者搭建了临时镜像，在操作之前大家可以将如下环境变量加入到用户环境变量中：
``` bash
$ export PUB_HOSTED_URL=https://pub.flutter-io.cn
$ export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn
```

### 安装 Flutter 插件

插件中安装 Flutter 插件，安装完毕重新载入软件，安装 Flutter 插件的时候会默认的安装 Dart 插件（Dart是谷歌开发的计算机编程语言）:

![img1.png](/images/flutter-vscode/img1.png)

### Flutter:new project

打开vscode命令面板（comm+shift+p）选择Flutter:new project：

![img2.png](/images/flutter-vscode/img2.png)

### 项目简介

输入我们需要的项目名称选择对应存放的文件位置，等待依赖下载，我们会看到下图：

![img3.png](/images/flutter-vscode/img3.png)

### 添加设备

点击设备可以创建设备，开启已有设备，开启设备后选择到调试（虫子），添加调试配置，只管添加配置，然后保存就好了：

![img4.png](/images/flutter-vscode/img4.png)

### 开启调试

选择左上方的开启调试。项目开始打包构建安装到选择的选择的设备上：

![img5.png](/images/flutter-vscode/img5.png)

### 编辑

`lib/main.dart`中编辑插入`Text('hello flutter')`，保存文件，我们会发现效果会立马呈现到 App 上：

![img6.png](/images/flutter-vscode/img6.png)

### 至此，教程就到这里了。


