---
title: 更新和安装软件包
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

​这一章的所有关于软件操作的测试环境为：Fedora33。

## 有关命令摘要

### RPM 命令摘要

| 命令                    | 任务                                            |
| ----------------------- | ----------------------------------------------- |
| rpm -qa                 | 列出当前安装的所有 RPM 软件包                   |
| rpm -q NAME             | 显示系统上安装的 NAME 的版本                    |
| rpm -qi NAME            | 显示有关软件包的详细信息                        |
| rpm -ql NAME            | 列出软件包中含有的所有文件                      |
| rpm -qc NAME            | 列出软件包中含有的配置文件                      |
| rpm -qd NAME            | 列出软件包中含有的文档文件                      |
| rpm -q --changelog NAME | 显示软件包的简短更新日志摘要                    |
| rpm -q --scripts NAME   | 显示软件包在安装、升级或删除时运行的 shell 脚本 |

### DNF 命令摘要

| 命令                      | 任务                           |
| ------------------------- | ------------------------------ |
| dnf list [NAME-PATTERN]   | 按名称列出已安装和可用的软件包 |
| dnf group list            | 列出已安装和可用的软件组       |
| dnf search KEYWORD        | 按关键字搜索软件包             |
| dnf info PKGNAME          | 显示软件包的详细信息           |
| dnf install PKGNAME       | 安装软件包                     |
| dnf group install GRPNAME | 安装软件组                     |
| dnf update [PKGNAME]      | 更新所有（或指定的）软件包     |
| dnf remove PKGNAME        | 卸载软件包                     |
| dnf history               | 查看 DNF 历史事务              |

## 注册系统获得红帽支持

### 使用图形界面工具进行注册

​通过 Gnome 图形界面打开 *Red Hat Subscription Manager*（红帽订阅管理器），进行用户身份验证后进入订阅窗口点击 *Register* 按钮，填写并提交相关信息之后即可订阅成功。关于具体操作不再赘述。

### 从命令行注册

```bash
# 1. 注册系统到红帽账户
[user@host ~]\$ subscrption-manager register --username=USERNAME --password=PASSWORD
# 2. 查看可用的订阅
[user@host ~]\$ subscription-manager list --available | less
# 3. 自动附加订阅
[user@host ~]\$ subscription-manager attach --auto
# 或者从可用订阅列表中的特定池中附加订阅
[user@host ~]\$ subscription-manager attach --pool=poolID
# 4. 查看已用的订阅
[user@host ~]\$ subscription-manager list --consumed
# 5. 取消注册系统
[user@host ~]\$ subscription-manager unregister
```

> *subscription-manager* 也可搭配激活密钥使用而不必使用用户名或密码以为实现自动化安装和部署提供便利。

### 红帽授权证书

​注册红帽后，系统的授权证书存储在 /etc/pki 和其子目录中。

- */etc/pki/product* 中的证书知名系统上安装了哪些红帽产品；
- */etc/pki/consumer* 中的证书指明系统所注册到的红帽账户；
- */etc/pki/entitlement* 中的证书指明该系统附加有哪些订阅。

## RPM 软件包

​*RPM 软件包管理器*最初是由红帽开发的，该程序提供了一种标准的方式来打包软件进行分发。与使用从存档提取到文件系统的软件相比，采用 RPM 软件包形式管理软件要简单得多。管理员可以通过它跟踪软件包所安裝的文件，需要删除哪些软件（如果卸载）并检查确保显示支持软件包（如果安装）。有关已安裝软件包的信息存储在各个系统的本地RPM数据库中。

​红帽为红帽企业 Linux 提供的所有软件都以 RPM 软件包的形式提供。

### RPM 软件包的文件名约定

​RPM 软件包的文件名一般的格式为：name-version-release.architecture.rpm，如：coreutils-8.30-4.el8.x86_64.rpm。

- name 是描述软件内容的一个或多个词语；
- version 是原始软件的版本号；
- release 是基于该版本的软件包的发行版号，由软件打包商设置，后者不一定是原始软件开发商；
- architecture 是编译的软件包运行的处理器架构。*noarch* 表示该软件包的内容不限定架构。

