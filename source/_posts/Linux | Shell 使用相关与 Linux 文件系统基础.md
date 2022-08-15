---
title: Shell 使用相关与 Linux 文件系统基础
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

## Shell 使用相关

### 在图形界面中打开终端

​最常用的是找终端的图标单击打开，也可以使用快捷键 "`Alt` + `F2`" 打开 ”Enter a command“ 窗口（类似于 windows 上的 ”运行“ 窗口），输入 ”`gnome-terminal`“ 按回车即可打开图形界面自带的终端。

### 终端上常用的快捷键操作

a. tab：可以实现命令的自动补全（当所输入的内容可以推断出唯一结果时），连按两次会给出所有可能匹配的结果。（此功能由软件包 bash-completion 提供，最小化安装操作系统时可能不会自动安装）；

b. 调整光标位置或内容调整：

| 快捷键 | 效果  |
| --- | --- |
| Ctrl + A | 跳到命令行语句的开头 |
| Ctrl + E | 跳到命令行语句的末尾 |
| Ctrl + U | 将光标处到命令行语句开头的内容清除 |
| Ctrl + K | 将光标处到命令行语句末尾的内容清除 |
| Ctrl + ← | 跳到命令行语句中前一字的开头 |
| Ctrl + → | 跳到命令行语句中下一字的末尾 |
| Ctrl + R | 在历史记录列表中搜索某一模式的命令 |
| Esc + . | 插入执行的上条命令的参数 |

### Shell 历史命令操作

​使用 `history` 命令可以查看历史执行过的命令。

```bash
[root@localhost ~]\# type history
history is a shell builtin        # history 命令是 bash 内建命令
[root@localhost ~]\# history    # 使用 history 命令可以查看之前执行过的命令
    1  ip addr show
    2  clear
    3  vim /etc/sysconfig/network-scripts/ifcfg-ens160
    4  ls /etc/sysconfig/network-scripts/
    5  ls
    6  ls -a /etc/sysconfig/network-scripts/
    7  ll -a /etc/sysconfig/network-scripts/
    8  clear
    9  vim /etc/sysconfig/network-scripts/ifcfg-ens160 
   10  reboot
   11  ip addr show
   12  type history
   13  history
```

​此时如果还想执行之前执行过的第 7 行命令，则只需执行：

```bash
[root@localhost ~]\# !7                    # 使用 "!" 后跟历史序号即可执行 history 结果里的第该序号条命令
ll -a /etc/sysconfig/network-scripts/    # 执行的历史命令的内容会首先列出
total 8                                    # 然后是执行结果
drwxr-xr-x. 2 root root   26 Apr  9 06:01 .
drwxr-xr-x. 6 root root 4096 Apr  9 04:39 ..
-rw-r--r--. 1 root root  282 Apr  9 06:01 ifcfg-ens160
```

​特殊的，如果想再次执行上一次执行的命令，如下例所示：

```bash
[root@localhost ~]\# type file    # 执行 ”!!“ 前的上一条命令
file is /usr/bin/file
[root@localhost ~]\# !!    # 使用 ”!!“ 再次执行
type file                # 同样会先显示执行内容
file is /usr/bin/file    # 执行结果
# sudo !! 的例子
[student@localhost root]\$ dnf update    # 该普通用户没有权限，无法升级本地软件包
2021-04-10 22:13:38,682 [ERROR] dnf:5314:MainThread @logutil.py:194 - [Errno 13] Permission denied: '/var/log/rhsm/rhsm.log' - Further logging output will be written to stderr
Not root, Subscription Management repositories not updated
[student@localhost root]\$ sudo !!    # 使用 sudo 提升当前用户的权限再次执行上次未成功的操作（前提是当前用户是管理员，即在 sudoers 里）
sudo dnf update
Updating Subscription Management repositories.
```

### Shell 配置文件

| 文件路径 | 描述  |
| --- | --- |
| \~/.bashrc 和 /etc/bashrc | 分别对应当前用户和全局用户的 bash 的一些个性化设置，如命令别名、函数等，通常不建议直接修改 /etc/bashrc 而应该在 /etc/profile.d/ 目录中添加自己的脚本 |
| ~/.bash_profile 和 /etc/profile | 分别对应当前用户和全局用户的 bash 环境变量以及启动程序等，通常不建议直接修改 /etc/bashrc 而应该在 /etc/profile.d/ 目录中添加自己的脚本 |
| ~/.bash_history | 存储当前用户终端中执行过的历史命令，环境变量 `$HISSIZE` 定义其存储大小 |
| ~/.bash_logout | 存储当前用户登出终端时要执行的命令 |

