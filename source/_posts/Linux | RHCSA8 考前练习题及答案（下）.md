---
title: RHCSA8 考前练习题及答案（下）
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

## 附加题：有可能出（ServerA 或 Node1）

### 添加 sudo 免密操作

允许 sysmgrs 组成员 sudo 时不需要密码。

```shell
# visudo 区别于直接改配置文件，退出前有语法检查
[root@node1 ~]\# visudo
# %wheel        ALL=(ALL)       NOPASSWD: ALL
%sysmgrs ALL=(ALL) NOPASSWD: ALL

# 测试
[root@node1 ~]\# su - natasha
[natasha@node1 ~]$ sudo useradd zhangsan
[natasha@node1 ~]$ sudo userdel -r zhangsan
```

### 配置创建新用户的密码策略

创建新用户时，默认密码策略为 20 天后，密码会过期。

```shell
[root@node1 ~]\# vim /etc/login.defs
PASS_MAX_DAYS 20

# 测试：
[root@node1 ~]\# useradd ligoudan
[root@node1 ~]\# chage -l ligoudan
Last password change : Apr 29, 2021
Password expires : May 19, 2021
Password inactive : never
Account expires : never
Minimum number of days between password change : 0
Maximum number of days between password change : 20
Number of days of warning before password expires : 
```

### 创建脚本

创建一个名为 myresearch 的脚本该脚本放置在 /usr/bin 下该脚本用来查找 /usr 下所有小于 10m 且具有修改组 ID 权限的文件，将这些文件放置于/root/myfiles 下

> 📌 -size的参数注意 M 大写，k 小写

```shell
[root@node1 ~]\# vim /usr/bin/myresearch
#!/bin/bash
if [ ! -d /root/myfiles ]; then
mkdir /root/myfiles
fi
find /usr -size -10M -perm /g=s -exec cp -a {} /root/myfiles \;

# 或者
mkdir /root/myfiles 
find /usr -size -10M -perm -g=s -exec cp -a {} /root/myfiles \;

[root@node1 ~]\# chmod +x /usr/bin/myresearch
[root@node1 ~]\# /usr/bin/myresearch

# 测试
[root@node1 ~]\# ll -h /root/myfiles
-rwx--s--x. 1 root slocate 47K Aug 12 2018 locate
-rwx--s--x. 1 root lock 22K Aug 12 2018 lockdev
-r-xr-sr-x. 1 root ssh_keys 464K Nov 26 2018 ssh-keysign
-rwx--s--x. 1 root utmp 13K Aug 12 2018 utempter
-rwxr-sr-x. 1 root tty 23K Dec 11 2018 write
```

### 创建脚本

创建一个名为 newsearch 的脚本1、该脚本放置在 /usr/bin 下2、该脚本用来查找 /usr 下所有大于 30k ，但是小于 50k 且具有 SUID 权限的文件，将这些文件放置于 /root/newfiles 下

> 📌 UID 权限则为 -u=s 或 /u=s，GID 权限则为 -g=s 或 /g=s
> 
> 📌 -size 的参数注意 M 大写，k 小写

```shell
# 创建一个文本文件，判断语法以 if 开头 fi 结尾，-d 检查是否目录
[root@node1 ~]\# vim /usr/bin/newsearch
#!/bin/bash
if [ ! -d /root/newfiles ];then
mkdir /root/newfiles
fi
find /usr -size +30k -size -50k -perm /u=s -exec cp -a {} /root/newfiles \;

# 或者
mkdir /root/newfiles
find /usr -size +30k -size -50k -perm /u=s -exec cp -a {} /root/newfiles \;

# 所有人都有权限
[root@node1 ~]\# chmod +x /usr/bin/newsearch
# 运行脚本
[root@node1 ~]\# /usr/bin/newsearch

# 检查
[root@node1 ~]\# ll -h /root/newfiles 
-rws--x--x. 1 root root 33K Dec 17 2019 chfn
-rwsr-x---. 1 root cockpit-wsinstance 46K Mar 12 2020 cockpit-session
-rwsr-xr-x. 1 root root 33K Dec 13 2019 passwd
-rwsr-xr-x. 1 root root 33K Dec 17 2019 umount
-rwsr-xr-x. 1 root root 37K Dec 19 2019 unix_chkpwd
-rws--x--x. 1 root root 47K Nov 20 2018
```

