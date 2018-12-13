---
title: 解决本地开发ajax跨域问题
tags:
  - ajax
categories:
  - http
date: 2018-12-03 21:33:09
---
前后端分离，本地前端开发调用接口会有跨域问题，一般有以下几种解决方法：
1. 后端接口打包到本地运行（缺点：每次后端更新都要去测试服下一个更新包，还要在本地搭建java运行环境，麻烦）。
2. CORS跨域：后端接口在返回的时候，在header中加入'Access-Control-Allow-origin':* 之类的（有的时候后端不方便这样处理，前端就蛋疼了）。
3. 用nodejs搭建本地http服务器，并且判断访问接口URL时进行转发，完美解决本地开发时候的跨域问题。
4. 使用谷歌的插件`allow-control-allow-origi`解决，或者谷歌开启允许跨域，参考 http://camnpr.com/archives/chrome-args-disable-web-security.html
