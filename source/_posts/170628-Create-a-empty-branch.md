---
title: 170628-Create_a_empty_branch_under_git
toc: true
date: 2017-06-28 20:15:51
categories:
tags:
---

参考链接

Creating Project Pages manually
作者: Volcano 发表于October 15, 2012 at 3:38 pm

版权信息: 可以任意转载, 转载时请务必以超链接形式标明文章原始出处和作者信息及此声明

永久链接 - http://www.ooso.net/archives/636

有时候我们需要在Git下创建一个空分支，从头开始Coding —— 这大概是那些重构帝最喜欢的事情。参考了github，才找到一个合适的方法。

怎样安全的进行这项操作

我们需要建一个“孤立”的空分支，为了尽可能的保证数据安全，最好还是重新clone一份代码。
```
$git clone https://github.com/user/repo.git
# Clone our repo

# Cloning into \'repo\'...
# remote: Counting objects: 2791, done.
# remote: Compressing objects: 100% (1225/1225), done.
# remote: Total 2791 (delta 1722), reused 2513 (delta 1493)
# Receiving objects: 100% (2791/2791), 3.77 MiB | 969 KiB/s, done.
# Resolving deltas: 100% (1722/1722), done.
```
开工

这里以github的操作为例，下面试图创建一个名为gh-pages的空分支
```
$cd repo

$ git checkout --orphan gh-pages
# 创建一个orphan的分支，这个分支是独立的
Switched to a new branch \'gh-pages\'

git rm -rf .
# 删除原来代码树下的所有文件
rm \'.gitignore\'
```

注意这个时候你用git branch命令是看不见当前分支的名字的，除非你进行了第一次commit。

下面我们开始添加一些代码文件，例如这里新增了一个index.html
```
$ echo \"My GitHub Page\" > index.html
$ git add .
$ git commit -a -m \"First pages commit\"
$ git push origin gh-pages
```
在commit操作之后，你就可以用git branch命令看到新分支的名字了，然后push到远程仓库。
