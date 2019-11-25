---
title: 常用git命令总结
date: 2019-11-25 20:15:24
author:  "Vinecnt Ko"
tags:
    - git
---

> 记录在日常使用git时遇到的问题和解决方案

除了最频繁的`git pull` 、 `git push` 等操作，在工作和日常使用中，还会遇到各种各样的问题和情景，在这里做一些记录，方便总结、查看



### 1. Your branch is ahead of 'origin/master' by 3 commits

I am getting the following when running `git status`

```
Your branch is ahead of 'origin/master' by 3 commits.
```

I have read on some other post the way to fix this is run `git pull --rebase` but what exactly is rebase, will I lose data or is this simple way to sync with master?



### answers

------

You get that message because you made changes in your local master and you didn't push them to remote. You have several ways to "solve" it and it normally depends on how your workflow looks like:

- In a good workflow your remote copy of master should be the good one while your local copy of master is just a copy of the one in remote. Using this workflow you'll never get this message again.
- If you work in another way and your local changes should be pushed then just `git push origin`assuming origin is your remote
- If your local changes are bad then just remove them or reset your local master to the state on remote **git reset --hard origin/master**

------

### 2. git修改远程仓库地址

方法有三种：

1. 修改命令

   ```
   git remote set-url origin [url]
   ```

2. 先删后加

   ```
   git remote rm origin
   git remote add origin [url]
   ```

3. 直接修改config文件

### 3. git放弃修改&放弃增加文件

1. 本地修改了一堆文件(并没有使用git add到暂存区)，想放弃修改。 
   单个文件/文件夹：

```bash
$ git checkout -- filename
```

​	所有文件/文件夹：

```bash
$ git checkout .
```

2. 本地新增了一堆文件(并没有git add到暂存区)，想放弃修改。 
   单个文件/文件夹：

```bash
$ rm filename / rm dir -rf
```

所有文件/文件夹：

```bash
$ git clean -xdf
// 删除新增的文件，如果文件已经已经git add到暂存区，并不会删除！
```

3. 本地修改/新增了一堆文件，已经git add到暂存区，想放弃修改。 
   单个文件/文件夹：

```bash
$ git reset HEAD filename
```

​	所有文件/文件夹：

```bash
$ git reset HEAD .
```

4. 本地通过git add & git commit 之后，想要撤销此次commit

```bash
$ git reset commit_id
//这个id是你想要回到的那个节点，可以通过git log查看，可以只选前6位 
// 撤销之后，你所做的已经commit的修改还在工作区！
```

```bash
$ git reset --hard commit_id
//这个id是你想要回到的那个节点，可以通过git log查看，可以只选前6位 
// 撤销之后，你所做的已经commit的修改将会清除，仍在工作区/暂存区的代码不会清除！
```

### 4. 删除分支

首先查看项目的分支（包括本地和远程）

```bash
$ git branch -a
```

1. 删除本地分支

```bash
$ git branch -d <branchname>
```

2. 删除远程分支

```bash
$ git push origin --delete <branchname>
```



### 5.删除远程文件

> 项目开发初期由于`.gitignore` 文件配置不正确很有可能导致某些不需要的目录上传到 git 远程仓库上了，这样会导致每个开发者提交的时候这些文件每次都会不同。除了一开始提交的时候注意配置好 `.gitignore` 文件外，我们也需要了解下出现这种问题后的解决办法

具体操作步骤如下：

1. 预览将要删除的文件

   ```
   git rm -r -n --cached 文件/文件夹名称 
   
   加上 -n 这个参数，执行命令时，是不会删除任何文件，而是展示此命令要删除的文件列表预览。
   ```

2. 确定无误后删除文件

   ```
   git rm -r --cached 文件/文件夹名称
   ```

3. 提交到本地并推送到远程服务器

   ```
   git commit -m "提交说明"
   git push origin master
   ```

4. 修改本地 .gitignore 文件 并提交

   ```
     git commit -m "提交说明"
     git push origin master
   ```

### 6. gitlab或github下fork后如何同步源的新更新内容？

> gitlab或github下，a开发者fork了b开发者的项目后，如果b开发人员更新代码后，a开发者如何获得更新？

具体步骤如下:

1. 给fork配置远程库

```bash
gir remote -v
```

查看远程状态

- 确定一个将被同步给 fork 远程的上游仓库

```bash
git remote add upstream URL
```

2. 同步fork

- 从上游仓库 fetch 分支和提交点，提交给本地 master，并会被存储在一个本地分支upstream/master 

```bash
git fetch upstream
```

- 切换到本地主分支

```bash
git checkout master
```

- 把 upstream/master 分支合并到本地 master 上，这样就完成了同步，并且不会丢掉本地修的内容

```bash
git merge upstream/master
```



### 7. 对本地commit进行还原 

```bash
git reset --soft HEAD~1
```

注：reset命令基于最近一次提交，多次执行会回到更早以前，这可能会超出预期。

### 8. 放弃修改，远程覆盖本地代码

与问题1一样，本地超前远程分支，但同时不想要本地的修改，在复习下

在使用Git的过程中，有些时候我们只想要git服务器中的最新版本的项目，对于本地的项目中修改不做任何理会

```bash
git fetch --all
git reset --hard origin/master
git pull
```

