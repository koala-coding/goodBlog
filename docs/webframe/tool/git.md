---
title: 如何进阶成公司 Git 小能手(常见问题总结)
date: 2020-03-15
tags:
   - 工具
   - git
---

[[toc]]

## 前言
Git 命令对于程序员的你来说再熟悉不过，但是发现好多小伙伴都是会一些基本的提交流程，当使用过程中遇到问题的时，查到的命令还不敢用，总是请教组里那几个精通 Git 小伙伴。本文对 Git 使用过程中常出现的问题进行总结并且对 Git 的一些误区概念说明了一些，看完后记得自己尝试下，希望你也能成为组里被请教的那 个 Git 小能手。


## Git 经典图

![](../../.vuepress/public/images/170defa8173379ee.jpg)

  一张经典的 Git 流程图(来源阮一峰老师的博客)

### 图中的几个专用名词解释：
1. Workspace: 工作区
2. Index / Stage: 暂存区
3. Repository: 本地仓库
4. Remote: 远程仓库


## git 提交可能遇到的一些问题
### git 想丢弃还没有到暂存区的代码修改，不想一个一个去撤销代码，把当前所有修改代码丢弃

git checkout .

### git 已经提交
### git 提交到本地仓库有问题怎么办？

#### 情况一：最近一次 commit 的代码有问题怎么办？
这时候可能有小伙伴说直接修改再提交一次不就好了，这里说一下优雅的方式，不进行再一次提交，修改这次提交。

```shell
git add 我是修改内容.txt
git commit --amend
```
【amend】修正，会对最新一条 commit 进行修正，会把当前 commit 里的内容和暂存区（stageing area）里的内容合并起来后创建一个新的 commit，用这个新的 commit 把当前 commit 替换掉。

输入上面的命令后，Git 会进入提交信息编辑界面，然后你可以删除之前的 changeId，并且修改或者保留之前的提交信息，:wq 保存按下回车后，你的 commit 就被更新了。

对于 amend 还可能出现几种小问题，下面列举下：

##### 刚刚写的提交信息有问题，想修改怎么办？
```shell
git commit --amend -m "新的提交信息"
```
##### 刚刚提交完代码发现，我有个文件没保存，漏了提交上去怎么办？
最简单的方式，再次 commit：
```shell
git  commit -m "提交信息"
```

另一中方式,使用--no-edit，它表示提交信息不会更改，在 git 上仅为一次提交。

```shell
git add changgeFile // changeFile 刚刚漏了提交的文件
git commit --amend --no-edit
```
#### 情况二：最新提交的代码没问题，它上一次提交的有问题怎么办？
上面说的是最新一次的提交出了问题，接下来说之前提交的代码发现有问题了想修改，应该怎么办？
需要一个新的命令：
```shell
git rebase -i 
```
rebase -i 是 rebase --interactive 的缩写形式，意为「交互式 rebase」。所谓「交互式 rebase」，就是在 rebase 的操作执行之前，你可以指定要 rebase 的 commit 链中的每一个 commit 是否需要进一步修改。

> 注意点：看 commit 历史的时候，最新的提交在最下面，刚开始使用时候总是搞错。

输入上面的命令后，会进入下面的编辑界面。

![](../../.vuepress/public/images/170defc0052e9653.jpg)

根据编辑界面中的提示，我们把要修改的倒数第二个 commit，也就是上面的【修改代码格式首行缩进】前面 pick 指令改为 edit。edit的意思编辑器中已给了解释，应用这个commit，但是停下来修正。改完之后，esc退出，:wq 保存。

会显示如下信息。

![](../../.vuepress/public/images/170defc711daac8c.jpg)

这个rebase过程已经停在倒数第二个 commit 的位置了，修改完成你要修改的内容，再次提交。
```shell
git add .
git commit --amend
```

