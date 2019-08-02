---
title: git常用操作
toc: true
date: 2019-01-18 09:36:10
categories:
tags:
---

## 常用操作

1. dev分支测试ok后合并到master分支
```
git checkout dev
git pull
git checkout master
git merge dev
git push -u origin master
```

2. master分支更新后dev1分支同步master分支
```
git checkout master 
git pull 
git checkout dev
git merge master 
git push -u origin dev
```

3. 从dev分支创建release分支，例如要发布2.0版本
```
git checkout -b release-2.0 develop
// change something to commit
git commit -a -m "Bumped version number to 2.0"
git push
```

4. release分支合并到master分支同上1

5. 删除本地release分支
```
git branch -d release-2.0
git push
```

6. 删除远程release分支
```
git push origin --delete release-2.0
```

7. bug分支，master分支上或其他某分支发现bug，则从当前分支上分出来一个branch
```
git checkout -b ISSUE-110 master
// change something
git commit -a -m "Bumped version number to 2.0.1"
//fix bug
git commit -m "Fixed ISSUE-110 problem"
git checkout master
git merge --no-ff ISSUE-110
git tag -a 2.0.1
git checkout develop
git merge --no-ff ISSUE-110
git branch -d ISSUE-110
```

8. 查看分支关系
git log --graph --decorate --oneline --simplify-by-decoration --all

9. 退出git log方法
英文字母Q


