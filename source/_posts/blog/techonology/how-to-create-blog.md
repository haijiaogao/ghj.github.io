---
title: 使用hexo.js + travis.cli + Next.js主题在Github上搭建个人博客
date: 2020-07-14 15:08:42
categories:
- Techonology
- Node.js

tags: 
- Node.js
- hexo.js 
- travis.cli
- Next.js
- Github
---

## 一、动因
笔者最近很闲，于是把很多之前在ToDoList上的计划提上了日程，其中就包含搭建自己的个人blog。又由于笔者算是互联网从业者，于是选择在自己的github搭建，而不是直接写在其他的技术blog上面。

## 二、准备

1. 首先，github账户一定要基本具备
2. 考察网上成型方案，当然笔者在此走了许多的弯路，下面会详细记录笔者遇到的若干小白问题。
3. 开动

## 三、方案1-基于Jekyll
基于Jekyll的方案是Github Page直接支持的方案，该方案简单，步骤少，配置少，基于markdown。但是我最终没有采用该方式。主要的原因是因为主题少，插件少，而且在本地运行的时候，需要安装ruby相关工具，ruby并不在我的技术栈，以及我想了解的技术栈之内，故放弃之。

Jekyll方案指南:[Jekyll方案指南](https://pages.github.com/)

其中有个细节点需要注意：
在新建repo库的时候，如果说想以username.github.io/进行访问，则库名必须为username.github.io。

示意：![repo库必须为`username.github.io`](images/create-repo.png)


## 四、方案2-基于node.js的hexo.js+Next.js主题方案

方案一很好，入门方便，但是我在preview的时候总是对主题不太满意，之前看到过一些别人的blog的风格非常的简洁，搜了搜，是基于Next.js的，所以为了提高难度，改为用node.js+hexo.js+Next.js的方案二进行博客的搭建。

网上一搜教程一堆，正好发现了github 目前增加了自动化构建相关的内容，一并练练手，反正闲着也是闲着。

废话有点多，具体步骤如下：

#### 1. 安装node：我的mac很久以前安装过node.js大概是3年前的旧事了，所以node版本很老，大概是6.x的版本，查看了一下官网上的稳定版都已经更新到了12.x而且，hexo的新版最低要求也是8.x的，所以我是从更新node开始的。

>安装（Mac os可以直接使用brew命令，其他平台参考node.js官网）
``` bash
brew install node
```
>更新(我没有使用brew，而是通过一个nvm脚本)
``` bash
 curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash

nvm list

nvm install 12.18.2

```

#### 2. 安装hexo-cli

``` bash
npm install hexo-cli -g
```

#### 3. 使用hexo初始化一个项目,我的叫blog
```
hexo init blog
```
接下来就可以用hexo相关的命令进行blog的配置和博客书写啦

#### 4. 使用travis-cli实现github repo库的自动编译

> 需要注意的一点，使用travis-cli后会自动把主分支（假设hexo项目是push到主分支）上的hexo项目编译生成的public目录内容push到一个新分支，默认叫gh-pages。但是如果按照方案一的repo name命名后，github pages默认只能使用master分支的内容。so，此处repo库一定不要命名成 ~~username.github.io~~，以笔者为例，笔者命名为`blogs`

> [travis](https://hexo.io/zh-cn/docs/github-pages) 配置githubpages指南,参考链接，需要涉及到github的Token获取，客户端blog项目中增加travis.yml配置等内容，此处略过不提


配置travis的过程算是顺利，但是，当我把项目代码push 到master分支后，我发现travis编译失败了，而且一直是失败在了yarn下载package配置的地方，经过网上高人指点，我在项目中删除掉了`package-lock.json`以及`yarn-lock.json`两个配置文件，并且修改了`travis.yml`配置，最终终于编译成功^ ^

我的travis.yml如下：
``` travis.yml
sudo: false
language: node_js
node_js:
  - 10 # use nodejs v10 LTS
cache: npm
branches:
  only:
    - master # build master branch only
before-install:  # 我改了这里
  - npm install -g hexo-cli # 我改了这里
install: # 我改了这里
  - npm install  # 我改了这里
  - npm install hexo-deployer-git --save  # 我改了这里
script:
  - hexo clean # 我改了这里
  - hexo generate # generate static files
deploy:
  provider: pages
  skip-cleanup: true
  github-token: $GH_TOKEN
  keep-history: true
  on:
    branch: master
  local-dir: public

```

[Github提交](https://github.com/haijiaogao/blogs/commit/6586f08a04dc01647bc6be4614d03e22d847356f)


#### 5.使用hexo-deployer-git 实现私有库编译

* 安装[hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)
```
npm install hexo-deployer-git --save
```

* 配置_config.yml

```
deploy:
  type: git
  repo: <repository url> # https://bitbucket.org/JohnSmith/johnsmith.bitbucket.io
  branch: [branch]
  message: [message]
```

* 胜利只差一步执行

```
hexo clean && hexo deploy -m "deploy message "
```

#### 6. [Next.js](https://github.com/iissnan/hexo-theme-next)主题安装以及配置

该主题非常的简洁，而且包含很多插件，有兴趣可以鼓捣鼓捣，应该还挺有意思
具体设置步骤在github中写的非常详尽，此处不再复述。

说一下我遇到的问题吧，在使用了next主题之后，发现顶部的home和archives以及categories几个跳转菜单，都跳不过去，本地调试不行，放在github中也不行，后面总是有几个奇怪的字符串，然后报404的错误。

* **Round 1**:

我开始以为是favico的问题，因为在chrome的调试窗口下，会报这个图标404，一顿操作猛如虎之后，图标确实陪好了，可这几个链接，还是不行

* **Round 2**

我又猜测是我的_config.yml中配置的permlink的问题。好吧，该猜测实在是毫无根据。所以我又失败了

* **Round 3**

莫非是主题的问题？我决定换回landscape主题试一下！奇迹般的可以跳过去！！
所以，Next主题到底哪里出了问题呢，我在github中搜到了相关的[issue](https://github.com/theme-next/hexo-theme-next/issues/2)，最后，我发现了问题的根源

> 以下内容来自于`projectRoot/source/_data/next.yml`
```
menu:
  home: / || home
  #about: /about/ || user
  #tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

万恶之源就在于``*home: / ~~|| home~~* `` || home也生成到了路径里面！！

> 以下是正确示范
```
menu:
  home: /
  #about: /about/ || user
  #tags: /tags/ || tags
  categories: /categories/
  archives: /archives/
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

其他的细枝末节就不说啦，我的小站总算是完成了万里长征的第一步。

## Refreence：

Hexo: 
https://hexo.io/zh-cn/docs/github-pages
https://hexo.io/zh-cn/docs/permalinks

Travis:
https://docs.travis-ci.com/

Next.js:
https://github.com/iissnan/hexo-theme-next

## 完！
