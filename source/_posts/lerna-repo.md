---
title: lerna管理前端packages的最佳实践
date: 2018-07-21 20:55:06
tags:
  - package
categories:
  - lerna
---

## 背景

对于维护过多个package的同学来说，都会遇到一个选择：**这些package是放在一个仓库里维护还是放在多个仓库里单独维护**，数量较少的时候，多个仓库维护不会有太大问题，但是当package数量逐渐增多时，一些问题逐渐暴露出来：
**一、**package之间相互依赖，开发人员需要在本地手动执行**npm link**，维护版本号的更替；
**二、**issue难以统一追踪，管理，因为其分散在独立的repo里；
**三、**每一个package都包含独立的node_modules，而且大部分都包含babel,webpack等开发时依赖，安装耗时冗余并且占用过多空间。

## 什么是lerna
lerna到底是什么呢？lerna官网上是这样描述的。
> 用于管理具有多个包的JavaScript项目的工具。
这个介绍可以说很清晰了，引入lerna后，上面提到的问题不仅迎刃而解，更为开发人员提供了一种管理多packages javascript项目的方式。
**一、**自动解决packages之间的依赖关系。
**二、**通过git 检测文件改动，自动发布。
**三、**根据git 提交记录，自动生成CHANGELOG

### 常用命令
#### 全局安装lerna
lerna 我们需要全局安装lerna工具。
``` bash
$ npm i -g lerna
# 或
$ yarn global add lerna
```

#### 为所有项目安装依赖，类似于npm/yarn i
``` bash
$ lerna bootstrap
```

#### 提交对项目的更新
运行该命令会执行如下的步骤：
1. 运行lerna updated来决定哪一个包需要被publish
2. 如果有必要，将会更新lerna.json中的version
3. 将所有更新过的的包中的package.json的version字段更新
4. 将所有更新过的包中的依赖更新
5. 为新版本创建一个git commit或tag
6. 将包publish到npm上

``` bash
$ lerna publish # 用于发布更新
$ lerna publish --skip-git # 不会创建git commit或tag
$ lerna publish --skip-npm # 不会把包publish到npm上
```

#### 使用lerna 初始化项目
``` bash
$ lerna init # 固定模式(Fixed mode)默认为固定模式，packages下的所有包共用一个版本号(version)
$ lerna init --independent # 独立模式(Independent mode)，每一个包有一个独立的版本号
```

#### 为packages文件夹下的package安装依赖
``` bash
$ lerna add <package>[@version] [--dev] # 命令签名
$ lerna add babel # 该命令会在package-1和package-2下安装babel
$ lerna add react --scope=package-1 # 该命令会在package-1下安装react
$ lerna add package-2 --scope=package-1 # 该命令会在package-1下安装package-2
```

#### 对包是否发生过变更
``` bash
$ lerna updated
# 或
$ lerna diff
```

#### 显示packages下的各个package的version
``` bash
$ lerna ls
```

#### 清理node_modules
``` bash
$ lerna clean
```

#### 运行npm script，可以指定具体的package
``` bash
$ lerna run
```

## lerna.json解析
``` json
{
  "version": "1.1.3",
  "npmClient": "npm",
  "command": {
    "publish": {
      "ignoreChanges": [
        "ignored-file",
        "*.md"
      ]
    },
    "bootstrap": {
      "ignore": "component-*",
      "npmClientArgs": ["--no-package-lock"]      
    }
  },
  "packages": ["packages/*"]
}
```
**version：**当前库的版本
**npmClient：** 允许指定命令使用的client， 默认是 npm， 可以设置成 yarn
**command.publish.ignoreChanges：**可以指定那些目录或者文件的变更不会被publish
**command.bootstrap.ignore：**指定不受 bootstrap 命令影响的包
**command.bootstrap.npmClientArgs：**指定默认传给 lerna bootstrap 命令的参数
**command.bootstrap.scope：**指定那些包会受 lerna bootstrap 命令影响
**packages：**指定包所在的目录

