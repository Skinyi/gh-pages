---
title: Linux 基本架构与 Shell 初识
lang: zh-CN
tags: 
  - Linux
  - RHEL
  - RHCE
  - 认证考试
categories: 
  - [学习, Linux]
  - [日常记录]
  - [迁移文章]
---
> 🔴 此文章由之前的 Typora 笔记迁移过来，内容可能已经过时。

## Linux 架构概览

​Linux，全称 GNU/Linux，是一种免费使用和自由传播的类 UNIX 操作系统，它基于 POSIX 的多用户、多任务、同时支持多线程和多 CPU,它能运行主要的 unix 工具软件、应用程序和网络协议。GNU/Linux 是宏内核的操作系统，整体贯彻模块化的开发思想，因此支持针对不同应用场景和设备硬件情况进行裁剪定制。下图展示了 linux 内核的基本模块构成。

> 🟢 严格来说，linux 是指 linux 内核，GNU/Linux 才是一个包含软件和内核的完整操作系统。

![Linux 内核架构](images/linux/Linux-内核架构.png)

## Shell 与命令初识

​Shell 是操作系统内核的外壳，用以保护操作系统内核和提供用户和操作系统内核进行交互的接口。同时 shell 还是一种程序设计语言：作为命令语言，它交互式解释和执行用户输入的命令或者自动地解释和执行预先设定好的一连串的命令；作为程序设计语言，它定义了各种变量和参数，并提供了许多在高级语言中才具有的控制结构，包括循环和分支。

​Shell 可以分为两大类，即图形界面 shell 和命令行式 shell。图形界面 shell 相较于命令行 shell 比较占用系统资源且灵活性不高，命令行 shell 因为要记忆大量的 linux 基本操作命令而比较难以掌握，不过很多 Linux 命令本身提供了查阅手册和帮助选项用以提示用户相关的交互指令用法。Shell 根据历史阶段的不同有不同的实现，在 RHEL 中支持 sh 和 bash（默认） 命令式交互 shell（但调用 sh 时仍是使用的 bash，bash 是 sh 的一个最成功的改进产品）。

​以命令行交互方式使用 shell 时，启动终端字符界面后，它在等待用户输入命令时会闪烁一个“`█`”小光标，在它的前面会有一串shell提示符，如下所示（请忽略"$"或"#"前边的\\转义符号，下同）：

``` bash
[user@host ~]\$ █
```

> 🟢 Shell提示符的构成
> 
> user：当前登陆用户，如果使用 root 用户的则显示 root；
> host：登陆终端的主机名，一般由 ip 或域名构成；
> \~：当前操作所位于的工作路径，“\~”代表当前用户的家目录（默认登陆后的工作目录），如示例则完整路径为：/home/user，其他路径以完整绝对路径显示。
> \$：以该系统的普通用户登陆时则显示美元符号“\$”，表明当前会话以普通用户登陆，若以 root 用户登陆时则显示井号“#”，用以提示用户谨慎操作。
> Shell 提示符的内容及形式由 *PS1* 环境变量确定。

​Shell 命令主要由三部分组成，命令、选项、参数，如下例子：

``` bash
[root@host ~]\# usermod -L user01 # command->usermod options->-L args->user01
# 选项若由字母指定，一般前面加一个 "-"，空格后跟参数，若由单词指定，一般前面加 "--"，= 后跟参数
```

​可以在一行中使用分号将不同语句隔离开，shell 会按序执行，但是这样做是有风险的，因为后面的语句不论前面的执行结果成功与否都会执行，因此依赖于前面执行结果的语句可能会得到错误的结果。为了使操作更加安全，可以使用”`&&`“将它们连接起来，这个符号连接的两条语句只有第一条语句执行成功后第二条才会执行。与此相反，”`||`“连接的两条语句只有前一条执行失败的时候才会执行第二条。

