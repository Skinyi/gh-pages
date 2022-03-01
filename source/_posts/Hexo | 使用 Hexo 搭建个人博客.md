---
title: 使用 Hexo 搭建个人博客
lang: zh-CN
tags: 
  - Javascript
  - Nodejs
  - Hexo
  - Markdown
categories: 
  - [学习, 杂项]
  - [日常记录]
---

一直以来都有搭建个人网站或者说博客的想法，今天终于把这件事做成了，感谢 ~~Github~~ 各大代码托管平台的 ~~GitHub~~ XXX Pages 服务，使我们即使没有公网服务器也能搭个人人可访问的静态个人网站，而且无需任何费用。当然国内码云也有这项服务，~~由于一些理由我没有在那上面弄，~~ Github Pages 由于国内访问速度太慢了，我就在码云上面也创建了仓库，然而启用 Pages 功能需要实名认证。言归正传，按照 Hexo 社区的惯例以及正好作为自己对搭建个人网站过程的记录，我的个人网站上的第一篇文章就打算写写这个搭建博客的过程。

## 准备工作

搭建博客所需的环境和官网地址如下：

| 名称        | 作用                                 | 官网                                       |
| ----------- | ------------------------------------ | ------------------------------------------ |
| Nodejs      | Hexo 博客框架依赖的开发环境          | https://nodejs.org/en/                     |
| Git         | 推送博客至 github 或其他一些代码托管 | https://git-scm.com/                       |
| Hexo        | 快速、简洁且高效的博客框架           | https://hexo.io/zh-cn/                     |
| Purer theme | 一款简洁高效的响应式个人博客主题     | https://github.com/fengkx/hexo-theme-purer |

博客系统平台环境搭建在我的 Fedora 35 虚拟机上。

```bash
[skinyi@localhost ~]$ uname -a
Linux localhost.localdomain 5.16.9-200.fc35.x86_64 #1 SMP PREEMPT Fri Feb 11 16:29:17 UTC 2022 x86_64 x86_64 x86_64 GNU/Linux
```

## 安装步骤

我非常建议你按照我上面所给的表格来访问官方文档来分步进行部署。官方文档永远不会过时而且详尽，会最大程度上解决你的疑虑。以下是我对此次部署过程的一次记录，其中会着重记录我的部署过程中踩过的一些坑。

### 安装 Nodejs

使用 Linux 系统我个人比较倾向于从软件仓库获得软件，但是软件仓库里的软件不一定会是最新版的，但是这种方式可以使我们省去自己编译、配置的时间。Nodejs 官网上的下载页面里的 Linux 二进制版本是压缩包，要从软件仓库获得它的话需要点击下面的 `Installing Node.js via package manager` 链接来查阅官方文档。

根据文档的描述，Fedora 35 可以通过安装 dnf 软件模块的方式来进行安装，可以先看看仓库里已有的软件流版本：

```bash
[skinyi@localhost ~]$ dnf module list nodejs
上次元数据过期检查：0:08:01 前，执行于 2022年02月23日 星期三 18时50分04秒。
Fedora Modular 35 - x86_64
Name                    Stream                 Profiles                                             Summary                           
nodejs                  12                     default, development, minimal                        Javascript runtime                
nodejs                  14                     default, development, minimal                        Javascript runtime                
nodejs                  15                     default, development, minimal                        Javascript runtime                
nodejs                  16                     default, development, minimal                        Javascript runtime                

Fedora Modular 35 - x86_64 - Updates
Name                    Stream                 Profiles                                             Summary                           
nodejs                  12                     default, development, minimal                        Javascript runtime                
nodejs                  14                     common, development, minimal                     Javascript runtime                
nodejs                  15                     default, development, minimal                        Javascript runtime                
nodejs                  16 [e]                 common, development, minimal                 Javascript runtime                

提示：[d]默认，[e]已启用，[x]已禁用，[i]已安装
```

Hexo 官方文档中建议 Nodejs 版本使用 Nodejs 12.0 及以上版本的，在这里我选择安装 Nodejs 16 版本的。

```bash
[skinyi@localhost ~]$ sudo dnf module install nodejs:16
```

安装完成后查看 Nodejs 版本号：

```bash
[skinyi@localhost ~]$ node -v
v16.14.0
```

从仓库安装完成后，需要升级 npm 版本以及为了不用以 root 权限执行 npm 命令，需要更改 npm 的配置：