## 使用lerna的基本工作流
### 环境配置
* Git 在一个lerna工程里，是通过git来进行代码管理的。所以你首先要确保本地有正确的git环境。 如果需要多人协作开发，请先创建正确的git中心仓库的链接。 因此需要你了解基本的git操作，在此不再赘述。
* npm仓库 无论你管理的package是要发布到官网还是公司的私有服务器上，都需要正确的仓库地址和用户名。 你可运行下方的命令来检查，本地的npm registry地址是否正确。
``` bash
$ npm config ls
```
* lerna 我们需要全局安装lerna工具
``` bash
$ npm i -g lerna
# 或
$ yarn global add lerna
```

### 初始化一个lerna工程
> 在这个例子中，我将在我本地d:/jobs 根目录下初始化一个lerna工程。
1、在`d:/jobs`下创建一个空的文件夹，命名为`lerna-demo`：
``` bash
$ mkdir lerna-demo
```
2、初始化 通过cmd进入相关目录，进行初始化
```bash
$ cd d:/jobs/lerna-demo
$ lerna init
```
执行成功后，目录下将会生成这样的目录结构。
```
- packages(目录)
- lerna.json(配置文件)
- package.json(工程描述文件)
```
3、添加一个测试package
> 默认情况下，package是放在packages目录下的。
``` bash
# 进入packages目录
cd d:/jobs/lerna-demo/packages
# 创建一个packge目录
mkdir module-1
# 进入module-1 package目录
cd module-1
# 初始化一个package
npm init -y
```
执行完毕，工程下的目录结构如下:
```
--packages
	--module-1
		package.json
--lerna.json
--package.json
```
4、安装各packages依赖 这一步操作，官网上是这样描述的
> 在当前的Lerna仓库中引导包。安装所有依赖项并链接任何交叉依赖项。
``` bash
$ cd d:/lerna-demo
$ lerna bootstrap
```
在现在的测试package中，module-1是没有任何依赖的，因此为了更加接近真实情况。你可已在module-1的`package.json`文件中添加一些第三方库的依赖。 这样的话，当你执行完该条命令后，你会发现module-1的依赖已经安装上了。

5、发布 在发布的时候，就需要`git`工具的配合了。 所以在发布之前，请确认此时该lerna工程是否已经连接到git的远程仓库。你可以执行下面的命令进行查看。
``` bash
git remote -v
// print log
origin  git@github.com:meitianyitan/docm.git (fetch)
origin  git@github.com:meitianyitan/docm.git (push)
```
本篇文章的代码托管在Github上。因此会显示此远程链接信息。 如果你还没有与远程仓库链接，请首先在github创建一个空的仓库，然后根据相关提示信息，进行链接。
``` bash
$ lerna publish
```
执行这条命令，你就可以根据cmd中的提示，一步步的发布packges了。
实际上在执行该条命令的时候，lerna会做很多的工作。
``` 
-  Run the equivalent of  `lerna updated`  to determine which packages need to be published.
-  If necessary, increment the  `version`  key in  `lerna.json`.
-  Update the  `package.json`  of all updated packages to their new versions.
-  Update all dependencies of the updated packages with the new versions, specified with a  [caret (^)](https://docs.npmjs.com/files/package.json#dependencies).
-  Create a new git commit and tag for the new version.
-  Publish updated packages to npm.
```
到这里为止，就是一个最简单的lerna的工作流了。但是lerna还有更多的功能等待你去发掘。
lerna有两种工作模式,Independent mode和Fixed/Locked mode，在这里介绍可能会对初学者造成困扰，但因为实在太重要了，还是有必要提一下的。
lerna的默认模式是Fixed/Locked mode，在这种模式下，实际上lerna是把工程当作一个整体来对待。每次发布packges，都是全量发布，无论是否修改。但是在Independent mode下，lerna会配合Git，检查文件变动，只发布有改动的packge。

## 使用lerna提升开发流程体验
接下来，我们从一个demo出发，了解基于lerna的开发流程。

### 项目初始化
![img1.jpg](/images/lerna-repo/img1.jpg)

