---
title: RHCSA8 考前练习题及答案（上）
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

## 在 Node1 上完成以下任务

### 配置网络

在 node1 上配置网卡 ens160 的网络，要求如下： 

* 主机名：node1.domain8.example.com
* IP 地址: 192.168.30.10/24
* 网关：192.168.30.254
* DNS：114.114.114.114

```bash
[root@localhost ~]\# hostnamectl set-hostname node1.domain8.example.com
[root@node1 ~]\# hostname
node1.domain8.example.com
[root@node1 ~]\# nmcli con mod ens160 connection.autoconnect yes \ 
> ipv4.method manual \
> ipv4.address 192.168.30.10/24 \
> ipv4.gateway 192.168.30.254 \
> ipv4.dns 114.114.114.114
[root@node1 ~]\# nmcli con up ens160
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/6)
[root@node1 ~]\# nmcli con show ens160 | grep ipv4
ipv4.method:                            manual
ipv4.dns:                               114.114.114.114
ipv4.dns-search:                        --
ipv4.dns-options:                       --
ipv4.dns-priority:                      0
ipv4.addresses:                         192.168.30.10/24
ipv4.gateway:                           192.168.30.254
......
```

### 配置 yum/dnf 使用指定存储库

给系统配置默认存储库，要求如下： YUM 的 两 个 存 储 库 的 地 址 分 别 是 ：

* ftp://host.domain8.example.com/dvd/BaseOS
* ftp://host.domain8.example.com/dvd/AppStream

```bash
# 1. 方法一：利用 dnf-utils 软件包（需提前找到该包名）
[root@node1 ~]\# rpm -ivh ftp://host.domain8.example.com/dvd/BaseOS/Packages/dnf-utils-4.0.2.2-3.el8.noarch.rpm
[root@node1 ~]\# dnf-config-manager --add-repo ftp://host.domain8.example.com/dvd/BaseOS
[root@node1 ~]\# dnf-config-manager --add-repo ftp://host.domain8.example.com/dvd/AppStream
vim /etc/yum.conf # 编辑修改 gpgcheck=0,如方法二所示

# 2. 方法二：手动编辑添加
[root@node1 ~]\# vim /etc/yum.repos.d/default.repo
[root@node1 ~]\# cat !$
cat /etc/yum.repos.d/default.repo
[baseos]
name=baseos
baseurl=ftp://host.domain8.example.com/dvd/BaseOS
enabled=1
gpgcheck=0

[base]
name=base
baseurl=ftp://host.domain8.example.com/dvd/AppStream
enabled=1
gpgcheck=0

# 3. 验证：
[root@node1 ~]\# yum repolist
Updating Subscription Management repositories.
Unable to read consumer identity

This system is not registered to Red Hat Subscription Management. You can use subscription-manager to register.

repo id                              repo name
base                                 base
baseos                               baseos
```

### 调试 SELinux

非标准端口 82 上运行的 WEB 服务器在提供内容时遇到问题。根据需要调试并解决问题， 并使其满足以下条件： 

1. 系统上的 web 服务器能够提供 /var/www/html 中所有现在有的 html 文件（注意：不要删除或者其他方式改动现有的文件内容）
2. Web 服务器通过 82 端口访问 
3. Web 服务器在系统启动时自动启动

