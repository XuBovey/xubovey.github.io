---
title: chrome自动打开ahlqs bengpala 51m33
toc: true
date: 2017-02-08 09:37:39
categories:
tags:
---

# 背景
网上下载了个破解工具，然后还无视windows defender提示的有问题。结果...杯了个具。

经过很多次的折腾...
<!-- more -->

# 卸载、删除
卸载了当前安装的所有软件。不能卸载的看文件或文件夹的更新时间如果是当前的考虑是否删除，总之删除了很多，当然可能有的是不需要的。
嗯嗯，中间不能删除的就没管了，结果。

# 顽固之主
ahlqs bengpala 51m33这三个网页总是在chrome打开，无论修改默认浏览器或chrome重置都无效。最后在这个[网页](https://productforums.google.com/forum/#!topic/chrome/WF6jm5fGwBo)找到了解决办法。  
> Deactivate "msiql" in starting programs at task manager. Then delete application by going in its folder. Worked for me.  

怎么操作呢：
1. 打开任务管理器，找到msiql这个任务
2. 在任务上右键选择打开文件位置
3. 结束msiql任务
4. 删除msiql文件所在文件夹。

因为被折磨的比较厉害，所以当我发现文件是在local/tmp目录下的时候就残忍的把所有文件都删除了，这个文件之前已经被删除过，那些不能删的就没动。

# 教训
这个就是用盗版，还不停杀毒软件提醒的后果。_-_