我们需要维护一个UI组件库，其包含2个组件，分别为House（房子）和Window（窗户）组件，其中House组件依赖于Window组件。
我们使用lerna初始化整个项目，并且在packages里新建了2个package，执行命令进行初始化：
``` bash
$ lerna init
```
初始化化后目录结构如下所示：
``` js
.
├── lerna.json
├── package.json
└── packages
    ├── house
    │   ├── index.js
    │   ├── node_modules
    │   └── package.json
    └── window
        ├── index.js
        ├── node_modules
        └── package.json
```

### 增加依赖
![img2.jpg](/images/lerna-repo/img2.jpg)

接下来，我们来为组件增加些依赖，首先House组件不能只由Window构成，还需要添加一些外部依赖（在这里我们假定为lodash）。我们执行：
``` bash
$ lerna add lodash --scope=house
```

这句话会将lodash增添到House的dependencies属性里，这会儿你可以去看看package.json是不是发生变更了。

我们还需要将Window添加到House的依赖里，执行：
``` bash
$ lerna add window --scope=house
```

就是这么简单，它会自动检测到window隶属于当前项目，直接采用symlink的方式关联过去。

> symlink:符号链接，也就是平常所说的建立超链接，此时House的node_modules里的Window直接链接至项目里的Window组件，而不会再重新拉取一份，这个对本地开发是非常有用的。

### 发布到npm
![img3.jpg](/images/lerna-repo/img3.jpg)

接下来我们只需要简单地执行lerna publish，确认升级的版本号，就可以批量将所有的package发布到远程。

> 默认情况下会推送到系统目前npm对应的registry里，实际项目里可以根据配置leran.json切换所使用的npm客户端。

### 更新模块
![img4.jpg](/images/lerna-repo/img4.jpg)

接下来，我们变更了Window组件，执行一下lerna updated，便可以得知有哪些组件发生了变更。
``` js
→ lerna updated
lerna info version 2.9.1
lerna info Checking for updated packages...
lerna info Comparing with v1.0.9.
lerna info Checking for prereleased packages...
lerna info result
- jx-house
- jx-window
```
我们可以看到，虽然我们只变更了window组件，但是lerna能够帮助我们检查到所有依赖于它的组件，对于没有关联的组件，是不会出现在更新列表里的，这个对于相比之前人工维护版本依赖的更新，是非常稳健的。

### 集中版本号或独立版本号
截止目前，我们已经成功发布了2个package，现在再新增一个Tree组件，它和其他2个package保持独立，随后我们执行lerna publish，它会提示Tree组件的版本号将会从0.0.0升级至1.0.0，但是事实上Tree组件仅仅是刚创建的，这点不利于版本号的语义化，lerna已经考虑到了这一点，它包含2种版本号管理机制。

* fixed模式下，模块发布新版本时，都会升级到leran.json里编写的version字段
* independent模式下，模块发布新版本时，会逐个询问需要升级的版本号，基准版本为它自身的package.json，这样就避免了上述问题。

如果需要各个组件维护自身的版本号，那么就使用independent模式，只需要去配置leran.json即可。

### 总结
![img4.jpg](/images/lerna-repo/img4.jpg)
lerna不负责构建，测试等任务，它提出了一种集中管理package的目录模式，提供了一套自动化管理程序，让开发者不必再深耕到具体的组件里维护内容，在项目根目录就可以全局掌控，基于npm scripts，可以很好地完成组件构建，代码格式化等操作，并在最后一公里，用lerna变更package版本，将其上传至远端。

## lerna最佳实践
为了能够使lerna发挥最大的作用，根据这段时间使用lerna 的经验，总结出一个最佳实践。下面是一些特性。
* 采用Independent模式
* 根据Git提交信息，自动生成changelog
* eslint规则检查
* prettier自动格式化代码
* 提交代码，代码检查hook
* 遵循semver版本规范
大家应该也可以看出来，在开发这种工程的过程的，最为重要的一点就是规范。因为应用场景各种各样，你必须保证发布的packge是规范的，代码是规范的，一切都是有迹可循的。这点我认为是非常重要的。

## 工具整合
在这里引入的工具都是为了解决一个问题，就是工程和代码的规范问题。
* husky
* lint-staged
* prettier
* eslint

## yarn workspaces

### 命令

#### 在根目录安装 npm 包，以 danger 为例：
``` bash
$ yarn add danger --dev -W
``` 