> 🟢 配置文件读取情景及读取顺序
> 
> 1. 图形界面登陆时：/etc/profile -> ~/.profile
> 2. 登陆后，打开终端后：-> /etc/bashrc -> ~/.bashrc
> 3. 文本模式登陆时：/etc/bashrc -> /etc/profile -> ~/.bash_profile
> 4. 从其它用户 `su` 到该用户，则分两种情况：
>   a. 带 `-l, -, --login` 参数：/etc/bash -> /etc/profile -> ~/.bash_profile
>   b. 没有带 `-l` 参数：/etc/bashrc -> /bashrc
> 5. 注销时，或退出 `su -l` 登陆的用户时会读取：~/.bash_logout
> 6. 执行自定义的 shell 文件时，若使用 `bash -l a.sh` 的方式，则 bash 会读取：/etc/profile 和 ~/.bash_profile，若使用其他方式，如：`bash a.sh`，`./a.sh`，`sh a.sh` 则不会读取上面任何文件。

## Linux 文件系统基础

​大多数 linux 发行版采用 FHS（Filesystem Hierarchy Standard,文件系统层次化标准）来组织文件系统结构，这是一种以一个根节点开始的倒置的树形结构，FHS定义了系统中每个区域的用途、所需要的最小构成的文件和目录，同时还给出了例外处理与矛盾处理。

​FHS 定义了两层规范，一是 linux 文件系统根目录（即 "`/`" ）下的各个目录该放什么文件数据，二是针对/usr及/var这两个目录的子目录来进行了更深一层的定义。

​Linux 的文件结构树根据每个人的喜好不同可能会被组织成各式各样的形式，这无疑会导致一种混乱的局势，FHS 一定程度上提供了一种共识，是即使是不同的 linux 发行版不会因为文件结构的混乱产生很大的学习成本。

​在Linux文件系统中有两个特殊的目录，一个用户所在的工作目录，也叫当前目录，可以使用一个点来表示；另一个是当前目录的上一级目录，也叫父目录，可以使用两个点来表示：

* . ：代表当前的目录，也可以使用 ./ 来表示；
  
* .. ：代表上一层目录，也可以使用 ../ 来代表。
  

​如果一个目录或文件名以一个点 "`.`" 开始，表示这个目录或文件是一个隐藏目录或文件(如：.bashrc)，以默认方式查找时，不显示该目录或文件。

### linux 文件系统中的一些重要目录及其作用