```bash
# 在当前用户主目录创建 npm 全局安装目录
[skinyi@localhost ~]$ mkdir ~/.npm-global
# 配置 npm 使用刚才建的目录路径
[skinyi@localhost ~]$ npm config set prefix '~/.npm-global'
```

添加 ~/.npm-global/bin 路径到用户环境变量：

```bash
[skinyi@localhost ~]$ echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.bash_rc
[skinyi@localhost ~]$ source ~/.bash_rc
```

### 安装Git

使用 `dnf` 包管理器安装 Git：

```bash
[skinyi@localhost ~]$ sudo dnf install git-core
```

git 全局配置，如果需要通过 git 将博客部署到 Github Pages 上，需要配置 git 的一些全局设置：

```bash
[skinyi@localhost ~]$ git config --global user.name "github 上的用户名"
[skinyi@localhost ~]$ git config --global user.email "注册 github 账号使用的邮箱"
```

### 安装 Hexo

以上依赖软件安装成功后，就可以进行 Hexo 的安装了：

```bash
[skinyi@localhost ~]$ npm install -g hexo-cli
```

在工作目录里初始化自己的博客项目：

```bash
[skinyi@localhost ~]$ hexo init blog
[skinyi@localhost ~]$ cd blog
[skinyi@localhost ~]$ npm install
[skinyi@localhost ~]$ hexo server
```

可以通过浏览器访问本地地址：http://127.0.0.1:4000 来预览生成的博客网站，若要在外部访问的话需要通过防火墙开放 4000 端口，在此不再赘述。

## 更换博客主题

从 Hexo 主题列表里筛选了一圈后我终于选中了此时用的这款主题，它的 Github 链接是：https://github.com/fengkx/hexo-theme-purer ，如果你也喜欢这款主题，可以通过项目主页的的说明文档来了解它的安装、配置及使用，不过建议你在看作者的文档前先大致看下 Hexo 的使用文档来了解了解 Hexo 的一些基本概念，这样看作者的文档再上手就比较容易了。

> 这款主题是基于 EJS 和 Tailwind CSS 构建的，以后若有兴趣构建自己的主题可以参考使用这两种技术。

### 下载主题文件

我没有按照官方使用 git clone 同步的方法来下载主题，直接通过下载仓库源码的方式下载下来。gh-pages 是我的整个博客项目的主目录。

```bash
[skinyi@localhost gh-pages]$ wget -c https://github.com/fengkx/hexo-theme-purer/archive/refs/heads/master.zip -O themes/
[skinyi@localhost gh-pages]$ unzip themes/master.zip
[skinyi@localhost gh-pages]$ mv themes/hexo-theme-purer-master themes/purer
```

文档中为了避免由于主题更新而导致原有的主题目录下的 `_config.yml` 文件失效故而选择把主题的 `_config.yml` 放在主题外面并取名为 `_config.theme.yml`，在进行编译时将此文件的内容写进主题底下的 `_config.yml`。虽然我之后不会更新这个主题但是我还是按照文档里的做了，只不过我起的名字为 `_config.purer.yml`。

将 purer 主题下的 `_config.example.yml` 复制到项目主目录下。然后编辑 package.json 的内容如下：

```bash
[skinyi@localhost gh-pages]$ cp themes/purer/_config.example.yml _config.purer.yml
[skinyi@localhost gh-pages]$ code package.json
```

{% codeblock lang:json package.json %}
{
  "scripts": {
    "theme": "cat ./_config.purer.yml > ./themes/purer/_config.yml ",
    "build": "npm run theme && hexo generate",
    "clean": "hexo clean",
    "deploy": "npm run theme && hexo deploy",
    "server": "npm run theme && hexo server"
  },
}
{% endcodeblock %}

### 配置主题

#### 安装及配置插件

首先可选的工作是卸载不需要或不推荐的渲染器：

```bash
[skinyi@localhost gh-pages]$ npm uninstall hexo-renderer-stylus
[skinyi@localhost gh-pages]$ npm uninstall hexo-renderer-marked
```

安装 markdown-it 渲染器以及其他常用插件：