### 设置默认权限

用户 manalo 在 node1 上，所有新创建的文件都应具有 -r--r--r-- 的默认权限，此用户的所有新创建目录应具有 dr-xr-xr-x 的默认权。

```shell
[root@node1 ~]\# su - manalo

# umask 之间没有等号，~/.bashrc = /home/manalo/.bashrc
[manalo@node1 ~]$ vim ~/.bashrc
# User specific aliases and functions
umask 222
[manalo@node1 ~]$ souce ~/.bashrc # 配置生效

# 检查
[manalo@node1 ~]$ touch a.txt
[manalo@node1 ~]$ mkdir dir1
[manalo@node1 ~]$ ll
total 0    -r--r--r--. 1 a a 0 Oct 26 18:04 a.txt    dr-xr-xr-x. 2 a a 6 
```

### 配置容器使其自动启动（B 卷）

利用注册服务器上的 rsyslog 镜像，创建一个名为 logger 的容器

1. 面向 wallah 用户，配置一个 systemd 服务
2. 该服务命名为 `container-logger` ，并在系统重启时自动启动，无需干预
3. 将服务配置为在启动时自动将 /home/wallah/var_log` 挂载到容器中的 /var/log 下
4. 在容器中执行命令 podman exec logger logger -p authpriv.info SUIBIAN

```shell
[root@node1 ~]\# ssh wallah@node1
[wallah@node1 ~]$ podman login -u admin -p redhat321
login Succeeded!
[wallah@node1 ~]$ podman search registry.domain250.example.com/
INDEX NAME DESCRIPTION STARS OFFICIAL AUTOMATED
example.com registry.domain250.example.com/rhel8/rsyslog 0
[wallah@node1 ~]$ podman run -d --name logger -v /home/wallah/var_log:/var/log:Z registry.domain250.example.com/rhel8/rsyslog
[wallah@node1 ~]$ podman stop logger
[wallah@node1 ~]$ loginctl enable-linger
[wallah@node1 ~]$ loginctl show-user wallah
[wallah@node1 ~]$ mkdir -p ~/.config/systemd/user/
[wallah@node1 ~]$ cd ~/.config/systemd/user/
[wallah@node1 ~]$ podman generate systemd -n logger -f
[wallah@node1 ~]$ systemctl --user enable --now container-logger
[wallah@node1 ~]$ systemctl --user status container-logger

# 测试
[wallah@node1 ~]$ podman exec logger logger -p authpriv.info SUIBIAN
[wallah@node1 ~]$ grep SUIBIAN /home/wallah/var_log/secure
2021-08-13T11:45:37.824980+00:00 574761d4236a root: SUIBIAN
[wallah@node1 ~]$ exit
[root@node1 ~]\# reboot
[root@node1 ~]\# ps -aux | grep podman
```

### 容器 nginx

利用注册服务器上的 nginx 镜像，创建一个名为 nginx 的容器

1. 面向 wallah 用户，配置一个 systemd 服务
   
   该服务命名为 `container-nginx`，并在系统重启时自动启动，无需干预

2. 在 /home/wallah/www 下创建文件 index.html ，内容为 hello nginx

3. 将服务配置为在启动时自动将 /home/wallah/www 挂载到容器中的 /usr/share/nginx/html 下将容器主机上的端口 8080 映射到容器上的端口 80

```shell
[root@node1 ~]\# ssh wallah@node1
[wallah@node1 ~]$
[wallah@node1 ~]$ podman login registry.domain250.example.com
Username: admin
Password: redhat321
login Succeeded!
[wallah@node1 ~]$ podman search registry.domain250.example.com/
INDEX NAME DESCRIPTION STARS OFFICIAL AUTOMATED    
example.com registry.domain250.example.com/library/nginx 0

