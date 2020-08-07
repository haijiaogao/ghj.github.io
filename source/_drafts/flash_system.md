## 重装系统总结

### 1. 备份系统所在磁盘：

１．~/.*

所有常用的应用的以.开头的文件可以都copy下来，我并没有，我只copy了 .vim .oh-my-zsh .ssh  .git  ，但是在用的过程中发现有些没有考全，如bin/（在.bashrc和.zshrc文件中有引用到的目录）以及/opt目录下的所有的内容等
2．document/常用软件，项目文件等
3. desktop/日常文件等，这个其实只是单纯备份，用的时候再解压也一样


### 2.重装后工作

### 首先是apt-update，然后安装各种包(vim,git,autojump,chrome ,openjdk,sublime等以及编译相关的)

> open-jdk，chrome，sublime等需要单独添加源以及key。

### 搜狗输入法环境搭建

1. 下载deb包：官网下载，很小，几乎不用备份
2. shell执行
```
sudo dpkg -i sogouxxx.deb

sudo apt install -f
```
3. 打开设置，languagueand locale,增加中文语言，安装语言相关更新，切换输入接口为fcitx,系统默认为ibus,然后重启


4.重启后，右上角可以配置fxitx，添加输入法，取消掉当前语言小钩钩，然后搜索sogou,并添加，就可以输入中文了。

参考：`https://www.jianshu.com/p/cafe12618293`


### 网络环境搭建

1. clash

用的备份的`clash-linux-amd64-v0.19.0.gz`，解压文件.然后mv to /opt ，最后ln到usr/bin/clash

把备份的配置文件放在~/.config/clash下面之后就可以在命令行执行clash 启动应用了


### 备份文件的还原
主要就是我备份的那些软件，配置等等。


### 一些没有备份的工作以及日常应用的安装：





** sublime **

1.安装依赖包

```
sudo apt update
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

2. Import the repository’s GPG key using the following curl command :

```
curl -fsSL https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -
```

3.Add the Sublime Text APT repository to your system’s software repository list by typing:

```
sudo add-apt-repository "deb https://download.sublimetext.com/ apt/stable/"
```

4. Once the repository is enabled, update apt sources and install Sublime Text 3 with the following commands:

```
sudo apt update
sudo apt install sublime-text
```

5.完成，首次打开时需要输入License，搜索到有效的license来自`https://gist.github.com/mehedicsit/7389e31b7857b377870a32e91be1c7c5`

如下内容：

```
Thanks GUys use this oone

----- BEGIN LICENSE -----
Member J2TeaM
Single User License
EA7E-1011316
D7DA350E 1B8B0760 972F8B60 F3E64036
B9B4E234 F356F38F 0AD1E3B7 0E9C5FAD
FA0A2ABE 25F65BD8 D51458E5 3923CE80
87428428 79079A01 AA69F319 A1AF29A4
A684C2DC 0B1583D4 19CBD290 217618CD
5653E0A0 BACE3948 BB2EE45E 422D2C87
DD9AF44B 99C49590 D2DBDEE1 75860FD2
8C8BB2AD B2ECE5A4 EFC08AF2 25A9B864
------ END LICENSE ------​

```

5.install package manager,由于我配置了代理，直接ctrl+shift+p ,输入install package 之后回车就安装成功了，如果不成功可参考网上其他方案



electronic-wechat

参考：`https://www.linuxbabe.com/desktop-linux/install-wechat-linux`

由于我有备份electronic-wechat的zip包，所以采用直接解压缩安装的方式：
How To Install Electronic Wechat on Linux via Tarball
This is a traditional way to install Wechat on Linux. Go to github and download the tar.gz file. Choose linux-x64.tar.gz or linux-ia32.tar.gz according to your OS architecture. Once downloaded, open a terminal window and navigate to the download folder where the tar.gz file is located. Then issue the following command to extract it.

```
tar xvf linux-x64.tar.gz

或者

unzip electronic-wechat-linux-x64-2.3.1.zip -d electronic-wechat
```

打开electronic-wechat目录后执行：

```
./electronic-wechat
```

下面步骤为了方便使用，并非必须

```
sudo mv {path to electronic-wechat}/electronic-wechat-linux-x64 /opt

sudo ln -s /opt/electronic-wechat-linux-x64/electronic-wechat /usr/bin/wechat

```
So now we just need to press ** ALT+F2 ** and enter wechat command to launch the WeChat client on Linux.

