---
title: 开始-用hexo+Travis部署独立博文
date: 2016-9-25
categories: 
- hexo
tags: 
- hexo
toc: true
---

# 简述
需要安装：
- [node](https://nodejs.org/en/download/) :选择Windows Installer (.msi)，32bit/64bit根据系统自行选择。
安装完成后就可以用npm进行其他插件的安装了，安装时以管理员身份安装。
- hexo 
- [git](https://git-scm.com/)
- [Ruby](http://rubyinstaller.org/downloads/)
- Travis CI  

<!--more-->

需要注册：
- github

## nodejs安装
下载适合的[hexo](https://nodejs.org/en/download/)，右键以管理员身份运行，之后选择默认安装即可。  
``` bash
$ node -v
$ v.5.0
```
表示安装成功。进入下一步。

## hexo安装
``` bash
$ npm install hexo --save
```
hexo初始化，我在D盘新建了文件夹hexo，进入文件夹，按下shift+右键>在此处打开命令窗口。  
``` bash
$ hexo init 
```
>如果使用git方式进行部署，执行npm install hexo-deployer-git --save来安装所需的插件
``` bash
$ npm install hexo-deployer-git --save
```
然后打开_config文件，找到最后，做以下更改：
``` bash
$ # Deployment
$ ## Docs: https://hexo.io/docs/deployment.html
$ deploy:
$  type: git
$  repo: https://github.com/XuBovey/xubovey.github.io.git
$  branch: master 
```
git配置身份信息:  
``` bash
$ git config --global user.email "you@example.com"
```
``` bash
$ git config --global user.name "Your name"
```

## hexo预览/发布
生成/预览/发布，命令如下：
``` bash
$ hexo g //生成
$ hexo s //预览，hexo s --debug可以调试
$ hexo d //发布
```
执行`hexo s`后，在浏览器中输入[http://localhost:4000/.](http://localhost:4000/.)即可预览编辑的页面。  
因为前面已经配置过git的身份信息，因此可以直接发布，输入命令：
``` bash
$ hexo d
```

## 添加新博文
执行命令：
``` bash
$ hexo new test //test为新添加文章的名字
```
然后可以在根目录下的source/_post文件夹找到test.md文件，打开后内容如下：
``` bash
$ ---
$ title: test
$ date: 2016-09-25 21:32:53
$ tags:
---
```

## 主题配置
hexo有很多主题可供选择，[知乎](https://www.zhihu.com/question/24422335)
上有相关的哪些好的提问，选择喜欢的主题参考以下命令git到本地。  
``` bash
$ git clone https://github.com/iissnan/hexo-theme-next.git themes/next
```
安装与配置步骤创建的hexo文件夹下的_config文件，做以下修改：  
``` bash
$ # Extensions
$ ## Plugins: https://hexo.io/plugins/
$ ## Themes: https://hexo.io/themes/
$ #theme: landscape
$ theme: next
```
关于主题的配置可以参考next的[说明文档](http://theme-next.iissnan.com/getting-started.html)，比较详细。
阅读说明文件时只强调一点：其中的站点配置未见就是刚才用的_config文件。而主题中的_config才是主题配置文件。
> 为了描述方便，在以下说明中，将前者称为 站点配置文件， 后者称为 主题配置文件。  

至此已经可以了，只要每次添加新的文章后执行如下命令即可完成，生成和发布：
``` bash
$ hexo g -d
```

## 走的更远
### 不同电脑如何进行同步
### 用Travis进行自动更新github pages

# 参考链接
- [史上最详细的Hexo博客搭建图文教程](https://xuanwo.org/2015/03/26/hexo-intor/)
- [hexo不同风格](https://hexo.io/themes/)
- [hexo不同风格比较-看家顺张的回答](https://www.zhihu.com/question/24422335)
- [献给写作者的 Markdown 新手指南](http://www.jianshu.com/p/q81RER)
- [使用 Travis CI 自动更新 GitHub Pages](http://notes.iissnan.com/2016/publishing-github-pages-with-travis-ci/)
- [用markdown写博客：hexo + gitcafe](https://wizardforcel.gitbooks.io/markdown-simple-world/content/7.html)