​从软件存储库搜索、下载和安装软件包时只需要软件包的名称。如果存在多个版本，则会安装最高版本。

​每个 RPM 软件包是包含以下三个组成部分的特殊存档：

- 软件包安装的文件；
- 元数据-与软件包有关的信息，如名称、版本、发行版和架构；软件包的摘要和描述；是否要求安装其他软件包；授权许可信息；软件包更改日志以及其他详细信息；
- 在安装、更新或删除此软件包时可能运行的脚本，或者在安装、更新或删除其他软件包时触发的脚本。

​通常，软件提供商使用 GPG 密钥对 RPM 软件包进行数字签名，RPM 会通过 GPG 密钥签名来验证软件包的完整性。若 GPG 签名不匹配，则 RPM 会拒绝安装软件包。

​RPM 更新软件会先删除旧有的软件包（通常会保留配置文件）然后安装新版，除了内核。

### 2. 使用 RPM 实用程序处理本地软件包

​*rpm* 实用程序可获取软件包文件和已安装软件包的内容的相关信息。默认情况下，它从已安装软件包的本地数据库中获取信息。但是可以使用 *-p* 选项来指定想获取有关已下载软件包文件的信息以实现在安装前的检查。

​使用 `rpm` 命令查询本地软件包数据库信息的一般格式为：

```bash
rpm -q [select-options] [query-options]	# 查询本地软件包数据库信息
```

​关于已安装的软件包的一般信息的查询：

```bash
[skinyi@skinyi ~]\$ rpm -qa		# 列出已安装的所有软件包
libgcc-10.2.1-9.fc33.x86_64
linux-firmware-whence-20210208-118.fc33.noarch
tzdata-2021a-1.fc33.noarch
libreport-filesystem-2.14.0-15.fc33.noarch
hwdata-0.345-1.fc33.noarch
......
# rpm -qf FILENAME	查询哪个软件包提供安装的文件
[skinyi@skinyi ~]\$ rpm -qf /etc/yum.repos.d
fedora-repos-33-3.noarch
fedora-repos-33-5.noarch
```

​关于特定软件包的信息查询：

```bash
# 1. rpm -q PKG_NAME		列出当前安装的软件包的版本
[skinyi@skinyi ~]\$ rpm -q dnf
dnf-4.8.0-1.fc33.noarch
# 2. rpm -qi PKG_NAME	获取有关软件包的详细信息
[skinyi@skinyi ~]\$ rpm -qi kernel
Name        : kernel
Version     : 5.10.23
Release     : 200.fc33
Architecture: x86_64
Install Date: 2021年03月19日 星期五 10时09分52秒
Group       : Unspecified
Size        : 0
License     : GPLv2 and Redistributable, no modification permitted
......
# 3. rpm -ql PKG_NAME	列出软件包安装的文件
[skinyi@skinyi ~]\$ rpm -ql openssh
/etc/ssh
/etc/ssh/moduli
/usr/bin/ssh-keygen
/usr/lib/.build-id
/usr/lib/.build-id/4b
......
# 4. rpm -qc PKG_NAME	仅列出软件包安装的配置文件
[skinyi@skinyi ~]\$ rpm -qc openssh-clients
/etc/ssh/ssh_config
/etc/ssh/ssh_config.d/50-redhat.conf
# 5. rpm -qd PKG_NAME	仅列出软件包安装的文档文件
[skinyi@skinyi ~]\$ rpm -qd net-tools
/usr/share/man/de/man5/ethers.5.gz
/usr/share/man/de/man8/arp.8.gz
/usr/share/man/de/man8/ifconfig.8.gz
......
# 6. rpm -q --scripts PKG_NAME	列出在安装或删除软件包之前或之后运行的 shell 脚本
[skinyi@skinyi ~]$ rpm -q --scripts openssh-server
preinstall scriptlet (using /bin/sh):
getent group sshd >/dev/null || groupadd -g 74 -r sshd || :
getent passwd sshd >/dev/null || \
  useradd -c "Privilege-separated SSH" -u 74 -g sshd \
  -s /sbin/nologin -r -d /var/empty/sshd sshd 2> /dev/null || :
postinstall scriptlet (using /bin/sh):

 
if [ $1 -eq 1 ] && [ -x /usr/bin/systemctl ]; then 
        # Initial installation 
        /usr/bin/systemctl --no-reload preset sshd.service sshd.socket || : 
fi 

......
# 7. rpm -q --changelog PKG_NAME	列出软件包的更改信息
[skinyi@skinyi ~]\$ rpm -q --changelog audit
* 三 8月 11 2021 Steve Grubb <sgrubb@redhat.com> 3.0.5-1
- New upstream bugfix release

* 日 8月 08 2021 Steve Grubb <sgrubb@redhat.com> 3.0.4-1
- New upstream feature release
......
# 8. rpm -pq[ilcd...]	PKG_NAME	查询本地软件包（已下载未安装）的上述信息。
```