```bash
# 查看原本的 SELinux 设置
[root@node1 ~]\# semanage port -l | grep ^http_port_t        # 查看属于 http_port_t 类型上下文的端口
http_port_t                    tcp      80, 81, 443, 488, 8008, 8009, 8443, 9000
[root@node1 ~]\# ls -lZ /var/www/html | grep html$        # 查看 /var/www/html 中的 html 文件的安全上下文
total 0
-rw-r--r--. 1 root root unconfined_u:object_r:var_t:s0 0 Aug  4 22:52 1.html
-rw-r--r--. 1 root root unconfined_u:object_r:var_t:s0 0 Aug  4 22:52 index.html

# 修改涉及的端口及文件的 SELinux 设置
[root@node1 ~]\# semanage port -a -t http_port_t -p tcp 82
[root@node1 ~]\# restorecon -RFv /var/www/html
Relabeled /var/www/html from unconfined_u:object_r:var_t:s0 to system_u:object_r:httpd_sys_content_t:s0
Relabeled /var/www/html/1.html from unconfined_u:object_r:var_t:s0 to system_u:object_r:httpd_sys_content_t:s0
Relabeled /var/www/html/index.html from unconfined_u:object_r:var_t:s0 to system_u:object_r:httpd_sys_content_t:s0
[root@node1 ~]\# systemctl enable --now httpd

# 测试访问。宿主机访问还需放开防火墙
[root@node1 ~]\# curl localhost:82
```

### 创建用户账户

创建下列用户、用户组，并按要求完成设置： 

1. 组名为 sysmgrs
2. natasha 用户的附属组是 sysmgrs
3. harry 用户的附属组是 sysmgrs
4. john 用户的 shell 是非交互式 shell，且不是 sysmgrs 组的成员
5. natasha、harry、john 的密码是 redhat

```bash
[root@node1 ~]\# groupadd sysmgrs
[root@node1 ~]\# useradd -G sysmgrs natasha    # -p 选项指定的加密后的密码，不推荐使用
[root@node1 ~]\# useradd -G sysmgrs harry
[root@node1 ~]\# useradd -s /sbin/nologin john
[root@node1 ~]\# for i in natasha harry john; do echo redhat | passwd --stdin $i; done
```

### 配置 Cron 作业

配置 cron 作业，该作业每隔 5 分钟运行并执行以下命令：
logger “EX200 in progress”，以用户 natasha 身份运行

```bash
[root@node1 ~]\# crontab -e -u natasha
*/5 * * * * logger "EX200 in progress"
[root@node1 ~]\# tail -f /var/log/cron    # 验证任务
```

### 创建协作目录

创建具有特殊权限的目录 /home/managers，要求如下： 

1. /home/managers 目录属于 sysmgrs 组
2. 此目录可以被 sysmgrs 的组成员读取、写入和访问，但是其他任何用户不具备这些权限。（不包括 root 用户）
3. 在 /home/managers 目录中创建的文件的所属组自动变成 sysmgrs

```bash
[root@node1 ~]\# mkdir /home/managers
[root@node1 ~]\# chgrp sysmgrs /home/managers
# [root@node1 ~]\# chmod 2770 /home/managers
[root@node1 ~]\# chmod g=rws,o=- /home/managers    # g+s 实现目录内新建文件自动继承目录的文件
# 验证
[root@node1 ~]\# ls -ld /home/managers
drwxrws---. 2 root sysmgrs 6 Aug  8 10:46 /home/managers
```

### 配置 NTP

配置 node1 作为 NTP 的客户端，配置同步阿里云的 ntp 授时服务：ntp.aliyun.com。

```bash
[root@node1 ~]\# timedatectl | grep NTP
              NTP service: active
[root@node1 ~]\# systemctl status chronyd
● chronyd.service - NTP client/server
   Loaded: loaded (/usr/lib/systemd/system/chronyd.service; enabled; vendor preset: enabled)
   Active: active (running) (thawing) since Mon 2022-08-08 11:14:54 CST; 10min ago
   ......
[root@node1 ~]\# vim /etc/chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server ntp.aliyun.com iburst
pool 2.rhel.pool.ntp.org iburst

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift
......
[root@node1 ~]\# systemctl restart chronyd
[root@node1 ~]\# chronyc sources    # 203.107.6.88 是域名 ntp.aliyun.com 返回的授时服务 ip
210 Number of sources = 7
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* 203.107.6.88                  2   7   357    70  -6780us[  -10ms] +/-   37ms
......
```

