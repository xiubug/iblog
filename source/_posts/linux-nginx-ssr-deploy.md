---
title: next、nuxt等服务端渲染框架如何在服务器部署，并用PM2守护程序
date: 2018-08-03 20:31:37
tags:
    - spa
    - pm2
    - ssr
categories:
    - nginx
---

貌似从前几年，前后端分离逐渐就开始流行起来，把一些渲染计算的工作抛向前端以便减轻服务端的压力，但为啥现在又开始流行在服务端渲染了呢？如vue全家桶或者react全家桶，都推荐通过服务端渲染来实现路由。搞得我们慌得不行，不禁让我想起一句话：`从来没有任何一门语言的技术栈像Javascript一样，学习者拼尽全力也不让精通。`没办法，流行，咱们就得学！

前断时间写了一篇[vuejs、react如何在服务器部署？](http://www.sosout.com/2018/08/02/linux-nginx-spa-deploy.html)，结果反响不错！最近好多朋友私信或邀请问很多关于next.js和nuxt.js的问题，比如`关于nextjs 和 nuxtjs如何部署`，`pm2如何配合`...在这里我们就一起讨论下在服务器上使用PM2守护next.js、nuxt.js等服务端渲染框架构建的项目！该篇我们只讨论`服务端渲染应用部署`，`静态应用部署`就是我前段时间写的[vuejs、react如何在服务器部署？](http://www.sosout.com/2018/08/02/linux-nginx-spa-deploy.html)。

#### Nginx配置

既然是应用，我们就应该有域名，在这里我们以` nginx配置 `为例，简单配置如下：
**Next域名：**http://next.sosout.com/ 
**Nuxt域名：**http://nuxt.sosout.com/ 
``` nginx
http {
    ....  # 省略其他配置
   
    server {
        listen 80;
        server_name  *.sosout.com;
        
        if ($host ~* "^(.*?)\.sosout\.com$") {
            set $domain $1;
        }

        location / {
            if ($domain ~* "next") {
                root /mnt/html/next;
            }
            if ($domain ~* "nuxt") {
                root /mnt/html/nuxt;
            }
            proxy_set_header   Host             $host;
            proxy_set_header   X-Real-IP        $remote_addr;
            proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto  $scheme;
        }
        access_log  /mnt/logs/nginx/access.log  main;
    }

    #tcp_nopush     on;

    include /etc/nginx/conf.d/*.conf;
}

```

#### Nginx反向代理
由于服务端渲染的各个应用端口号各不相同，因此这个时候我们就需要反向代理了，配置如下：

``` nginx
#通过upstream nodejs 可以配置多台nodejs节点，做负载均衡
#keepalive 设置存活时间。如果不设置可能会产生大量的timewait
#proxy_pass 反向代理转发 http://nodejs

upstream nodenext {
	server 127.0.0.1:3001; #next项目 监听端口
	keepalive 64;
}

server {
	listen 80;
	server_name next.sosout.com;
	location / {
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;  
		proxy_set_header Connection "upgrade";
		proxy_set_header Host $host;
		proxy_set_header X-Nginx-Proxy true;
        proxy_cache_bypass $http_upgrade;
        proxy_pass http://nodenext; #反向代理
	}
}

upstream nodenuxt {
	server 127.0.0.1:3002; #nuxt项目 监听端口
	keepalive 64;
}

server {
	listen 80;
	server_name nuxt.sosout.com;
	location / {
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;  
		proxy_set_header Connection "upgrade";
		proxy_set_header Host $host;
		proxy_set_header X-Nginx-Proxy true;
        proxy_cache_bypass $http_upgrade;
        proxy_pass http://nodenuxt; #反向代理
	}
}
```
`服务器的准备工作已完成，接下来我们就分别看看Next.js和Nuxt.js服务端渲染应用如何部署？`

#### Next.js服务端渲染应用部署

部署 Next.js 服务端渲染的应用不能直接使用` next `命令，而应该先进行编译构建，然后再启动 Next 服务，官方通过以下两个命令来完成：
``` bash
$ next build
$ next start
```
官方推荐的 package.json 配置如下：

``` json
{
  "name": "my-app",
  "dependencies": {
    "next": "latest"
  },
  "scripts": {
    "dev": "next",
    "build": "next build",
    "start": "next start"
  }
}
```
而我更推荐如下配置，稍后你会发现这样和` pm2 `一起使用更方便，自动化部署也方便:

``` json
{
  "name": "my-app",
  "dependencies": {
    "next": "latest"
  },
  "scripts": {
    "dev": "next",
    "start": "next start -p $PORT",
    "build": "next build && PORT=3001 npm start"
  }
}
```
next.js服务端渲染应用部署这样就完成了，官方先后执行`npm run build 、npm start`即可完成部署。而我这边只要执行` npm run build `，其实我只是把两个合并成一个，并设置了端口以便区别其他应用，避免端口占用！

**接下来简单的说一下next这几个命令：**
`next: `启动一个热加载的Web服务器（开发模式）
`next build: `利用webpack编译应用，压缩JS和CSS资源（发布用）。
`next start: `以生成模式启动一个Web服务器 (`next build` 会先被执行)。

#### Nuxt.js服务端渲染应用部署
**其实部署 Nuxt.js 服务端渲染的应用和` Next.js `极其相似！在这里我就把代码粘粘贴贴，复复制制，改改写写。。。。**
Nuxt.js 服务端渲染的应用不能直接使用` nuxt `命令，而应该先进行编译构建，然后再启动 Nuxt 服务，官方通过以下两个命令来完成：

``` bash
$ nuxt build
$ nuxt start
```
官方推荐的 package.json 配置如下：

``` json
{
  "name": "my-app",
  "dependencies": {
    "nuxt": "latest"
  },
  "scripts": {
    "dev": "nuxt",
    "build": "nuxt build",
    "start": "nuxt start"
  }
}
```
而我更推荐如下配置，稍后你会发现这样和` pm2 `一起使用更方便，自动化部署也方便:

``` json
{
  "name": "my-app",
  "dependencies": {
    "nuxt": "latest"
  },
  "scripts": {
    "dev": "nuxt",
    "start": "PORT=3002 nuxt start",
    "build": "nuxt build && npm start"
  }
}
```
nuxt.js服务端渲染应用部署这样就完成了，官方先后执行`npm run build 、npm start`即可完成部署。而我这边只要执行` npm run build `，其实我只是把两个合并成一个，并设置了端口以便区别其他应用，避免端口占用！

**接下来简单的说一下nuxt这几个命令：**
`nuxt: `启动一个热加载的Web服务器（开发模式）
`nuxt build: `利用webpack编译应用，压缩JS和CSS资源（发布用）。
`nuxt start: `以生成模式启动一个Web服务器 (`nuxt build` 会先被执行)。

#### PM2守护程序
**Next.js使用pm2，进入对应的应用目录，执行以下命令：**

``` bash
$ pm2 start npm --name "my-next" -- run build
```
**Nuxt.js使用pm2，进入对应的应用目录，执行以下命令：**

``` bash
$ pm2 start npm --name "my-nuxt" -- run build
```
`使用pm2时，把两个部署命令合成一个更方便！`执行完pm2的启动命令后，我们用` pm2 list `查看一下进程列表，我截一下我个人服务器的pm2列表：

![img1.png](linux-nginx-ssr-deploy/img1.png)

以后您就可以用pm2进行维护了，比如我们的next应用更改了代码，因为当时创建时给next应用命名的进程名称为` my-next `，因此我们可以直接使用` pm2 reload my-next `进行重载。接下来我就简单介绍一下pm2，如果有需要，我可以另写一篇关于pm2的文章！

#### pm2 简单介绍
pm2是nodejs的一个带有负载均衡功能的应用进程管理器的模块，类似有Supervisor，forever，用来进行进程管理。

##### 一、安装：
``` bash
$ npm install pm2 -g
```
##### 二、启动：
``` bash
$ pm2 start app.js
$ pm2 start app.js --name my-api       #my-api为PM2进程名称
$ pm2 start app.js -i 0                #根据CPU核数启动进程个数
$ pm2 start app.js --watch             #实时监控app.js的方式启动，当app.js文件有变动时，pm2会自动reload
```
##### 三、查看进程：
``` bash
$ pm2 list
$ pm2 show 0 或者 # pm2 info 0         #查看进程详细信息，0为PM2进程id 
```
##### 四、监控： 
``` bash
$ pm2 monit
```
##### 五、停止：
``` bash
$ pm2 stop all                         #停止PM2列表中所有的进程
$ pm2 stop 0                           #停止PM2列表中进程为0的进程
```
##### 六、重载：
``` bash
$ pm2 reload all                       #重载PM2列表中所有的进程
$ pm2 reload 0                         #重载PM2列表中进程为0的进程
```
##### 七、重启：
``` bash
$ pm2 restart all                      #重启PM2列表中所有的进程
$ pm2 restart 0                        #重启PM2列表中进程为0的进程
```
##### 八、删除PM2进程：
``` bash
$ pm2 delete 0                         #删除PM2列表中进程为0的进程
$ pm2 delete all                       #删除PM2列表中所有的进程
```
##### 九、日志操作：
``` bash
$ pm2 logs [--raw]                     #Display all processes logs in streaming
$ pm2 flush                            #Empty all log file
$ pm2 reloadLogs                       #Reload all logs
```
##### 十、升级PM2：
``` bash
$ npm install pm2@lastest -g           #安装最新的PM2版本
$ pm2 updatePM2                        #升级pm2
```
##### 十一、更多命令参数请查看帮助：
``` bash
$ pm2 --help
```
##### 十二、PM2目录结构：
* 1、默认的目录是：当前用于的家目录下的.pm2目录（此目录可以自定义，请参考：十三、自定义启动文件），详细信息如下：
``` js
$HOME/.pm2                   #will contain all PM2 related files
$HOME/.pm2/logs              #will contain all applications logs
$HOME/.pm2/pids              #will contain all applications pids
$HOME/.pm2/pm2.log           #PM2 logs
$HOME/.pm2/pm2.pid           #PM2 pid
$HOME/.pm2/rpc.sock          #Socket file for remote commands
$HOME/.pm2/pub.sock          #Socket file for publishable events
$HOME/.pm2/conf.js           #PM2 Configuration
```
##### 十三、自定义启动文件：
* 1、创建一个test.json的示例文件，格式如下：
``` json
{
  "apps":
    {
      "name": "test",
      "cwd": "/data/wwwroot/nodejs",
      "script": "./test.sh",
      "exec_interpreter": "bash",
      "min_uptime": "60s",
      "max_restarts": 30,
      "exec_mode" : "cluster_mode",
      "error_file" : "./test-err.log",
      "out_file": "./test-out.log",
      "pid_file": "./test.pid"
      "watch": false
    }
}
```
* 2、参数说明：
``` js
apps：json结构，apps是一个数组，每一个数组成员就是对应一个pm2中运行的应用
name：应用程序的名称
cwd：应用程序所在的目录
script：应用程序的脚本路径
exec_interpreter：应用程序的脚本类型，这里使用的shell，默认是nodejs
min_uptime：最小运行时间，这里设置的是60s即如果应用程序在60s内退出，pm2会认为程序异常退出，此时触发重启max_restarts设置数量
max_restarts：设置应用程序异常退出重启的次数，默认15次（从0开始计数）
exec_mode：应用程序启动模式，这里设置的是cluster_mode（集群），默认是fork
error_file：自定义应用程序的错误日志文件
out_file：自定义应用程序日志文件
pid_file：自定义应用程序的pid文件
watch：是否启用监控模式，默认是false。如果设置成true，当应用程序变动时，pm2会自动重载。这里也可以设置你要监控的文件。
```

#### 部署（以nuxt为例）

#### 基础模板的部署方式

何为基础模板？使用了` vue init nuxt-community/starter-template <project-name> `进行搭建的！

##### **第一步，打包**

在执行` npm run build `的时候，` nuxt `会自动打包。

##### **第二步，选择要部署的文件（社友最关心的步骤）：**

 - ` .nuxt/ `文件夹
 - ` package.json `文件
 - ` nuxt.config.js `文件(如果你配置proxy等，则需要上传这个文件，建议把它传上去)

##### **第三步，启动你的nuxt：**

使用pm2启动你的nuxt.js：

``` bash
$ npm install // or yarn install 如果未安装依赖或依赖有更改
$ pm2 start npm --name "my-nuxt" -- run start
```
