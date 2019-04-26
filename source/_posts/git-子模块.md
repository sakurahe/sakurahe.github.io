---
title: git 子模块
date: 2019-04-25 16:18:38
tags:
    - git
---

## 前言

今天在提交hexo源码至[github](https://github.com/sakurahe/sakurahe.github.io/tree/blog)的时候，出现了黄色警告，大概意思就是说在一个Git仓库 中添加 其他 Git 仓库，然后上网查阅了相关资料。主要使用git的 `git submodule` 命令为一个git项目添加子git项目。

## 添加子模块

假设你想添加[hexo-theme-aircloud](https://github.com/aircloud/hexo-theme-aircloud.git)库到你的项目中，使用

```
git submodule add https://github.com/aircloud/hexo-theme-aircloud.git themes/aircloud
```

在这里，themes目录下的aircloud文件夹就是新增加的子模块目录，命令运行完后，会生成一个 `.gitmodules` 的文件，该文件存放的是子模块的信息，使用 `cat` 查看内容为：

```
[submodule "themes/aircloud"]
        path = themes/aircloud
        url = https://github.com/aircloud/hexo-theme-aircloud.git
```

>**.gitmodules文件**：保存项目 URL 与已经拉取的本地目录之间的映射，有多个子模块则含有多条记录，会随着版本控制一起被拉去和推送的。

## 更新子模块

首先需要进入子模块目录，然后执行以下命令：

```
git fetch
git merge origin/master
```

此外，还可以在主目录下，直接用下面的命令更新子模块

```
git submodule update --remote name
```

_注意：上面都是对master分支的操作，无法操作其他分支_

使用下面的方式，更新库的dev分支：

```
git config -f .gitmodules submodule.name.branch dev
git submodule update --remote
```

>这里对 .gitmodules 加了 -f 参数，修改提交后对所有用户有效。

## 删除子模块

有添加，当然也会有删除。以删除 `themes/aircloud` 为例：

- 使用 `git rm --cached themes/aircloud` 将themes/aircloud从版本控制中删除（本地仍保留），若不需要可以不带 `--cached` 进行完全删除
- 使用 `rm -rf .git/modules/themes/aircloud`, 删除.git下的缓存模块，最后提交项目。

经过上面的删除后还可以进行添加子模块。

## 克隆含有子模块的仓库

若需要克隆含有子模块的仓库，直接进行克隆是无法拉取子模块的代码，可加上 `--recursive` 参数，如下：

```
git clone --recursive https://github.com/...
```

或者使用下面的三部操作：

```
git clone  https://github.com/...
git submodule init
git submodule update
```

更多子模块的操作，请参考官方文档：[Git 工具 - 子模块](https://git-scm.com/book/zh/v1/Git-%E5%B7%A5%E5%85%B7-%E5%AD%90%E6%A8%A1%E5%9D%97)
