---
title: Github+Hexo搭建个人博客
date: 2017-02-23 00:52:12
tags: hexo

---

# Ubuntu上结合Github用Hexo搭建博客

### [](#u7B80_u4ECB "简介")简介
在搭建博客的过程中会涉及到下面这些东西：

1.  Hexo
2.  Git
3.  Github Pages
4.  Npm
5.  Nodejs

**[Hexo](https://github.com/hexojs/hexo)**

```
一款基于Node.js的简单、快速、强大的静态博客框架

```

<!--more-->

**Hexo搭建博客和github有什么关系**

那Hexo就是一个博客框架，关Github什么事情呢，这还被你说对了，还真不怎么和github相关，用hexo弄一个博客出来很简单。当然你也可以选择用wordpress来结合hexo，只是这里选择用github pages服务，那又说到了一个东西:**github pages**。

**[Github Pages](https://pages.github.com/)**

上面是网址，可以自己进去看看，简单的说就是github提供的一种用来展示托管在自己github仓库上的静态网页。github pages也有自己的一套框架，只用github pages也可以搭建自己的博客。

**[Npm](https://www.npmjs.com/)**

一个NodeJs包管理和分发工具，全称为Node Package Manager。和Ruby的gem，Python的pypi类似。通过npm能快速的部署hexo框架，毕竟hexo基于nodejs。

**[Node.js](https://nodejs.org/en/)**

Node是一个Javascript运行环境(runtime)。实际上它是对Google V8引擎进行了封装。V8引 擎执行Javascript的速度非常快，性能非常好。Node对一些特殊用例进行了优化，提供了替代的API，使得V8在非浏览器环境下运行得更好。

### [](#u5B89_u88C5 "安装")安装

#### [](#1-__u5B89_u88C5Node-js "1\. 安装Node.js")1\. 安装Node.js

在Ubuntu下面部署很容易的= =,在终端输入：

```
sudo apt-get install --yes nodejs

```

Nodejs的部署工作就完成了。其他的Linux发行版可以参照下面的教程[Installing Node.js via package manager](https://github.com/nodejs/node-v0.x-archive/wiki/Installing-Node.js-via-package-manager)

也可以在Node的官网上直接[下载](https://nodejs.org/en/download/)安装。

安装好之后，在终端输入nodejs即可进入到nodejs的交互模式中。

**Note**
需要注意一个问题就是，在hexo中的nodejs文件在运行时使用的是`node xxx/js`这样的形式，而在Ubuntu下面直接运行`node xxx.js`会失败，报错为`/usr/bin/env: node: No such file or directory`，网上有些说是和node的版本有关，实际上是因为NodeJs在Ubuntu上默认安装之后，需要`nodejs xxx.js`这样用，解决方法为创建如下软链接，保证可以运行`node xxx.js`:

```
ln -s /usr/bin/nodejs /usr/bin/node

```

#### [](#2-__u5B89_u88C5npm "2\. 安装npm")2\. 安装npm

在终端输入：

```
sudo apt-get install npm

```

#### [](#3-__u5B89_u88C5hexo "3\. 安装hexo")3\. 安装hexo

终端输入：

```
npm install hexo-cli -g

```

在这里可能会报错，由于没有root权限导致无法安装hexo，`sudo su`root一下再安装一次就好了。

到这里就完成了对Hexo的初步安装了，直接在终端输入`hexo`会出现相关的信息。

#### [](#4-__u5B89_u88C5git "4\. 安装git")4\. 安装git

首先安装git

```
sudo apt-get update
sudo apt-get install git

```

设置用户信息

```
$ git config --global user.name "Xiaojuanmao"//用户名
$ git config --global user.email  "daque@hustunique.com"//填写自己的邮箱

```

检查SSH keys

```
$ cd ~/. ssh

```

如果提示No such file or directory 说明你是第一次使用git。按照如下步骤处理SSH Keys，如果存在SSH Keys，则直接跳过下面分割线内的部分。

* * *

**配置SSH Keys**

*   生成新的SSH Keys

    ```
    $ ssh-keygen -t rsa -C "邮件地址@youremail.com"

    ```

    会出现下面的提示：

    ```
    Generating public/private rsa key pair.
    Enter file in which to save the key
    (/Users/your_user_directory/.ssh/id_rsa):

    ```

    直接回车，存储在默认的目录下面。系统会提示输入密码，密码的作用是在向仓库提交代码的时候用到，可以防止其他人向自己的仓库提交代码。输入密码后，相关的会生成.ssh文件。

*   添加新的SSH Keys到GitHub

    通过下面的命令进入目录，该目录下存放着刚才生成的密钥文件

    ```
    $ cd ~/. ssh

    ```

    登陆github系统。点击右上角的 Account Settings—->SSH Public keys —-> add another public keys。打开刚才目录下面的`id_rsa.pub`文件，将文件内容复制到key文本框中就可以了。

*   测试SSH
    可以输入下面的命令，测试SSH是否设置成功

    ```
    ssh -T git@github.com

    ```

    如果出现下面的信息，则说明设置成功

    ```
    Hi XXX! You've successfully authenticated, but GitHub does not provide shell access.

    ```

    **Note**
    也可能会报出错误：`Agent admitted failure to sign using the key.Permission denied (publickey).`这是由于没有将新建的ssh密钥加入，下面的命令可以解决：

    ```
    ssh-add   ~/.ssh/id_rsa

    ```

* * *

### [](#u4F7F_u7528 "使用")使用

#### [](#1-__u4F7F_u7528github_u521B_u5EFA_u535A_u5BA2_u4ED3_u5E93 "1\. 使用github创建博客仓库")1\. 使用github创建博客仓库
在github上创建一个仓库，**仓库的名字和用户名必须对应**，如我的帐户名为`Xiaojuanmao`,则创建的仓库名称为`Xiaojuanmao.github.io`。这样存放在github上的远程仓库就准备好了，下面来用hexo来初始化本地的仓库内容。

#### [](#2-_Hexo_u521D_u59CB_u5316 "2\. Hexo初始化")2\. Hexo初始化

在主文件夹下创建一个hexo文件夹，进入文件夹，在终端输入如下的命令

```
hexo init

```

会给出这样的反馈：`INFO Copying data to ~/hexo INFO You are almost done! Don't forget to run 'npm install' before you start blogging with Hexo!`
接着按照上面的提示，输入命令

```
npm install

```

会自动在目录下面安装node_modules。接着在命令行中启动本地的服务器，可以用来预览个人博客的样子：

```
hexo server

```

反馈信息会提示已经在挂在了本地的服务器：`INFO Hexo is running at http://0.0.0.0:4000/. Press Ctrl+C to stop.`

在浏览器中打开`http://0.0.0.0"4000/`可以看到网页的整个框架已经生成了。有个默认的主题，如果觉得这个主题不好看，hexo还有好多主题可供更换。

#### [](#3-__u6DFB_u52A0_u6587_u7AE0 "3\. 添加文章")3\. 添加文章

打开命令行，进入到hexo的目录下，利用如下的命令，可以新建一个.md格式的文件。

```
hexo new "My New Post"
反馈信息：INFO  Created: ~/hexo/source/_posts/My-New-Post.md

```

刷新刚才的`localhost:4000`，就能看到一篇新的博客出现了，用起来还是炒鸡方便的。创建之后再去编辑这个.md文件，写自己想写的内容就可以了。

#### [](#4-__u751F_u6210_u9759_u6001_u7F51_u9875 "4\. 生成静态网页")4\. 生成静态网页

下面的命令生成静态的网页，在将本地的内容部署到github上面去之前，一定要先执行这个步骤。

```
hexo generate
   或者 hexo g

```

执行完之后，会在./public的目录下生成一系列的.html,.css文件。

#### [](#5-__u90E8_u7F72_u5230Github "5\. 部署到Github")5\. 部署到Github

在和github完成对接之前，需要去配置hexo自己的配置文件`_config.yml`。关于这个文件里面的一些内容，需要进行一些修改：

```
# Hexo Configuration
## Docs: http://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/

# Site
title: Xiaojuanmao's Blog # 网站的标题
subtitle: Life began in 1990 # 网站的副标题
description: Coding Life # 显示在网页最下面的描述，类似于tag
author: Xiaoxiaoda
email: daque@hustunique.com
language: zh_CN
timezone:

# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git #这里不要写github了，hexo3.0之后用git代替了github
  repository: git@github.com:Xiaojuanmao/Xiaojuanmao.github.io.git # 填写自己的git仓库地址，之前创建好了的
  branch: master

```

更改完配置文件之后保存，通过下面的命令部署到github上：

```
hexo generate 或者 hexo g  #生成静态网页
hexo deploy 或者 hexo d #部署到github
上面两个命令可以和并为 hexo d -g

```

**Note**
部署的过程中可能会出现如下的问题：

```
ERROR Deployer not found: github

```

遇到这个不要慌，是hexo升级到3.0之后用git代替了github，所以需要再输入下面的命令，安装git的deployer

```
npm install hexo-deployer-git --save

```

安装之后就可以将静态的网页部署到github的远程仓库上面。