​如果你要执行的命令比较冗长，可以使用”`\`“将其切割为多行，如下例所示：

``` bash
[user@host ~]\$ command --this_is_a_very_long_option \ # 保持各个组成部分完整，不要拆分
> very_long_parameter_that_you_never_see_before_1 \
> very_long_parameter_that_you_never_see_before_2
```

### 使用本地终端登录到本地计算机

​终端是一个基于文本的界面，用于向计算机系统输入命令以及显示计算机系统的输出。本地终端是指运行在连接了键盘和显示器直接输入输出的设备上的文本界面，这些设备也称为 Linux 计算机的物理控制台。物理控制台支持多个虚拟控制台，它们可以运行单独的终端。每个虚拟控制台均支持独立的登录会话。

> 🟢 使用 *Ctrl + Alt + F1~F6* 快捷键进行虚拟控制台一到六的快速终端切换。

在 RHEL8 中，如果图形环境可用，则登陆屏幕默认会在称为 tty1 的第一个虚拟控制台中运行，其余第二到第六虚拟控制台则提供文本 Shell 交互界面。用户在登陆界面登陆成功后则会为该用户在二到六顺延空闲的一个虚拟控制台建立图形会话。要在图形界面中获得 Shell 提示符，需要使用图形界面提供的终端应用程序。

> 🟢 使用以下命令查看系统默认以什么方式进行登陆：
> 
> [root@localhost opt]\# systemctl get-default # set-default 或 isolate 选项可以设置其它登陆方式（一般为multi-user.target）（前者重启生效永久保存，后者即时生效临时有效）
> graphical.target    # 默认图形界面登陆

​登陆用户若想退出当前图形界面的交互，需要进行注销操作，注意：切换用户的界面同样是返回 tty1 进行登陆操作但不退出当前操作用户，同样在下一个未使用的虚拟控制台为切换过去的用户提供图形交互界面。用户注销后，物理控制台则自动切回第一虚拟控制台的图形登陆界面，等待选择用户并登录。

​无外设服务器提供了串行端口用以为管理员提供登陆服务，该串行端口为远程登陆服务器和该应用服务器提供连接，串口登陆为网络配置有误或者尚未配置网络的服务器提供登陆支持以修复，不过大多数下，网络配置无误有效的服务器通过网络来进行登陆。

### 使用远程服务登陆到网络计算机

​Linux 用户和管理员通常需要通过网络连接到远程系统来获得对远程系统的 shell 访问。随着现代虚拟化以及云计算技术的不断发展，网络登陆变得更加十分普遍，在 Linux 中，最常见的是使用 Secure Shell Daemon(SSHD) 服务为宿主计算机提供远程登陆获得命令提示符的服务。其它需要登陆到远程服务器的计算机则需要安装 SSH 客户端。最主流的 SSH 实现为 OpenSSH 实现。通常一台 Linux 机器上是同时安装了 SSH 客户端和服务端了的，用以实现登陆登陆其它机器和为其它登陆到自己的机器提供会话。如下例：

``` bash
[user@host ~]\$ ssh remoteuser@remotehost # sshd 服务默认监听 22 端口，登陆时不指定的话客户端默认去连 22
remoteuser@remotehost\`s password: password # 密码不会显示在终端上
[remoteuser@remotehost ~]\$ █
```

> 🟢 本示例展示了通过本地 user 用户远程登陆到了远程 remotehost 上的 remoteuser 用户，且这个过程需要提供身份验证，本示例是使用密码验证，登陆成功后 shell 提示符变为了远程计算机的 shell 提示符。

​ssh 命令通过加密连接来防止通信被窃听或劫持密码和内容。有些系统不允许用户使用密码登陆到远程主以增强安全性，其使用的用户身份验证方法是公私钥身份验证。这种验证方法的流程是：<b>**用户本地存放私钥、远程计算机存放公钥，用户登录时通过为 ssh 命令提供公钥文件路径的参数，ssh 程序通过与远程计算机上安装的授权公钥进行比对，如果两者是匹配的，则不需要密码就可登陆**</b>，如下例：

``` bash
[user@host ~]\$ ssh -i mylab.pem remoteuser@remotehost
[remoteuser@remotehost ~]\$ █
```