​安装离线下载的软件包：

```bash
# rpm -ivh name-version-release.arch.rpm	# 安装软件包
# 选项意义：
-i, --install	# 安装软件包
-v, --verbose	# 显示详细信息
-h, --hash	# 安装的时候列出哈希标记
```

> 安装第三方软件时要确保可信，因为在安装过程中允许以 root 用户执行任意脚本。

### 提取 rpm 软件包中的文件

​使用 `rpm2cpio` 命令可以将 rpm 软件包转成格式为 cpio 的特殊归档文件，使用 `cpio` 命令则可以提取 cpio 归档文件的内容。因此，要实现一条命令提取一个软件包中的内容，可以通过管道来实现：

```bash
[user@host ~]\$ rpm2cpio name-version-release.arch.rpm | cpio -id [PATTERN]	# 按照 PATTERN 指定的模式提取 rpm 包中的文件
# cpio 的命令选项
-i, --extract	# 从包中提取文件
-d, --make-directories	# 需要时自动创建目录
```

​提取的文件会存放在当前目录，且会自动创建子目录。

## 使用 DNF 管理软件包

​*DNF* 为 *RPM* 的前端软件，除了可以安装软件包，它也可以实现从软件仓库搜索、下载、解决安装依赖的比 *RPM* 更完善的功能。

​*YUM* 作为软件包管理器的老前辈将要被 *DNF*所取代（由于其性能差、内存占用过多、依赖解析速度变慢等长期存在的问题），故将原教科书中的关于 YUM 介绍的一章换成对 DNF 的介绍。DNF 使用 c、c++、python 语言编写，将依赖管理模块抽象为 libsolv 库来管理软件包的依赖，相比 YUM 提高了性能。DNF 和 YUM 的用法大体上是相同的。

### 使用 DNF 查找软件

