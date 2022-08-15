---
title: 管理临时文件
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

​现代操作文件会在运行时产生大量的临时文件和目录，有些应用（和用户）会使用 */tmp* 目录来保存临时数据，还有一些应用（和用户）则使用更特定于任务的位置，如守护进程以及 */run* 下特定于用户的易失性（存储于内存，断电或重启丢失）目录。

​为了保持系统充分运行，有必要创建那些不存在的目录和文件，因为守护进程和脚本可能会依靠它们的存在，而清除旧文件后就不会填满磁盘空间或提供错误信息。

​RHEL 7 以上引入了一个名为 *systemd-tmpfiles* 的新工具，它提供了一种结构化和可配置的方法来管理临时目录和文件。在 systemd 启动系统后，其中一个最先启动的服务单元是 systemd-tmpfiles-setup。该服务运行命令 `systemd-tmpfiles --create --remove`。此命令从 */usr/lib/tmpfiles.d/\*.conf*、*/run/tmpfiles.d/\*.conf* 和 */etc/tmpfiles.d/\*.conf* 读取配置文件。系统会删除这些配置文件中标记的要删除的任何文件和目录，并且会创建标记要创建（或修复权限）的任何文件和目录，并使其拥有正确的权限（如有必要）。

## 使用定时器清理临时文件

​RHEL 内建了一个名为 systemd-tmpfiles-clean.timer 的 systemd 定时器单元定时触发 systemd-tmpfiles-clean.service 来执行 systemd-tmpfiles --clean 命令。

​定时器 systemd-tmpfiles-clean.timer 文件的内容：

```ini
#  SPDX-License-Identifier: LGPL-2.1+
# 
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Daily Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)

[Timer]
# 表示 systemd-tmpfiles-clean.service 将在系统启动 15min 后被触发
OnBootSec=15min
# 表示在上一次激活服务单元 1d 后再次触发 systemd-tmpfiles-clean.service
OnUnitActiveSec=1d
```

​服务单元 systemd-tmpfiles-clean.service 文件的内容：

```ini
#
#  This file is part of systemd.
#
#  systemd is free software; you can redistribute it and/or modify it
#  under the terms of the GNU Lesser General Public License as published by
#  the Free Software Foundation; either version 2.1 of the License, or
#  (at your option) any later version.

[Unit]
Description=Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
DefaultDependencies=no
Conflicts=shutdown.target
After=local-fs.target time-sync.target
Before=shutdown.target

[Service]
Type=oneshot
ExecStart=/usr/bin/systemd-tmpfiles --clean
SuccessExitStatus=65
IOSchedulingClass=idle
```

​可以在 */etc/systemd/system* 里更改 systemd-tmpfiles-clean.timer 的内容来定制自己的临时文件清理频率。

## 手动清理临时文件

​命令 *systemd-tmpfiles --clean* 解析的配置文件与 *systemd-tmpfiles --create* 命令相同，但前者不会创建文件和目录，而是会清除在比配置文件中定义的最长期限更近的时间尚未访问、更改或修改的所有文件。

​*systemd-tmpfiles*到的配置文件的基本语法由七列构成：”类型“、”路径“、”模式“、”UID“、”GID“、”期限“ 和 ”参数“。类型指的是 *systemd-tmpfiles* 应执行的操作；例如 *d* 表示创建还不存在的目录，或者 *Z* 表示以递归方式恢复 *SELinux* 上下文以及文件权限和所有权。示例：

```bash
d /run/systemd/seats 0755 root root -
# 在创建文件和目录时，如果目录 /run/systemd/seats 还不存在，则创建该目录，所有者为用户 root 和组 root ，权限设置为 rwxr-x-r-x。系统不会自动清除该目录。
D /home/student 0700 student student 1d
# 如果目录 /home/student 还不存在，则创建该目录。如果存在，则清空其所有内容。运行 systemd-tmpfiles --clean 时，删除在超过一天时间内尚未被访问、更改或删除的所有文件。
L /run/fstablink - root root - /etc/fstab
# 创建指向 /etc/fstab 的符号链接 /run/fstab。绝对不要自动清除这一行。
```

### 配置文件优先级

​配置文件可位于三个位置：

- */etc/tmpfiles.d/\*.conf*
- */run/tmpfiles.d/\*.conf*
- */usr/lib/tmpfiles.d/\*.conf*

​*/usr/lib/tmpfiles.d* 中的文件是由相关 RPM 软件包提供的，不应编辑这些文件。*/run/tmpfiles.d/* 下的文件本身是易失性文件，通常由守护进程用来管理自己的运行时临时文件。*/etc/tmpfiles.d/* 下的文件旨在供管理员配置自定义临时位置，以及覆盖供应商提供的默认值。

​同名文件生效优先级：*/etc/tmpfiles.d/* > */run/tmpfiles.d/* > */usr/lib/tmpfiles.d/*。
