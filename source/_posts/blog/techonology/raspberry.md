---
title: "Raspberry 上手笔记"
date: 2020-07-16 16:27:38
categories:
- Raspberry
---


## 工具
准备刷机用的东西包括（必备）：

1. 16Gtf卡一个加读卡器（`U盘不行，按说应该也可以，但是步骤繁琐，没有继续折腾`）
2. 一根typeC数据线用于刷机

3. 下载刷机包，由于同事手里有先前下载好的包和刷机工具，所以我省去了这一步。具体我刷的是buster的树莓派简化版。
> 下载地址为：

* 刷机工具[etcher](https://www.balena.io/etcher/)：`https://www.balena.io/etcher/`
* 树莓派镜像：`https://www.raspberrypi.org/downloads/raspbian/`

## 步骤

(以下都狠简单，在网上一搜一大堆，只是个人记录用)
### 1. （废话，有空删掉）

购买树莓派，本人在转转上买了二手树莓派4b一个，200块。

### 2. 使用刷机工具制作系统刷到tf卡上。

步骤参考：`https://shumeipai.nxez.com/2019/04/17/https://www.fing.com/products/fing-appwrite-pi-sd-card-image-using-etcher-on-windows-linux-mac.html`

### 3. 树莓派首次联网，这里需要注意树莓派4b是带无线网卡的，可以直接先在tf卡制作好的系统中配置好无线ap信息。

步骤参考：`https://shumeipai.nxez.com/hot-explorer#beginner`
### 4. 把tf卡安装到树莓派上，开机。
> 首次启动
测试树莓派是否正常启动并且正常联网的方法：
1. 通过串口查看，或者连接显示设备
2. 都没有的话，只能过一会儿可以试试ssh raspberrypi.local 是否能连上。
3. 或者是通过路由器或者是[fing](https://www.fing.com/products/fing-app)工具查看局域网中所有设备，如果说有名字叫Raspberry相关的，应该就是树莓派.(我是这样搞的)


### 5. 必备应用安装

以下个人观点：
```
sudo apt install vim 
```

### 6. 配置vnc server

参考树莓派实验室：`https://shumeipai.nxez.com/2018/08/31/raspberry-pi-vnc-viewer-configuration-tutorial.html`


### 7. 我遇到的一些傻问题

* a. 没看教程，尝试自己安装vnc server
结果，当然失败了，系统自带的就行

* b.使用其他客户端尝试登陆上去
,树莓派4b自带的是[RealVNC](https://www.realvnc.com/en/connect/download/viewer/) 
如果连接的话，需要单独下载客户端，我试过mac上面的dump desktop连不上去。提示错误：

`no configured security type is supported by 3.3 vnc viewer raspberry pi`

* c. 下了RealVNC Viewer,但是傻憨憨的不知道默认用户名密码！查了半天才找到

默认账号：`pi`
默认密码：`raspberry`


## 结语
不要小瞧小小的树莓派，原本我以为都是linux的，和ubuntu的差不多，但是实际上还是有很多需要注意的地方，如果提前先在网上看看，可以避免很多低级错误！切记！

## 学习链接：

官网：`https://www.raspberrypi.org/`
树莓派实验室：`https://shumeipai.nxez.com/`
树莓派技术圈：`https://raspberrypi.tech/`