```bash
# 1. dnf help	显示用法信息
[skinyi@skinyi ~]\$ dnf help
usage: dnf [options] COMMAND

主要命令列表：

alias                     列出或创建命令别名
autoremove                删除所有原先因为依赖关系安装的不需要的软件包
check                     在包数据库中寻找问题
check-update              检查是否有软件包升级
clean                     删除已缓存的数据
deplist                   [已弃用，请使用 repoquery --deplist] 列出软件包的依赖关系和提供这些软件包的源
distro-sync               同步已经安装的软件包到最新可用版本
downgrade                 降级包
......
# 2. dnf list [KEYWORD]	显示已安装和可用的软件包
[root@skinyi ~]\# dnf list kernel
上次元数据过期检查：2:07:19 前，执行于 2021年09月01日 星期三 14时52分31秒。
已安装的软件包
kernel.x86_64             5.10.23-200.fc33              @updates
kernel.x86_64             5.13.12-100.fc33              @updates
# 3. dnf search KEYWORD	使用关键字搜索软件包
[skinyi@skinyi ~]\$ dnf search remote
上次元数据过期检查：0:02:41 前，执行于 2021年09月01日 星期三 16时43分32秒。
============================ 名称 和 概况 匹配：remote ==============================
R-remotes.noarch : R Package Installation from Remote Repositories
RemoteBox.noarch : Open Source VirtualBox Client with Remote Management
anyremote.x86_64 : Remote control through Wi-Fi or bluetooth connection
anyremote-data.x86_64 : Configuration files for anyRemote
anyremote-doc.x86_64 : Documentation for anyRemote
chrome-remote-desktop.x86_64 : Remote desktop support for google-chrome & chromium
......
# 4. dnf info PKG_NAME	查看软件包的详细信息，包括安装所需磁盘空间
[skinyi@skinyi ~]\$ dnf info httpd
上次元数据过期检查：0:05:28 前，执行于 2021年09月01日 星期三 16时43分32秒。
可安装的软件包
名称         : httpd
版本         : 2.4.48
发布         : 1.fc33
架构         : x86_64
大小         : 1.4 M
源           : httpd-2.4.48-1.fc33.src.rpm
仓库         : updates
概况         : Apache HTTP Server
URL          : https://httpd.apache.org/
协议         : ASL 2.0
描述         : The Apache HTTP Server is a powerful, efficient,
             : and extensible web server.
# 5. dnf provides PATHNAME	显示提供与指定路径名文件或目录相匹配的软件包
[skinyi@skinyi ~]\$ dnf provides /var/www/html
上次元数据过期检查：0:07:46 前，执行于 2021年09月01日 星期三 16时43分32秒。
httpd-filesystem-2.4.46-1.fc33.noarch : The basic directory
     ...: layout for the Apache HTTP Server
仓库        ：fedora
匹配来源：
文件名    ：/var/www/html

httpd-filesystem-2.4.48-1.fc33.noarch : The basic directory
     ...: layout for the Apache HTTP Server
仓库        ：updates
匹配来源：
文件名    ：/var/www/html

```

### 使用 DNF 安装和删除软件

```bash
# 1. dnf install PKG_NAME	# 获取并安装软件包，包括所有依赖项
[root@skinyi ~]\# dnf install git
上次元数据过期检查：2:04:10 前，执行于 2021年09月01日 星期三 14时52分31秒。
依赖关系解决。
================================================================
 软件包                 架构   版本               仓库     大小
================================================================
安装:
 git                    x86_64 2.31.1-3.fc33      updates 122 k
安装依赖关系:
 git-core               x86_64 2.31.1-3.fc33      updates 3.6 M
 git-core-doc           noarch 2.31.1-3.fc33      updates 2.3 M
 perl-AutoLoader        noarch 5.74-471.fc33      updates  31 k
 perl-B                 x86_64 1.80-471.fc33      updates 189 k
 perl-Carp              noarch 1.50-457.fc33      fedora   29 k
 perl-Class-Struct      noarch 0.66-471.fc33      updates  32 k
 perl-Data-Dumper       x86_64 2.174-459.fc33     fedora   56 k
......
# 2. dnf update [PKG_NAME]	# 更新指定软件或所有已安装软件
[root@skinyi ~]\# dnf update kernel
上次元数据过期检查：2:06:19 前，执行于 2021年09月01日 星期三 14时52分31秒。
依赖关系解决。
无需任何处理。
完毕！
```

> 若需查看当前系统运行中的内核，可以使用 `uname` 命令。
>
> ```bash
> [root@skinyi ~]\# uname -r
> 5.10.23-200.fc33.x86_64
> [root@skinyi ~]\# uname -a
> Linux skinyi.fedora 5.10.23-200.fc33.x86_64 \#1 SMP Thu Mar 11 22:18:30 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
> ```

```bash
# 3. dnf remove PKG_NAME	# 删除安装的软件包及其依赖包
[root@skinyi ~]\# dnf remove openssh
依赖关系解决。
================================================================
 软件包             架构      版本            仓库         大小
================================================================
移除:
 openssh            x86_64    8.4p1-7.fc33    @updates    1.8 M
移除依赖的软件包:
 openssh-clients    x86_64    8.4p1-7.fc33    @updates    1.8 M
 openssh-server     x86_64    8.4p1-7.fc33    @updates    924 k

事务概要
================================================================
移除  3 软件包
......
```