<table>    <tr>        <th>根目录</th>        <th>一级目录</th>        <th>二级目录</th>        <th>描述</th>    </tr>    <tr>        <td rowspan="33">/</td>        <td>boot</td>        <td></td>        <td>系统启动目录，包含可引导的 linux 内核和引导装载配置文件，删除后虚拟机无法启动</td>    </tr>    <tr>        <td>root</td>        <td></td>        <td>root 用户的家目录</td>    </tr>    <tr>        <td>home</td>        <td>student</td>        <td>/home是所有普通用户家目录存放的路径，普通用户一般再次路径下拥有和自己用户名一样的家目录，用以存放保存的文件、个人配置文件等，此例中有一个 student 用户的家目录：/home/student</td>    </tr>    <tr>        <td rowspan="7">usr</td>        <td>bin</td>        <td>见/bin，/bin下的命令在任何运行级别都可以使用，此路径下的命令在单用户模式下不能执行</td>    </tr>    <tr>        <td>sbin</td>        <td>见/sbin</td>    </tr>    <tr>        <td>local</td>        <td>手工安装的软件保存位置，一般建议源码包软件安装在这个位置</td>    </tr>    <tr>        <td>tmp</td>        <td>存放系统运行过程中产生的临时文件，其真实目录在/var/tmp，此目录存放需要长期保存的临时文件，未访问、修改、更改超 30 天的文件会被删除</td>    </tr>    <tr>        <td>local</td>        <td>手工安装的软件保存位置，一般建议源码包软件安装在这个位置</td>    </tr>    <tr>        <td>share</td>        <td>应用程序的资源文件保存位置，如帮助文档、说明文档和字体目录</td>    </tr>    <tr>        <td>include</td>        <td>C/C++ 等编程语言头文件的放置目录</td>    </tr>    <tr>        <td>bin</td>        <td></td>        <td>二进制文件目录的链接，真实目录为/usr/bin，普通用户和 root 用户均可操作，包含常用的 linux 用户命令</td>    </tr>    <tr>        <td>sbin</td>        <td></td>        <td>保存与系统环境设置相关的命令，只有 root 可以使用这些命令进行系统环境设置，真实目录在/usr/sbin</td>    </tr>    <tr>        <td>tmp</td>        <td></td>        <td>存放系统运行过程中产生的临时文件，存放的文件需要的生命周期较短，有些系统会十分频繁的清理这个目录（RHEL 自动删除未访问、修改、更改超 10 天的目录），为了追求读取和写入迅速有些系统甚至会将这个目录挂载到 RAM 上。在系统启动阶段，许多启动脚本将这个目录作为临时文件存放目录</td>    </tr>    <tr>        <td>lib</td>        <td></td>        <td>系统的动态库目录，存放/bin和/sbin目录中二进制文件依赖的运行库文件。几乎所有的应用程序都会用到该目录下的共享库</td>    </tr>    <tr>        <td>lib64</td>        <td></td>        <td>存放针对64位系统的特定的一些额外动态库的目录</td>    </tr>    <tr>        <td>opt</td>        <td></td>        <td>可选目录。一般用以存放第三方软件和数据文件</td>    </tr>    <tr>        <td>etc</td>        <td></td>        <td>配置文件保存位置。系统内所有采用默认安装方式的服务配置文件全部保存在此目录中，如用户信息、服务的启动脚本、常用服务的配置文件等</td>    </tr>    <tr>        <td>srv</td>        <td></td>        <td>服务目录，service 的缩写，主要用来存储本机或本服务器提供的服务或数据</td>    </tr>    <tr>        <td>sys</td>        <td></td>        <td>于2.6内核引入，存放sysfs在内存中，sysfs文件系统集成了下面3种信息：针对进程信息的proc文件系统、针对设备的devfs文件系统以及针对伪终端的devpts文件系统。该文件系统是内核设备树的一个直观反映。当一个内核对象被创建的时候，对应的文件和目录也在内核对象子系统中</td>    </tr>    <tr>        <td rowspan="6">var</td>        <td>tmp</td>        <td>/var为可变目录，用以存放经常变化的文件，如日志文件；/var/tmp见/usr/tmp</td>    </tr>    <tr>        <td>lib</td>        <td>程序运行中需要调用或改变的数据保存位置。如 MySQL 的数据库保存在 /var/lib/mysql 目录中</td>    </tr>    <tr>        <td>log</td>        <td>登陆文件放置的目录，其中所包含比较重要的文件如 /var/log/messages, /var/log/wtmp 等</td>    </tr>    <tr>        <td>run</td>        <td>一些服务和程序运行后，它们的 PID（进程 ID）保存位置</td>    </tr>    <tr>        <td>spool</td>        <td>里面主要都是一些临时存放，随时会被用户所调用的数据，例如 /var/spool/mail 存放新收到的邮件，/var/spool/cron/存放系统定时任务</td>    </tr>    <tr>        <td>nis和yp</td>        <td>NIS 服务机制所使用的目录，/var/nis 主要记录所有网络中每一个 client 的连接信息；/var/yp 是 linux 的 nis 服务的日志文件存放的目录</td>    </tr>    <tr>        <td>run</td>        <td></td>        <td>一个临时文件系统，存储系统启动以来的信息（PID和锁文件）。当系统重启时，这个目录下的文件应该被删掉或清除。如果你的系统上有 /var/run 目录，应该让它指向 run</td>    </tr>    <tr>        <td rowspan="4">proc</td>        <td colspan="2">虚拟文件系统，数据直接保存在内存中，将内核、进程、外部设备和网络状态归档为文本文件（系统信息都存放在这目录下），例如：uptime、network。在 linux 中，对应 procfs 格式挂载，目录下文件只能看不能改</td>    </tr>    <tr>        <td>cpuinfo</td>        <td>保存 CPU 信息</td>    </tr>    <tr>        <td>devices</td>        <td>保存设备驱动列表</td>    </tr>    <tr>        <td>net</td>        <td>保存网络协议信息</td>    </tr>    <tr>        <td>dev</td>        <td></td>        <td>设备文件，任何设备与接口设备都是以文件形式存在于这个目录的，包括终端设备 tty、软盘 fd、硬盘 hd、RAM ram 和 CD-ROM cd等，用户通过设备文件直接访问这些设备</td>    </tr>    <tr>        <td>mnt</td>        <td></td>        <td>临时挂载的文件系统，比如移动硬盘, U盘和其他操作系统的分区</td>    </tr>    <tr>        <td>media</td>        <td></td>        <td>媒体目录，也是挂载点目录，提供挂载和自动挂载设备的标准位置，如远程访问文件系统和可移动介质</td>    </tr></table>

### 文件系统归类：

1. 系统启动必须：/boot、/etc、/lib、/sys；
  
2. 二进制指令集合：/bin、/sbin；
  
3. 外部文件管理：/dev、/media、/mnt；
  
4. 账户相关：/root、/home、/usr、/usr/bin、/usr/sbin、/usr/src；
  
5. 运行过程相关：/var、/proc；
  
6. 扩展用的：/opt、/srv；
  

### 其它常用的重要文件位置及作用：

| 位置  | 作用  |
| --- | --- |
| /etc/yum.repos.d/redhat.repo | RHEL YUM 仓库的配置文件（安装、更新软件相关） |
| /etc/sysconfig/network-scripts/ifcfg-ens160 | 网卡配置文件（连接外网） |
| /boot/vmlinuz-4.18.0-240.el8.x86_64 | 内核文件 |
| /boot/initramfs-4.18.0-240.el8.x86_64.img | 是内存中的磁盘结构的实现，其中包含必要的工具和脚本，用于在将控制权交给根文件系统上的 init 应用程序之前挂载所需的文件系统 |
| /boot/grub2/.. | 与引导程序有关的文件 |
| /etc/ssh/sshd.conf | SSH 配置文件 |
| /etc/services | 常用和已注册的网络端口监听列表 |
|     |     |
|     |     |