---
title: Commit message 和 Change log 编写指南
date: 2018-08-07 19:40:44
tags:
    - git
categories:
    - git
---

Git 每次提交代码，都要编写 Commit message（提交说明），否则就不允许提交。

``` bash
$ git commit -m "hello world"
```
上面代码的 -m 参数，就是用来指定 commit mesage 的。如果一行不够，可以只执行 git commit，就会跳出文本编辑器，让你写多行。
``` bash
$ git commit
```
基本上，你写什么都行。

![img1.png](git-commit-change-writing-guide/img1.png)

但是，一般来说，commit message 应该清晰明了，说明本次提交的目的。

![img2.png](git-commit-change-writing-guide/img2.png)

目前，社区有多种 Commit message 的写法规范。本文介绍 Angular 规范，这是目前使用最广的写法，比较合理和系统化，并且有配套的工具。

### Commit message 的作用

#### 提供更多的历史信息，方便快速浏览。

比如，下面的命令显示上次发布后的变动，每个 commit 占据一行。你只看行首，就知道某次 commit 的目的。
``` bash
$ git log <last tag> HEAD --pretty=format:%s
```

#### 可以过滤某些commit（比如文档改动），便于快速查找信息。

比如，下面的命令仅仅显示本次发布新增加的功能。

``` bash
$ git log <last release> HEAD --grep feature
```

#### 可以直接从commit生成Change log。

### Commitizen：撰写合格 Commit message 的工具

Commitizen 是一个撰写合格 Commit message 的工具。使用之前，我们先安装它：

``` bash
$ npm install -g commitizen
```

然后，在项目目录里，运行下面的命令，使其支持 Angular 的 Commit message 格式。

``` bash
$ commitizen init cz-conventional-changelog --save --save-exact
```

以后，凡是用到 git commit 命令，一律改为使用 git cz。这时，就会出现选项，用来生成符合格式的 Commit message。

### commitlint

[commitlint](https://github.com/marionebl/commitlint) 用于检查 Node 项目的 Commit message 是否符合格式。使用前，我们先安装它：

``` bash
# Install commitlint cli and angular config
$ npm install --save-dev @commitlint/{config-conventional,cli}
# For Windows:
$ npm install --save-dev @commitlint/config-conventional @commitlint/cli
```
接着，把这个脚本加入 Git 的 hook。下面是在package.json里面使用 ghooks，把这个脚本加为 commit-msg 时运行。
``` json
{
    "husky": {
        "hooks": {
        "pre-commit": "lint-staged",
            "commit-msg": "commitlint -E HUSKY_GIT_PARAMS"
        }
    }
}
```
然后，每次 git commit 的时候，这个脚本就会自动检查 Commit message 是否合格。如果不合格，就会报错。这里面有使用 husky，所以我们还要安装 [husky](https://github.com/typicode/husky) 才能使用。

``` bash
$ npm install husky@next --save-dev
```

### Change log

如果你的所有 Commit 都符合 Angular 格式，那么发布新版本时， Change log 就可以用脚本自动生成。生成的文档包括以下三个部分：
* New features
* Bug fixes
* Breaking changes.

[conventional-changelog](https://github.com/conventional-changelog/conventional-changelog) 就是生成 Change log 的工具，运行下面的命令即可。
``` bash
$ npm install -g conventional-changelog-cli
$ conventional-changelog -p angular -i CHANGELOG.md -s -w -r 0
```
为了方便使用，可以将其写入 package.json 的 scripts 字段：
``` json
{
    "scripts": {
        "changelog": "conventional-changelog -p angular -i CHANGELOG.md -s -w -r 0"
    }
}
```
以后，直接运行下面的命令即可：
``` bash
$ npm run changelog
```