### 使用 DNF 安装或删除软件组

*DNF* 也有软件组的概念，软件组是出于特定目的归类到一起安装的相关软件的集合。

​在 RHEL8 中，有两种类型的组：常规组和环境组，其中常规组是软件包的集合，环境组是常规组的集合。一个组提供的软件包或组可能为 *mandatory*（安装该软件组时必须安装）、*default*（安装该组时通常会安装）或 *optional*（安装该组时不予以安装，除非特别要求）。

#### 查看已安装和可用的软件组名称

```bash
[root@skinyi ~]\# dnf group list
上次元数据过期检查：2:17:43 前，执行于 2021年09月01日 星期三 14时52分31秒。
可用环境组：
   Fedora 定制操作系统
   最小化安装
   Fedora Workstation
   Fedora Cloud Server
   ......
已安装的环境组：
   Fedora Server 版本
已安装组：
   无头设备（Headless）管理
可用组：
   3D 打印
   管理工具
   音频制作
   写作和出版
   ......
```

> 有些组一般通过环境组安装，默认为隐藏。可以通过 `dnf group list hidden` 命令列出这些隐藏组。

#### 显示软件组的相关信息

```bash
[root@skinyi ~]\# dnf group info "RPM Development Tools"
上次元数据过期检查：2:31:05 前，执行于 2021年09月01日 星期三 14时52分31秒。
组：RPM 开发工具
 描述：这些工具中包括了核心开发工具，如 rpmbuild。
 必要的软件包：
   redhat-rpm-config
   rpm-build
 默认的软件包：
   koji
   mock
   rpmdevtools
 可选的软件包：
   pungi
   rpmlint
```

#### 安装一个软件组

```bash
[root@skinyi ~]\# dnf group install "Deepin Desktop Environment" --with-optional
上次元数据过期检查：2:35:01 前，执行于 2021年09月01日 星期三 14时52分31秒。
依赖关系解决。
================================================================
 软件包                  架构   版本              仓库     大小
================================================================
安装组/模块包:
 chromium                x86_64 91.0.4472.164-1.fc33
                                                  updates  99 M
 deepin-calculator       x86_64 5.0.1-3.fc33      fedora  286 k
 deepin-calendar         x86_64 5.0.1-4.fc33      fedora  111 k
 deepin-desktop          x86_64 5.0.0-9.fc33.2    updates 1.9 M
 deepin-editor           x86_64 1.2.9.1-8.fc33.1  updates 1.3 M
 deepin-icon-theme       noarch 15.12.71-5.fc33   fedora   11 M
 deepin-image-viewer     x86_64 5.0.0-5.fc33      fedora  3.7 M
 deepin-picker           x86_64 5.0.1-3.fc33      fedora   83 k
 deepin-screenshot       x86_64 5.0.0-4.fc33      fedora  347 k
 ......
```

> *--with-optional* 选项指定安装可选的软件包。

### 查看软件包管理的历史事务

​所有安装和删除软件包的事务日志记录在 */var/log/dnf.rpm.log* 中。

```bash
[root@skinyi ~]\# tail -n 5 /var/log/dnf.rpm.log
2021-09-01T17:22:45+0800 INFO --- logging initialized ---
2021-09-01T17:23:00+0800 INFO --- logging initialized ---
2021-09-01T17:23:36+0800 INFO --- logging initialized ---
2021-09-01T17:25:56+0800 INFO --- logging initialized ---
2021-09-01T17:27:31+0800 INFO --- logging initialized ---
```

​使用 `dnf history` 显示安装和删除事务的摘要：

```bash
[root@skinyi ~]\# dnf history
ID     | 命令行                    | 日期和时间       	| 操作           | 更改   
--------------------------------------------------------------------------------
     3 | update                   | 2021-09-01 14:54 | I, U           |  283 EE
     2 | install nodejs           | 2021-03-19 11:05 | Install        |    6   
     1 |                          | 2021-03-19 10:07 | Install        |  658 EE
```

​使用 `dnf history undo ID` 选项可以撤销指定 ID 的事务。