```bash
# markdown-it 渲染器
[skinyi@localhost gh-pages]$ npm i -S hexo-renderer-markdown-it
# 支持从post_assert_folder 用 markdown 引入图片
[skinyi@localhost gh-pages]$ npm i -S hexo-asset-image
# 支持 emoji
[skinyi@localhost gh-pages]$ npm i -S markdown-it-emoji
# 支持数学公式
[skinyi@localhost gh-pages]$ npm i -S @iktakahiro/markdown-it-katex
```

由于 `hexo-renderer-markdown-it` 默认不生成 h1 的锚点，所以我们需要在站点配置文件添加如下设置，在插件对象里将刚才添加的两个插件加进去:

{% codeblock lang:yml _config.yml %}
markdown:
  html: true
  xhtmlOut: true
  breaks: true
  langPrefix:
  linkify: true
  typographer:
  quotes: “”‘’
  anchors:
    level: 1
    permalink: false
    separator: '-'
  plugins:
    - '@iktakahiro/markdown-it-katex'
    - markdown-it-emoji
{% endcodeblock %}

#### 创建常见页面

在 项目主目录/source 下按需添加 categories、tags、repositories、links、about 目录，这些分别对应博客框架中的分类页、标签页、仓库页（主题独有）、友链页、关于页。然后在其中创建对应的 index.md 文件。

我的项目没有要友链页，故最终目录结构如下：

```bash
[skinyi@localhost gh-pages]$ tree -L 2 source
source
├── about
│   └── index.md
├── categories
│   └── index.md
├── images
│   ├── avatar.jpg
│   └── favicon.png
├── _posts
│   ├── 使用 Hexo 搭建个人博客.md
│   └── images
├── README.md
├── repository
│   └── index.md
└── tags
    └── index.md

7 directories, 8 files
```

如果你事先看过 Hexo 的文档，你就会知道 `_post` 目录存放的是写好的文章以及我们刚刚创建的 *index.md* 需要添加 *front-matter*。以内容稍微多的 about 目录的 front-matter 为例：

```txt
---
title: 关于
description: 个人简介
layout: about
sidebar: custom
date: 2022-02-22 20:55:00
---
```

其他目录的 front-matter 可以参考主题作者文档中的 Demo 链接。

#### 定制主题配置

接下来需要修改项目主目录下的 `_config_purer.yml` 文件来定制自己的主题，这个参照主题文档按照自己的心意定制就成，我没有要友链和书单页面。

## 写作

所有已经发表的文章都在项目目录的 *source/_posts* 子目录下，Hexo 同样支持添加草稿，草稿目录里的文章默认不会渲染出来。对我来说懒得使用草稿了，因为吹牛根本不需要打草稿。我比较喜欢 VSCode 里边写边想，写完后再部署就可以了，根本不需要草稿。

博文是以 markdown 格式编写的，好在这门语言的学习成本并不高，多写多用多记就可以满足大部分使用需要，比较冷门的格式需要用的时候再查也花费不了太长时间。

## 部署文章

博客网站搭建起来了需要部署到公网上才能被他人看到，但是公网 IP 以及服务器需要花费不小的经济成本，好在一些代码托管网站都提供了 XXX Pages 服务。需要注意的是：在国内的平台上启用 Pages 服务一般都需要进行实名认证，而在 Github Pages 上部署不需要进行实名。

使用 git 的方式进行博客部署可以安装 hexo 插件 *hexo-deployer-git*：

```bash
[skinyi@localhost gh-pages]$ npm install hexo-deployer-git --save
```

在 *_config.yml* 中修改配置：
```yml
deploy:
  type: git
  repo: 
    github: <github 仓库地址> # 如 https://github.com/xxx/xxx.github.io
    gitee: <gitee 仓库地址> # 如 https://gitee.com/xxx/xxx
    branch: [代码分支] # 如 master
    message: [提交信息]
```

生成站点文件并推送至远程仓库：

```bash
[skinyi@localhost gh-pages]$ hexo clean && hexo deploy
```

> 通过 Git 方式提交时建议在本地生成公私钥对并将生成的公钥文件添加到远程仓库的公钥列表里，然后每次提交时就可以通过验证公私密钥对的方式而不用每次都输用户名及密码。

到这里基本算是介绍完了所有的搭建步骤，之后我会把我在 Typora 里写的所有学习笔记都搬运到这个博客中，当然这个网站的内容并不局限于一些跟技术相关的东西，我并不确定我写的这些东西会不会有人看，我也打算写一些技术博客以外的东西，不管以后怎么样、写些什么，就活在当下、享受并记录生活吧。