[wallah@node1 ~]$ mkdir /home/wallah/www    
[wallah@node1 ~]$ echo "hello nginx" > /home/wallah/www/index.html    
[wallah@node1 ~]$ podman run -d --name nginx -v /home/wallah/www:/usr/share/nginx.html:Z -p 8080:80 registry.domain250.example.com/library/nginx

[wallah@node1 ~]$ loginctl enable-linger    
[wallah@node1 ~]$ loginctl show-user wallah

[wallah@node1 ~]$ mkdir -p ~/.config/systemd/user/
[wallah@node1 ~]$ cd ~/.config/systemd/user/
[wallah@node1 ~]$ podman generate systemd -n nginx -f
[wallah@node1 ~]$ podman stop nginx
[wallah@node1 ~]$ systemctl --user enable --now container-nginx 
[wallah@node1 ~]$ systemctl --user status container-nginx

# 测试
[wallah@node1 ~]$ curl localhost:8080
hello nginx
[wallah@node1 ~]$ exit
[wallah@node1 ~]\# reboot
[wallah@node1 ~]\# curl localhost:8080
[wallah@node1 ~]\# ss -antup | grep 8080
[wallah@node1 ~]\# ps -aux | grep
```

## 在 Node2 上完成以下任务

### 重置 root 密码

将 node2 主机的密码设置成 redhat。你需要获得系统访问权限才能进行此操作。

**答案**：重新启动机器，进入 grub 界面后按 `e` 进入启动配置文件编辑界面，在 linux 命令行前的启动选项后面增加 `rd.break`，按 **CTRL + X** 键保存并退出编辑。然后进行以下操作：

1. 以 <b>读+写</b> 形式重新挂载 */sysroot*；

```bash
switch_root:\# mount -o remount,rw /sysroot
```

2. 切换为 `chroot` 存放位置，其中 */sysroot* 被视为文件系统树的根；

```bash
switch_root:\# chroot /sysroot
```

3. 设置新 root 密码；

```bash
sh-4.4\# echo redhat | passwd --stdin root
```

4. 确保所有未标记的文件（包括此时的 */etc/shadow*）在启动过程中都会重新获得标记；

```bash
sh-4.4\# touch /.autorelabel
```

5. 键入 `exit` 命令并执行两次，分别退出 `chroot` 和 *initramfs* shell。

此时，系统将继续进行启动，执行完整的 SELinux 重新标记，然后再次重新启动。

### 配置 YUM 源

给系统配置默认存储库，要求如下：

YUM 的 两 个 存 储 库 的 地 址 分 别 是 ： ftp://host.domain8.example.com/dvd/BaseOS 和 ftp://host.domain8.example.com/dvd/AppStream

```bash
# 见 ServerA 配置 yum/dnf 仓库部分 或：
[root@node2 ~]\# scp root@server1:/etc/yum.repos.d/Default.repo /etc/yum.repos.d/
```

### 调整逻辑卷的大小

将名字为 vo 的逻辑卷的大小调整到 200MiB，确保文件系统的内容保持不变。调整后的逻辑卷的大小范围在 180MiB 到 220MiB 的范围内都是可以接受的。

```bash
[root@node2 ~]\# df -h
[root@node2 ~]\# vgs myvol    # 查看 vo 存储组的信息
[root@node2 ~]\# lvextend -L 200M /dev/myvol/vo    # 将逻辑卷 vo 的大小正好调整为 200MiB
[root@node2 ~]\# resize2fs /dev/myvol/vo    # 扩展 vo 的文件系统大小
```

### 创建交换分区

向 node2 添加一个额外的交换分区 756MiB。交换分区应在系统启动时自动挂载。不要删除或以任何方式改动系统上的任何现有交换分区。

```bash
[root@node2 ~]\# lsblk    # 查看存储信息，以规划交换分区的位置
[root@node2 ~]\# fdisk /dev/sdb

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (3,4, default 3): 
First sector (2095106-8388607, default 2097152): 
Last sector, +sectors or +size{K,M,G,T,P} (2097152-8388607, default 8388607): +756M

