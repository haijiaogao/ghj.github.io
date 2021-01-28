---
title: 用python爬取网易金融股票每日数据并存入MySql数据库
tags: [python, mysql, ubuntu]
---

# 起因
唱老师最近研究股票和基金，每日盯着手机几个金融软件，有天突发其想给我提了个需求：是否有抓取某几只股票一段时间内的股票数据的接口？我想了想，查了查，网上好多人都这么干过。于是翻了几篇帖子和github上一些开源的库，开搞。

# 准备

> 数据其实可以存成csv的文件，后期可以用python的pandas软件进行数据分析，但是我因为对python了解的不太多，还是从自己熟悉的存到数据库入手，但是，其实差不多。但是存到数据库会有更多的坑需要一一去趟。

1. ubuntu 安装mysql-server以及相关包，并配置 mysql
 ```
sudo apt install mysql-server
sudo apt-get install python-mysqldb
sudo apt-get install libmysqlclient-dev 
``` 
其中第一个`mysql-server`装完了之后，就可以对mysql数据库进行配置，具体配置步骤如链接：[DegitalOcean]('https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-18-04')

后面两个都是为了python连接mysql数据库用的。

2. python 安装相关包

    ``` language=bash
    pip install beautifulsoup4
    pip install requests
    pip install lxml
    pip install MySQL-python
    pip install mysql-connector-python
    ```

    2.1 使用python venv环境模拟python2.7环境，进行开发
由于python2.7 的版本已经不再维护了，所以在使用pip下载包的时候会频繁的报如下错误：
```
Python 2.7 reached the end of its life on January 1st, 2020. Please upgrade your Python as Python 2.7 is no longer maintained. pip 21.0 will drop support for Python 2.7 in January 2021. More details about Python 2 support in pip can be found at https://pip.pypa.io/en/latest/development/release-process/#python-2-support pip 21.0 will remove support for this functionality
```

所以使用了[venv]('https://docs.python.org/3/tutorial/venv.html'),这样就可以方便的使用pip了，同时还可以分离开项目的依赖和系统python环境的依赖。
    2.2 pyCharm社区版安装
PyCharm社区版`https://www.jetbrains.com/pycharm/download/#section=linux`


# 网易股票数据接口

非常简单，大概是这样事儿的：
```
url= 'http://quotes.money.163.com/trade/lsjysj_' + 股票代码 + '.html?year=' + 年份（2020） + '&season=' + 季度（1,2,3,4）
```
返回的是一个html的页面，里面包含数据表格，需要使用python request模拟请求，并用beautifulSoup处理html，把数据整出来，存到mysql数据库中。

这里基本上是抄的别人的，没有什么技术难点，难点或许是在于对mysql数据库的操作上。但是也并不很复杂。

项目地址：[Github:]('https://github.com/haijiaogao/stockDemo.git')

# TODo
项目仅仅实现了数据导入的功能，后续还有很多目标：
1. 数据Web展示
2. 唱老师说需要知道某只股票的市值信息，以及盈利数据信息。
看了一下，接口都有，可以取到数据，难点在于接口很多，对于业务不了解的情况来说需要辨别那些是真的有用的那些是无用的。此处感觉更复杂。
3. 数据分析 