然后继续 rebase 过程,使用 rebase --continue 来继续 rebase 过程，把后面的 commit 直接应用上去。
```shell
git rabase --continue
```
> 另外在使用git rebase -i 的时候，里面带了不同的指令，都可以对已有的提交进行一些操作，比如 squash 对多个 commit 合并成一个 commit。


#### 情况三：刚刚写完的提交太烂了，不想改了，想直接丢弃怎么办？

你可以用 reset --hard 来撤销 commit
```
git reset --hard HEAD^
```
HEAD 表示 HEAD^ 往回数一个位置的 commit ，HEAD^ 表示你要恢复到哪个 commit。因为你要撤销最新的一个 commit，所以你需要恢复到它的父 commit ，也就是 HEAD^。那么在这行之后，你的最新一条就被撤销了。


### Git 代码已经 push 上去发现有问题
#### 如果出错内容还在本地私有分支
这种情况你修改后，再次提交会报错，由于你在本地对已有的 commit 做了修改，这时你再 push 就会失败，因为中央仓库包含本地没有的 commits。这种情况只在你自己的分支 branch1 ，可以使用强制 push 的方式解决冲突。
```shell
git push origin branch1 -f
```
-f 是 --force 的缩写，意为「忽略冲突，强制 push」

#### 如果出错内容已经 push 到了 master 分支
这种情况可以使用 Git 的 revert 指令。
```shell
git revert HEAD^
```

上面这行代码就会增加一条新的 commit，它的内容和倒数第二个 commit 是相反的，从而和倒数第二个 commit 相互抵消，达到撤销的效果。

在 revert 完成之后，把新的 commit 再 push 上去，这个 commit 的内容就被撤销了。

> revert 与前面说的 reset 最主要的区别是，这次改动只是被「反转」了，并没有在历史中消失掉，你的历史中会存在两条 commit ：一个原始 commit ，一个对它的反转 commit。


## Git 关于暂存的问题
假如正在开发手中需求的时候，突然来了个紧急 bug 要修复，这时候需要先 stash 已经写的部分代码，使自己返回到上一个 commit 改完 bug 之后从缓存栈中推出之前的代码，继续工作。
- 添加缓存栈: git stash
- 查看缓存栈: git stash list
- 推出缓存栈: git stash pop
- 取出特定缓存内容：git stash apply stash@{1}

> 注意：没有被 track 的文件（即从来没有被 add 过的文件不会被 stash 起来，因为 Git 会忽略它们。如果想把这些文件也一起 stash，可以加上 `-u` 参数，它是 `--include-untracked` 的简写。就像这样：`git stash -u`

  

## Git 分支相关问题
分支中的常用命令：
- git 拉取指定分支的代码： git clone -b 分支名称 地址

- 查看当前分支：git branch

- 查看远程分支：git branch -a

- 创建并切换分支：git checkout -b add_orderdesc

- 切换分支：git checkout 分支名称

- 查看当前的本地分支与远程分支的关联关系：git branch -vv

- 合并当前分支代码到master：

### 问题1：我想把本地创建的一个分支 koalanode提交到远程，并且远程分支名称要求 nodescript，且还未创建，需要怎能做？


1. 我先在远程建了一个分支 nodescript，我本地也有这么一个分支，名字和远程的分支名称还不一样。首先，我把我本地的分支名称修改成和远程分支相同。

2. 将本地新建分支push到自己的本地远程origin上，因为只在本地创建了一个新的分支，远程    origin 上还没有该分支
```shell
git push origin nodescript
```
3. 把本地分支与远程origin的分支进行关联处理(通过 --set-upstream-to 命令)

```shell
git branch --set-upstream-to=origin/add_orderdesc
```

4. 再次通过 `git branch -vv` 查看分支的关联关系，可见本地分支已于origin的分支建立上了关联关系,之后我们每次 push 或者 pull 的时候，只需要输入git push 或者git pull 


## git 用户名密码邮箱相关问题