Created a new partition 3 of type 'Linux' and of size 756 MiB.

Command (m for help): w
The partition table has been altered.
Syncing disks.

[root@node2 ~]\# mkswap /dev/vdb3    # 标记交换分区,查看新创建的交换分区的 UUID
[root@node2 ~]\# vim /etc/fstab    # 写入 fstab 文件
UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"    swap    swap    defaults    0    0
[root@node2 ~]\# swapon -a    # 启用交换分区
```

### 创建逻辑卷

根据如下要求，创建新的逻辑卷：

1. 逻辑卷的名字是 db，卷组是 dbgroup，大小是 60 个 PE size
2. dbgroup 的 PE size 是 16MiB
3. 格式化成 xfs 文件系统。并在系统启动时自动挂载到 /mnt/data

```bash
[root@node2 ~]\# lsblk    # 查看存储信息，以规划逻辑卷的位置
[root@node2 ~]\# fdisk /dev/vdb    # 创建主分区

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type
   p   primary (3 primary, 0 extended, 1 free)
   e   extended (container for logical partitions)
Select (default e): p

Selected partition 4
First sector (2095106-8388607, default 3645440): 
Last sector, +sectors or +size{K,M,G,T,P} (3645440-8388607, default 8388607): 

Created a new partition 4 of type 'Linux' and of size 2.3 GiB.

Command (m for help): w
The partition table has been altered.
Syncing disks.

[root@node2 ~]\# pvcreate /dev/vdb4    # 将 /dev/vdb? 创建为物理卷
[root@node2 ~]\# vgcreate -s 16M dbgroup /dev/vdb4    # 创建卷组 dbgroup，PE 大小为 16MiB,并将 /dev/vdb? 加入
[root@node2 ~]\# lvcreate -l 60 -n db dbgroup    # 从卷组 dbgroup 创建逻辑卷，大小是 60 个物理区块，名字是 db
[root@node2 ~]\# mkfs.xfs /dev/dbgroup/db    # 为逻辑卷创建文件系统
[root@node2 ~]\# mkdir /mnt/data    # 创建挂载点
[root@node2 ~]\# blkid /dev/dbgroup/db    # 查看逻辑卷的 UUID
[root@node2 ~]\# vim /etc/fstab    # 编辑 fstab，添加挂在信息
UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"    /mnt/data    xfs    defaults     0    0
[root@node2 ~]\# mount -a    # 立即挂载
```

### 创建 vdo 卷

根据如下要求，创建新的 VDO 卷：

1. 使用未分区的磁盘
2. 该卷的名称为 vdotest
3. 该卷的逻辑大小是 50G
4. 该卷使用 xfs 文件系统格式化
5. 该卷在系统启动时挂载到 /mnt/vdotest

```bash
[root@node2 ~]\# lsblk    # 查看存储信息，以下假设 /dev/vdc 还未分区
[root@node2 ~]\# yum install vdo kmod-kvdo    # 若未安装 vdo、kmod-kvdo 则需要安装
[root@node2 ~]\# vdo create --name=vdotest --device=/dev/vdc --vdoLogicalSize=50G    # 创建 vdo 卷
[root@node2 ~]\# mkfs.xfs -K /dev/mapper/vdotest    # 格式化该 vdo 卷
# 挂载方法一
[root@node2 ~]\# udevadm settle    # 自动挂载
# 挂载方法二
[root@node2 ~]\# mkdir /mnt/vdotest; blkid /dev/mapper/vdotest; vim /etc/fstab    # 手动挂载
UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"    /mnt/vdotest    defaults, _netdev    0 0
[root@node2 ~]\# mount -a
```

### 配置系统调优

为你的系统选择建议的’tuned’配置集并将它设置为默认设置。

```bash
[root@node2 ~]\# tuned-adm recommend     # 查看推荐设置
virtual-guest
[root@node2 ~]\# tuned-adm profile virtual-guest    # 设置推荐设置为默认
```
