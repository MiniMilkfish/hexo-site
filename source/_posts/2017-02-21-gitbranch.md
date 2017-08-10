---
title: git-branch
date: 2017/02/21
tags: [GIT]
excerpt: 列举下git-branch命令，作为查阅文档
---

# 简介

基本命令：git-branch 。作用为列举已有(包括当前、本地和远程)分支、创建新分支、删除本地或远程分支等。

# SYNOPSIS-概要

```bash
[-r | -a][--list | -l] 
[--color[=<when-] | --no-color] 
[-v [--abbrev=<length- | --no-abbrev]]
[--column[=<options-] | --no-column]
[(--merged | --no-merged | --contains) [<commit-]] [--sort=<key-]
[--points-at <object-] [<pattern-…​]

git branch [--set-upstream | --track | --no-track] [-l] [-f] <branchname- [<start-point>]
git branch (--set-upstream-to=<upstream> | -u <upstream>) [<branchname>]
git branch --unset-upstream [<branchname>]
git branch (-m | -M) [<oldbranch>] <newbranch>
git branch (-d | -D) [-r] <branchname>​
git branch --edit-description [<branchname>]
```

# OPTIONS-参数说明

## 分支查看

- **列出本地所有分支**
 
 ``-l|--list|git branch``

 当前分支可以通过`git status`查看，在这里当前分支前以星号*标识。

- **列出所有的远程分支**

 ``-r|--remote``

- **列出所有的远程和本地分支**

 ``-a|--all``

## 分支创建

- **创建新分支**

 ``git branch <name>``

- **创建并切换到新分支**

 ``git checkout -b <name-``

## 分支删除

- **删除一个分支**

 ``-d|--delete``

 被删除分支必须与他的上游分支(即从哪个分支切出来的)完全合并，或者如果是没有上游分支的分支，分支必须位于HEAD(即当前位置)处

- **`--delete --force`的简写**
 
 ``-D``

> 如上-d可以删除本地分支无疑，但是对于远程分支我们无法进行删除操作。所以当我们想要删除远程分支的时候，可以利用下面这条命令：`git push origin :<branchname>`。这条命令是git push origin <remote-branch-name>:<local-branch-name>的变相用法。

## 分支追踪

- **分支追踪**

 `--track|--set-upstream-to=origin/<branch> <branch>`

 本地分支和远程分支间存在着对应关系，利用这个对应关系可以直接使用git pull或者git push推送当前分支到远程对应分支，而不需要显示加对应关系。

 `git branch <branchname> --track origin/<branchname>`

 该条命令可以在本地创建新分支并且关联到远程分支。

 `git branch --set-upstream-to=origin/<branchname> <branchname>`

 对于未在创建分支时就设置对应追踪关系的，可以在后边利用上边这条命令去显示设置追踪关系。

## 其他

- **重命名分支**

 ``-m|--move``

- **`--move --force`的简写**

 ``-M``

- **强制命令**

 ``-f|--force``

 新建分支时如果某个分支已存在，则该参数可重置该分支到最原始的位置；该参数与-d同时使用时，可以不管分支的合并状态而直接删除；该参数与-m同时使用时，允许重命名分支名，即使新分支名称已经存在了。

- **颜色高亮**

 ``--color[=<when-]``

 高亮分支(按照当前所在、本地和远程区分)名称，when值可以是always(这也是默认值)、never或者auto。

 ``--no-color``

 无颜色高亮功能。相当于`--color=never`。

- **分列展示**

 ``--column[=<options-]|--no-column``

 以分列的形式展示分支列表(默认为一列)。具体展示形式可查看配置文件里的`column.branch`选项。

- **缩略形式**

 ``-v|-vv|--verbose``

 在查看分支列表的时候，该参数可以显示简单的sha1码和每个分支当前位置的提交信息。如果是`-vv`的话，则将对应的上游分支信息也展示出来。

- **缩略位数**

 ``--abbrev=<length>|--no-abbrev``

 分支sha1码缩写的位数或者不缩写进行全部展示。
