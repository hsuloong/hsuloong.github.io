---
title: GitHub+Hexo搭建个人博客站及备份原始网站数据过程记录
urlname: build-blog-website
date: 2018-02-28 19:56:13
copyright: true
mathjax: true
tags:
- Hexo
categories:
- Hexo
---

万事开头难，大约在一年前我就想着架个属于自己的博客站，但是由于各种原因多次准备架站时又不得不中途放弃，这次总算是上线了，写下这篇文章作为最终成功上线博客的留念。
## Hexo快速安装

### 注册Github账号

如果你已经有了账号可略过该步，传送门：（[注册GitHub传送门](https://github.com)）

### 创建repository
点击头像左边的+号（[新建repository传送门](https://github.com/new)），新建一个repository，“Repository name”格式必须为：username.github.io，username为你注册的用户名。设置完名字建议勾选“Initialize this repository with a README”，便于新建一个用于保存原始网站文件的分支（branch）。点击“Create repository”创建repository完毕。<br>
创建repository后项目下会有一个默认的master分支，该分支用来保存Hexo生成的网页文件。接着新建分支用来存放Hexo原始网站数据方便自己更换电脑时同步。比如创建一个名为backup的分支，并设置为默认分支（这步一定要做）。

### 安装Git工具
Git是开源的分布式的版本控制系统，具体见（[Git是什么传送门](http://www.runoob.com/git/git-tutorial.html)）。Git软件下载地址见（[Git下载传送门](https://git-scm.com/download/)）。安装过程和Windows常用软件相似。

### 绑定Git和你的GitHub账号
Git安装完毕后打开“Git Bash"，使用命令行界面绑定GitHub账号。输入以下命令：

``` 
git config --global user.name “你GitHub注册用户名”
```

``` 
git config --global user.email “你GitHub注册的邮箱账号”
```

以上便完成Git和你的GitHub账号的绑定
### 注册密钥
在“Git Bash”中输入

``` 
ssh-keygen -t rsa -C “你GitHub注册的邮箱账号”
```

会在Windows系统安装盘“C:\Users\你的电脑名称\\.ssh”目录下生成一个名为“id\_rsa.pub”的文件，用任何文本处理软件打开，推荐使用NotePad++打开。（[NotePad++下载传送门](https://notepad-plus-plus.org/download/v7.5.5.html)）<br>
打开GitHub用户设置界面，切换到“SSH and GPG keys”下（[添加SSH key传送门](https://github.com/settings/ssh/new)）。将“id_rsa.pub”文件内容复制的“Key”文本框中，“Title”文本框随便填写一些信息，比如阐明Key的用途。最后点击“Add SSH Key”按钮完成密钥的注册。

### 安装Node.js
由于Hexo基于Node.js，因此需要安装Node.js。（[下载传送门](https://nodejs.org/en/download/)）<br>
和普通Windows软件一样安装，安装结束后打开cmd或者PowerShell输入
``` 
node -v
```
如果输出版本号则证明安装成功。

### 克隆repository到本地
由于我们需要在另外一个分支备份网站的原始数据，为了避免上传原始网站数据到GitHub仓库的出现各种错误，在你本地新建一个文件夹，进入该文件夹后按住Shift键右击鼠标打开“Git Bash”，输入：

``` 
git clone git@github.com:你GitHub注册用户名/你GitHub注册用户名.github.io.git
```

结束后本地会新建一个名为“你GitHub注册用户名.github.io”的文件夹。

### 本地安装Hexo
进入上步“你GitHub注册用户名.github.io”的文件夹，按住Shift键右击鼠标打开“Git Bash”，输入：

``` 
npm install -g hexo-cli 
```

这是安装Hexo，需要一点时间下载。

然后输入：

``` 
hexo init blog
```

会在当前目录创建一个blog文件夹，所有的建站文件均在该文件夹下，将文件夹下的所有文件剪切到上级目录即“你GitHub注册用户名.github.io”文件夹，在该文件夹下按住Shift键右击鼠标打开cmd或者PowerShell输入：

``` 
hexo g
```

``` 
hexo s
```

然后在浏览器中输入
``` 
localhost:4000
```
便可以打开刚刚建好的博客站了。

### 推送博客站到GitHub上
上一步创建好的博客站只能在本地电脑中打开，需要推送到GitHub中才能通过“你GitHub注册用户名.github.io”域名方式被网友访问。

首先在“你GitHub注册用户名.github.io”本地文件夹下有一个名为“\_config.yml”的配置文件，这个称为站点配置文件，使用NotePad++打开并拉到文件最后，填入以下文字：

``` 
deploy:
  type: git 
  repo: https://github.com/你GitHub注册用户名/你GitHub注册用户名.github.io.git 
  branch: master
```

如果出现下面的问题可尝试使用如下配置格式：
> bash: /dev/tty: No such device or address  
error: failed to execute prompt script (exit code 1)  
fatal: could not read Username for 'https://github.com': No error 


``` 
deploy:
  type: git  
  repo: git@github.com:你GitHub注册用户名/你GitHub注册用户名.github.io.git  
  branch: master  
```

注意deploy下面需要缩进，冒号之后有空格。

接着进入“你GitHub注册用户名.github.io”本地文件下，在该文件夹下按住Shift键右击鼠标打开“Git Bash”依次输入以下命令：

```
npm install hexo-deployer-git --save  
hexo g -d
```

然后在浏览器中输入“你GitHub注册用户名.github.io”就可以打开刚才建的博客站了。至此建站部分全部完成。

### 备份网站原始数据到GitHub上
进入“你GitHub注册用户名.github.io”本地文件下，在该文件夹下按住Shift键右击鼠标打开“Git Bash”依次输入以下命令：

```
git add .  
git commit -m “你为该次备份提供的说明信息”
git push origin “你在repository中创建的另外一个分支名”
```

如果你以后新增修改文件，可以把上述三条命令全部运行一遍即可备份当前修改

至此本次GitHub安装Hexo及备份原始数据全部完成，享受你的个人博客时光吧！











