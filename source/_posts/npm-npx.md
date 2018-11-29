---
title: npx—npm 5.2.0 内置的包执行器
tags:
  - npx
categories:
  - npm
date: 2018-11-29 20:19:27
---
最近在更新 npm 5.2.0 的时候，我发现除了可以使用npm命令，还有一个命令可以使用，这就是npx。

npm使得安装和管理依赖包变得非常简单，和npm的类似，npx这款工具旨在提供给用户更方便的包操作体验。当我们使用一些工具命令或者可执行文件时，相对于npm，npx把这个过程变得更加简单了。

根据[zkat/npx](https://github.com/zkat/npx) 的描述，npx 会帮我们执行依赖包里的二进制文件。
举例来说，之前我们可能会写这样的命令：
``` bash
npm i -D webpack
./node_modules/.bin/webpack -v
```
如果你对 bash 比较熟，可能会写成这样：
``` bash
npm i -D webpack
`npm bin`/webpack -v
```
有了 npx，我们只需要这样：
``` bash
$ npx webpack -v
```
也就是说 npx 会自动查找当前依赖包中的可执行文件，如果找不到，就会去 PATH 里找。如果依然找不到，就会帮你安装！
npx 甚至支持运行远程仓库的可执行文件，如
``` bash
$ npx github:piuccio/cowsay hello
npx: 1 安装成功，用时 1.663 秒
 _______
< hello >
 -------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```
再比如 npx http-server 可以一句话帮你开启一个静态服务器！（第一次运行会稍微慢一些）
``` bash
$ npx http-server
npx: 23 安装成功，用时 48.633 秒
Starting up http-server, serving ./
Available on:
  http://127.0.0.1:8080
  http://192.168.5.14:8080
Hit CTRL-C to stop the server
```