### 配置 autofs

配置 autofs，按照以下要求自动挂载远程用户的家目录，要求如下： 

1. host.domain8.example.com ( 172.25.250.250 ) NFS 导出 /rhome 到您的系统
2. 此文件系统包含为用户 remoteuser1 预配置的主目录
3. remoteuser1 的主目录是 host.domain8.example.com:/rhome/remoteuser1
4. remoteuser1 的主目录应自动挂载到本地 /rhome 下的 /rhome/remoteuser1
5. 主目录必须可供其用户写入
6. remoteuser1 的密码是 redhat

```bash
[root@node1 ~]\# systemctl status autofs
● autofs.service - Automounts filesystems on demand
   Loaded: loaded (/usr/lib/systemd/system/autofs.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
[root@node1 ~]\# vim /etc/auto.master    # 编辑 autofs 的主配置文件
/misc   /etc/auto.misc
# 添加 nfs 条目 - rhome 作为基本挂载点
# | 本地 autofs 挂载点    | 针对该挂载点的配置文件    | (可选)挂载选项
/rhome    /etc/auto.nfs
[root@node1 ~]\# vim /etc/auto.nfs    # 编辑 nfs autofs 的配置
# 添加用户目录及远程 nfs 路径
# | 挂载点（相对于基本挂载点：/rhome/remoteuser1）    | 挂载选项（可读写）    | 远程位置
remoteuser1    -fstype=nfs,rw    host.domain8.example.com:/rhome/remoteuser1    # 以空格隔开
[root@node1 ~]\# systemctl enable --now autofs
[root@node1 ~]\# echo flectrag | passwd --stdin remoteuser1
[root@node1 ~]\# su - remoteuser1
```

### 配置 /var/tmp/fstab 的权限

配置文件权限，将文件 /etc/fstab 复制到 /var/tmp/fstab。配置 /var/tmp/fstab 的权限以满足如下条件： 

1. /var/tmp/fstab 属于 root 用户和 root 组 
2. /var/tmp/fstab 不能被任何人执行
3. 用户 natasha 有读写权限
4. 用户 harry 没有读写权限
5. 所有其他用户（当前或者未来）能够读取 /var/tmp/fstab

```bash
[root@node1 ~]\# cp /etc/fstab /var/tmp/fstab
[root@node1 ~]\# setfacl -m u:natasha:rw /var/tmp/fstab
[root@node1 ~]\# setfacl -m u:harry:- /var/tmp/fstab
[root@node1 ~]\# getfacl /var/tmp/fstab    # 验证
getfacl: Removing leading '/' from absolute path names
# file: var/tmp/fstab
# owner: root
# group: root
user::rw-
user:natasha:rw-
user:harry:---
group::r--
mask::rw-
other::r--
```

### 配置用户账户

配置用户 manalo ，其用户 ID 为 3533。此用户的密码应当为 redhat

```bash
[root@node1 ~]\# useradd -u 3533 manalo
[root@node1 ~]\# echo redhat | passwd --stdin manalo
Changing password for user manalo.
passwd: all authentication tokens updated successfully.
```

### 查找文件

查找属于 manalo 用户所属的文件，并拷贝到 /root/findfiles 目录

```bash
[root@node1 ~]\# mkdir /root/findfiles
[root@node1 ~]\# find / -user manalo -exec cp -a {} /root/findfiles \;
```

### 查找字符串

查找文件 /usr/share/xml/iso-codes/iso_639_3.xml 中包含字符串 ng 的所有行。并将所有这 些行的内容放到文件/root/list 中，/root/list 不得包含空行。

```bash
[root@node1 ~]\# grep ng /usr/share/xml/iso-codes/iso_639_3.xml > /root/list
```

### 创建归档

创建一个名为 /root/backup.tar.gz 的 tar 包，用来压缩 /usr/local 目录。 如果考试让你用的 /root/backup.tar.bz2 的 tar 包呢？