> 🟢 用户的私钥文件必须只能该用户本身拥有读取权限。用户首次登陆远程计算机时系统会发出警告，表示本地没有存储过远程计算机的密钥，如果你信任该密钥，则本地计算机会保存其密钥登陆且下次不再提醒。

​使用 `exit` 命令可以退出在远程计算机上的登陆，且 shell 提示符切换回本地的：

``` bash
[remoteuser@remotehost ~]\$ exit
logout
Connection to remotehost closed.
[user@host ~]\$ █
```

### Linux 命令执行优先级

​Shell 中可执行命令分为 shell 内建命令和外部命令。内建命令直接从内存中读取只需提供命令名，而外部命令需要从系统文件中读取可执行文件，一般来说可以直接指定可执行文件的绝对路径或相对路径，但是这样要输入长串的路径，比较麻烦，因此 shell 提供了环境变量 `$PATH` 来指定命令的搜索路径，这样的话执行命令就可以直接使用命令名来执行了，shell 程序会自动搜索 `$PATH` 环境变量中的路径，同时为了加快外部命令的执行过程 shell 还提供了缓存表机制。有些常用的命令可能输入起来比较长，因此还有别名机制用以实现快速输入执行。

查看命令类型可以使用 `type` 命令：

``` bash
[root@localhost ~]\# type type    # type 是内建命令,除此之外 echo、hash、alias、pwd也是
type is a shell builtin
[root@localhost ~]\# type top    # top 是外部命令
top is /usr/bin/top
[root@localhost ~]\# type rm    # rm 是 rm -i 的别名，除此之外 ll 是 ls -l 的别名
rm is aliased to `rm -i'
```

查看 *$PATH* 环境变量的内容：

``` bash
[root@localhost ~]\# echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin
# 可执行文件搜索相关
[root@localhost opt]\# type which    # which 也是别名
which is aliased to `(alias; declare -f) | /usr/bin/which --tty-only --read-alias --read-functions --show-tilde --show-dot`
[root@localhost opt]\# which top    # 查看命令可执行文件的绝对路径
/usr/bin/top
[root@localhost opt]\# type whereis    # 我之前执行过 whereis 此处 whereis 是从缓存表里加载的 
whereis is hashed (/usr/bin/whereis)
[root@localhost opt]\# whereis top    # 查看命令可执行文件绝对路径及帮助文档绝对路径
top: /usr/bin/top /usr/share/man/man1/top.1.gz
# 查看命令的缓存表
[root@localhost opt]\# hash
hits    command
   3    /usr/bin/whereis
   1    /usr/bin/ls
   1    /usr/bin/clear
```

​Shell 命令的执行优先级为：绝对路径或相对路径执行的命令 > 别名指定的命令 > Shell 内部命令 > hash 表中的命令 > $PATH 环境变量中指示的路径按序查找到的第一个命令。

### 执行当前工作目录中的可执行文件

​根据上面的结论可知，一般情况下，shell 是不会在当前工作目录搜索可执行文件的，因此我们可以通过指定当前可执行文件的绝对路径或相对路径来执行该命令，最简单方式自然是指定相对路径，linux 以 “`.`” 代表当前工作目录，因此执行当前工作目录下的命令 command 则可以通过以下方式（推荐）：

``` bash
[user@host ~]\$ ./command    # 其中 command 命令可执行文件的绝对路径为 /home/user/command
```

> 🔴 把 `./` 添加到环境变量 \$PATH 中也可以实现输入命令名直接查找执行的效果，但是这种方式并不安全。

``` bash
[user@host ~]\$ export $PATH=./:$PATH    # 这种方式最不安全，如将公共目录中的病毒程序名字取为 ls ，任何用户在此工作目录中都会优先执行病毒程序
[user@host ~]\$ export $PATH=$PATH:./    # 如果一定要设置的话可以这样设置：如果自己的命令名和系统的一样的话会优先执行系统的，自己的命令并不执行
[user@host ~]\$ command    # 配置好环境变量后执行上面的例子
```