### 公司仓库有账号密码，自己的github有账户密码，两个不同账户，有一次提交发现自己仓库的邮箱提交成了公司仓库设置的邮箱，有点尴尬，为什么会出现这种问题呢？

首先这个你在刚开始安装一趟的时候应该就用过:
``` shell
// 设置查看 git 用户名和邮箱
git config user.name   --查看git当前配置用户名
git config user.email  --查看git当前配置的邮箱
git config user.name 名称 设置用户名
git config user.email 邮箱 设置git邮箱
```
全局命令设置
``` shell
 git config  --global user.name 你的目标用户名；

 git config  --global user.email 你的目标邮箱名;
```

在项目中也可以查看这些信息 
```shell
vi ~/.gitconfig;
```
知道了这些配置修改之后，你可以选择全局配置下，在公司电脑，或者提交前自己看下，就不会再出现上面的尴尬问题了。


### 为什么我每次提交都需要输入github的密码，这个步骤如何省去？


## 提交相关问题
如何修改已经提交的内容

## 几个开发中常用的命令没有做分类
1. git checkout . 和 git checkout --filename
测试服务器测试时候，发现有一个bug，临时修改下里面的代码，测试一下问题是否解决，再去git仓库拉取新的代码时候，可以用上面的命令忽略掉刚才的修改

网上的说法是：git checkout -- filename的作用是把filename文件在工作区的修改撤销到最近一次git add 或 git commit时的内容



2. 如何修改已经提交到远程的代码
场景：提交了代码gerrit ，领导看了代码之后，发现有一些问题需要修改，需要做一些修改，再次提交。
查看当前分支你提交上次一到节点commitid是多少，这种情况可以通过工具看，也可以通过git log 命令查看

去gerrit中Abandon，然后执行
git reset 1dc340d792deace324e1ee9ec2f9f0a69d22b9c0，注意这个id是你当前分支上一个提交节点，然后会发现代码恢复到了提交前状态，再次修改问题，重新提交代码就可以，还是比较常见的操作。

3. 

## 关于 merge 可能出现的问题



## git 工作流

## git 提交规范
这里只列举我常用的提交格式类型
- feat：新功能（feature）
- fix：修补bug
- docs：文档（documentation）
- style： 格式（不影响代码运行的变动）
- refactor：重构（即不是新增功能，也不是修改bug的代码变动）
- test：增加测试
- chore：其他修改，比如构建过程或辅助工具的变动

git 更详细的提交规范可以看一下阮一峰老师的这篇文章，非常棒
[Commit message 和 Change log 编写指南](https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html)


## Git 工具
网上一些 Git 工具很多，推荐一个SourceThree，但是个人还是比较喜欢用  Git 命令，不然你会发现这项技能慢慢蜕化了，而且一些工具提交时总是带一些我不懂的参数。平时工具和 Git 命令配个使用，用它的可视化图表看提交是否有问题，当然也有公司觉得用命令没有工具安全的，自己选择就好了，嘿嘿。

SourceThree 下载地址：https://www.sourcetreeapp.com/

## vim 常用命令
使用 Git 的时候，偶尔会对 Vim 中对 shell 脚本进行简单操作，为了节约时间，列出几个常用的 vim 快捷命令。
- a,i,r,o,A,I,R,O	进入编辑模式
- :q	一般退出
- :q!	退出不保存
- :wq	保存退出
- yy	复制当前行的内容
- ZZ	保存离开
- dd	删除光标当前行

## 总结
本文对 Git 使用过程中常出现的问题进行了一个总结，后面遇到问题也会不断更新，可以收藏本文，需要的时候查阅，最后建议在记忆的时候围绕文初的图片，多使用。希望本文能帮助到小伙伴们。



参考文章：
vim 常用快捷键大全
https://www.jianshu.com/p/dde77e3b299f
git 提交规范
https://www.ruanyifeng.com/blog/2016/01/commit_message_change_log.html

附件：
一张git常用图片