```bash
[root@node1 ~]\# tar -czvf /root/backup.tar.gz /usr/local/    # 压缩为 tar.gz
[root@node1 ~]\# tar -cjvf /root/backup.tar.bz2 /usr/local/    # 压缩为 tar.bz2
[root@node1 ~]\# tar -tf /root/backup.tar.gz    # 验证
```

### 配置容器使其自动启动

利用仓库服务器上的 rsyslog 镜像，创建一个名为 logserver 的容器

> 仓库地址：registry.domain250.example.com，用户名：admin，密码：redhat。

1. 面向 wallah 用户，配置一个 systemd 服务

2. 该服务命名为 container-logserver ，并在系统重启时自动启动，无需干预

### 完成为容器配置持久存储

通过以下方式扩展上一个任务的服务

1. 配置主机系统的 journald 日志以在系统重启后保留数据，并重新启动日志记录服务

2. 将主机 /var/log/journal 目录下任何以 *.journal 的文件复制到 /home/wallah/container_logfile 中

3. 将服务配置为在启动时自动将/home/wallah/container_logfile 挂载到容器中的 /var/log/journal 下

```bash
# jouranld 的日志临时存储在 /run/log/journal/ 目录中，持久化时会保存在 /var/log/journal/ 目录中，该目录无需手动创建
# journald 日志的存储方式在 /etc/systemd/journald.conf 配置文件中定义

# 任务九 1-2：
## 1. 修改 journald 配置文件使其持久化存储日志
[root@node1 ~]\# vim /etc/systemd/journald.conf
[Journal]
#Storage=auto
Storge=persistent
#Compress=yes
[root@node1 ~]\# systemctl enable --now systemd-journald    # 重启（设置自启）journald 服务
## 2. 复制日志
[root@node1 ~]\# journalctl -u systemd-journald    # 查看日志服务生成的日志存储路径（主要是随机字符串目录）
[root@node1 ~]\# cp /var/run/journal/<long_random_string>/*.journal /home/wallah/containe_logfile/
[root@node1 ~]\# chown -R wallah:wallah /home/wallah/container_logfile

# 任务八 1-2，任务九 3：
[root@node1 ~]\# ssh wallah@node1    # 使用 ssh 本地远程 wallah 用户
## 1. 配置 logserver 容器
[wallah@node1 ~]\# podman login registry.domain250.example.com    # 登录到镜像仓库
Username: admin
Password: redhat
Login Succeeded!
[wallah@node1 ~]\# podman search registry.domain250.example.com/    # 查看仓库内的镜像列表
[wallah@node1 ~]\# podman pull registry.domain250.example.com/rhel/rsyslog    # 拉取镜像到本地
[wallah@node1 ~]\# podman run -d --name logserver -v /home/wallah/container_logfile:/var/log/journal:Z registry.domain250.example.com/rhel/rsyslog    # 启动容器，-d 后台运行，--name 指定容器名字，-v 指定目录映射(冒号和Z不可缺)
[wallah@node1 ~]\# podman ps # 查看运行中的容器
## 2. 为容器创建用户服务
[wallah@node1 ~]\# loginctl enable-linger [wallah]    # 允许 wallah 用户逗留（用户登出后其服务保持运行）
[wallah@node1 ~]\# mkdir -p ~/.config/systemd/user/    # 创建用户 systemd 服务配置目录
[wallah@node1 ~]\# cd ~/.config/systemd/user/
[wallah@node1 user]\# podman generate systemd -n logserver -f    # 创建用户服务
/home/wallah/.config/systemd/user/container-logserver.service
[wallah@node1 user]\# podman stop logserver    # 先停止容器以进行后续测试
[wallah@node1 user]\# systemctl --user enable --now container-logserver.service    # 启用用户服务并设置 container-logserver 服务的自启
## 3. 验证
[wallah@node1 user]\# podman ps    # 验证容器是否自启
```
