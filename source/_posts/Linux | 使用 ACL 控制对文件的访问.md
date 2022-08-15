---
title: 使用 ACL 控制对文件的访问
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

## 概述

​ACL,即访问控制列表，它可以根据实际要求指定多个用户和组以不同的文件权限集来访问文件。借助 ACL，可以使用与常规文件权限相同的权限标志（读取、写入和执行）向由用户名、组名、UID 或 GID 标识的多个用户和组授予权限。除了文件所有者和文件的组从属关系之外，这些额外的用户和组分别被称为指定用户和指定组，因为他们不是在长列表中指定的，而在 ACL 中指定的。

​用户可以对属于自己的文件和目录设置 ACL。被分配了 *CAP_FOWNER* Linux 功能的特权用户可以对任何文件或目录设置 ACL。新文件和子目录会自动从父目录的默认 ACL（若已设置）中继承 ACL 设置。与常规文件的访问规则相似，父目录层次结构至少需要其他搜索（执行）权限集，以便启用指定用户和指定组的访问权限。

​挂载的文件系统需指定启用 ACL 支持，其中 XFS 文件系统内置有 ACL 支持，ext4 或 ext4 默认情况下会启用 acl 选项，但在早期版本中，应确认是否启用了 ACL 支持。要启用文件系统 ACL 支持，须在 `mount` 命令或 */etc/fstab* 配置文件的文件系统条目中使用 ACL 选项。

## 查看 ACL 权限

​使用 `ls -l` 命令仅查看文件最少的 ACL 设置详细信息：

```bash
[root@localhost ~]\# ls -l
total 16
-rw-r--r--. 1 root root   18 Sep 16 17:01 1
-rw-------. 1 root root 2760 Jun  3 07:16 anaconda-ks.cfg
-rw-------. 1 root root 2078 Jun  3 07:16 original-ks.cfg
-rw-r--r--. 1 root root  140 Sep 16 17:31 tmp.sh
# 权限列最后一个字符为加号(+)代表该文件上存在若干条目的拓展 ACL 结构，但我没找到这样的文件。
```

> 使用 `chmod` 命令更改具有 ACL 的文件的组权限则不会更改组所有者权限，而是更改 ACL 掩码。如果目的是更新文件的组所有者权限，需使用 `setfacl -m g::perms <file>`。

​使用 `getfacl <file>` 查看文件的 ACL 详细信息。

```bash
[root@localhost ~]\# getfacl /run/log/journal/6651ee1da4824a37975cd5245c8cb076/system.journal 
getfacl: Removing leading '/' from absolute path names
# file: run/log/journal/6651ee1da4824a37975cd5245c8cb076/system.journal
# owner: root
# group: systemd-journal
user::rw-
group::r--
group:adm:r--
group:wheel:r--
mask::r--
other::---

```

```bash
[root@localhost ~]\# getfacl .
# file: .
# owner: root
# group: root
user::r-x
group::r-x
other::---

```

​一个进程能否访问文件，将按照以下规则应用文件权限和 ACL：

- 如果正在以文件所有者身份运行进程，则应用文件的用户 ACL 权限；
- 如果正在以指定用户 ACL 条目中列出的用户身份运行进程，则应指定用户 ACL 权限（只要掩码允许）；
- 如果正在以与文件的组所有者相匹配的组身份运行进程，或者以具有显示指定组 ACL 条目的组身份运行进程，则应用相匹配的 ACL 权限（只要掩码允许）；
- 否则，将应用文件的其他 ACL 权限。

> 当 systemd-journald 配置为使用持久存储时，系统管理员应在 /var/log/journal 文件夹上设置 ACL。

​*systemd-udevd* 监听内核发出的设备事件，并根据 udev 规则处理每个事件，该服务的功能相当于 Windows 上的设备管理器。它根据一组 udev 规则，来启用某些设备的 *uaccess* 标记，如 CD/DVD 播放器或刻录机、USB 存储设备、声卡等等。每个登陆 GUI 的用户将会刷新热插拔设备的 ACL。

```bash
[root@localhost ~]\# getfacl /dev/sr0
getfacl: Removing leading '/' from absolute path names
# file: dev/sr0
# owner: root
# group: cdrom
user::rw-
user:student:rw-
group::rw-
mask::rw-
other::---

```

## 更改 ACL 权限

​使用 `setfacl -m <acl> file` 或 `setfacl -M <file> file` 来修改文件的 acl 规则。（添加或修改、不会舍弃其他规则）。

```bash
[root@localhost ~]\# setfacl -m u:student:rwx tmp.sh
[root@localhost ~]\# getfacl tmp.sh
# file: tmp.sh
# owner: root
# group: root
user::rw-
user:student:rwx
group::r--
mask::rwx
other::r--

```

​使用 `setfacl --set <acl> file` 或 `setfacl --set-file <file> file` 选项来完全替代文件的 acl 规则。（应用新规则，舍弃所有旧规则）

​通过管道将 fileA 文件的 ACL 规则应用至 fileB：

```bash
[root@localhost ~]\# getfacl fileA | setfacl --set-file=- fileB	# - 可以指定使用 stdin 文件
```

​使用 *-R* 选项可以递归的修改目录中文件的 ACL。

```bash
[root@localhost ~]\# setfacl -R -m u:name:rx directory
```

​使用 `setfacl -x <acl> file` 来删除 acl 条目：

```bash
[root@localhost ~]\# setfacl -x u:name, g:name file
```

​使用 `setfacl -b file` 来删除文件的所有 ACL 条目：

```bash
[root@localhost ~]\# setfacl -b tmp.sh
[root@localhost ~]\# getfacl tmp.sh
# file: tmp.sh
# owner: root
# group: root
user::rw-
group::r--
other::r--

```

​为了确保在目录中创建的文件和目录继承特定的 ACL，须在目录上使用默认 ACL。

```bash
[root@localhost ~]\# setfacl -m d:u:name:rx directory
```

​使用 `setfacl -k directory` 命令删除目录的所有默认 ACL 条目。

```bash
[root@localhost ~]\# setfacl -k directory
```
