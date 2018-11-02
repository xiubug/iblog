---
title: 使用git修复线上指定版本的问题
date: 2018-08-05 20:28:38
tags: 
  - fix bug
categories:
  - git
---

作为一个码农，bug 就像家常便饭一样。有了 bug 就需要修复，在 git 中，由于分支是如此的强大，所以，每个 bug 都可以通过一个新的临时分支来修复，修复后，合并分支，然后将临时分支删除。
今天我们就来聊聊如何使用**git修复线上指定版本的问题**？
**第一步：查看远程分支，并确定要修复的分支，如图，远程分支为origin/V1.2.0.**
![img1.png](/images/git-assign-version-fix/img1.png)


如果没有远程分支或不清楚是哪个分支，那我相信在您每开发完一个版本发布生产时都会打包一个标签，就比如我们团队用的 gitlab 管理的项目：
![img2.png](/images/git-assign-version-fix/img2.png)

这边很清楚的能够看到我们有 3 个远程分支，59 个标签，找到对应的标签生成对应的分支即可。如果您们目前尚未使用 gitlab，那只能用 git 命令了，不懂的伙伴，强烈推荐去看[廖大神git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)。

**第二步：创建本地分支V1.2.0，并拉取远程分支代码，同时切换到本地分支，如图**
![img3.png](/images/git-assign-version-fix/img3.png)

**第三步：开发新代码，比如我这边测试代码空格，如图**
![img4.png](/images/git-assign-version-fix/img4.png)

**第四步：开发完成，正常提交流程：git status、git add -A、git commit -m "修复某某问题"，如图**
![img5.png](/images/git-assign-version-fix/img5.png)

**第五步：提交完成，把本地分支推送到远程分支git push origin V1.2.0:V1.2.0**

**第六步：切换到开发分支：git checkout dev，然后合并刚才修改的代码：git merge V1.2.0** 

**最后：删除新创建的分支：git branch -D V1.2.0** 
