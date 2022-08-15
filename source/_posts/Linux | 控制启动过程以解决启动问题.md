---
title: 控制启动过程以解决启动问题
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

## RHEL8 启动过程描述

​现代计算机系统是硬件与软件的复杂组合。从未定义的断电状态开始，到拥有登录提示符的运行中系统，这需要大量的硬件和软件配合工作。以下列表从较高层面概述了启动红帽企业 Linux 8 的物理 *X86_64* 系统所涉及的任务。X86_64虚拟机列表大致相同，但某些特定于硬件的步骤是由虚拟机监控程序在软件中处理的。

- 计算机已接通电源。系统固件（现代 UEFI 或更旧的 BIOS）运行开机自检（POST，并开始初始化部分硬件。

  使用系统 BIOS 或 UEFI 配置屏幕【通常在启动过程早期通过按特定的组合键（例如F2）即可进入】进行配置。

- 系统固件会搜索可启动设备，可能是在 UEFI 启动固件中配置的，也可能按照 BIOS 中配置的顺序搜索所有磁盘上的主启动记录（MBR）。
  
  使用系统 BIOS 或 UEFI 配置屏幕【通常在启动过程早期通过按特定的组合键（例如 F2 ）即可进入】进行配置。
  
- 系统固件会从磁盘读取启动加载器，然后将系统控制权交给启动加载器。在红帽企业 Linux 8 系统中，启动加载器为 <b>GRand Unified Boot loader version 2</b>（GRUB2）。
  
  使用 `grub2-install` 命令进行配置，它将安装 *GRUB2* 作为磁盘上的启动加载器。
  
- *GRUB2* 将从 */boot/grub2/grub.cfg* 文件加载配置并显示一个菜单，从中可以选择要启动的内核。
  
  使用 */etc/grub.d/* 目录、*/etc/default/grub* 文件和 `grub2-mkconfig` 命令进行配置，以生成*/boot/grub2/grub.cfg* 文件。
  
- 选择内核或超时到期后，启动加载器会从磁盘中加载内核和 *initramfs* ，并将它们放入内存中。*initramfs*是一个存档，其中包含启动时所有必要硬件的内核模块、初始化脚本等等。在红帽企业 Linux8 中，**initramfs* 包含自身可用的整个系统。

  使用 */etc/dracut.conf.d/* 目录、`dracut` 命令和 `lsinitrd` 命令进行配置，以检查*initramfs* 文件。

- 启动加载器将控制权交给内核，从而传递启动加载器的内核命令行中指定的任何选项，以及 initramfs 在内存中的位置。
  
  使用 */etc/grub.d/* 目录、*/etc/default/grub* 文件和 `grub2-mkconfig` 命令进行配置，以生成 */boot/grubz/grub.cfg* 文件。

- 对于内核可在 initramfs 中找到驱动程序的所有硬件，内核会初始化这些硬件，然后作为 PID 1 从 initramfs 执行 */sbin/init*。在红帽企业 Linux8 中，*/sbin/init* 是一个指向 systemd 的链接。

  使用内核 *init=* 命令行参数进行配置。

- initramfs 中的 systemd 实例会执行 *initrd.target* 目标的所有单元。这包括将磁盘上的 root 文件系统挂载于 */sysroot* 目录。

  使用 */etc/fstab* 进行配置。

- 内核将 root 文件系统从 initramfs 切换（回转）为 */sysroot* 中的 root 文件系统。随后，systemd 会使用磁盘中安装的 systemd 副本来自行重新执行。

- systemd 会查找从内核命令行传递或系统中配置的默认目标，然后启动（或停止）单元，以符合该目标的配置，从而自动解决单元间的依赖关系。本质上，*systemd* 目标是一组系统应激活以达到所需状态的单元。这些目标通常启动一个基于文本的登录或图形登录屏幕。

使用 */etc/systemd/system/default.target* 和 */etc/systemd/system/* 进行配置。

## 重新启动和关闭

​要关闭或从命令行重新启动正在运行的系统，可以使用 `systemctl` 命令。

​`systemctl poweroff` 会停止所有运行的服务，卸载所有文件系统（或者在文件系统无法卸载时以只读形式将其重新挂载），然后关闭系统。

​`systemctl reboot` 会停止所有运行的服务，卸载所有文件系统，然后重新启动系统。您也可以使用这些命令的简短版本`poweroff` 和 `reboot`，它们是 `systemctl` 同等命令的符号链接。

> `systemctl halt` 和 `halt` 也可以用于停止系统，但与 `poweroff` 不同，这些命令不会关闭系统，而是让系统进入能安全地手动关闭的状态。

## SYSTEMD Target

​systemd target 是一组系统应启动以达到所需状态的 systemd 单元。下表列出了最重要的目标：

| Target            | 用途                                                         |
| ----------------- | ------------------------------------------------------------ |
| graphical.target  | 系统支持多用户、图形和基于文本的登录。                       |
| multi-user.target | 系统仅支持多用户、基于文本的登录。                           |
| rescue.target     | sulogin 提示，表示基本系统初始化已完成。                     |
| emergency.target  | sulogin 提示，表示 initramfs 回转完成，且系统 root 以只读形式挂载于 */* 上。 |

​某个目标可能属于另一目标。例如，graphical.target 包含 multi-user.target，后者反过来取决于 basic.target 和其他目标。可以使用以下命令来查看这些依赖项。

```bash
[user@localhost ~]\$ systemctl list-dependencies graphical.target | grep target
graphical.target
* ﹂multi-user.target
*	|-basic.target
*	|	|-paths.target
*	|	|-slices.target
*	|	|-sockets.target
*	|	|-sysinit.target
*	|	|	|-cryptsetup.target
*	|	|	|-local-fs.target
*	|	|	|-swap.target
......
```

​要列出可用目标，可以使用以下命令：

```bash
[user@localhost ~]\$ systemctl list-units --type=target --all
  UNIT					LOAD	ACTIVE		SUB		DESCRIPTION
  ------------------------------------------------------------------------
  basic.target			loaded	active		active	Basic System
  cryptsetup.target 	loaded	active		active 	Local Encrypted Volumes
  emergency.target		loaded	inactive	dead	Emergency Mode
  getty-pre.target		loaded	inactive	dead	Login Prompts (Pre)
  getty.target			loaded	active		active	Login Prompts
  graphical.target		loaded	inactive	dead	Graphical Interface
  ......
```

### 运行时切换 Target

在运行的系统中，系统管理员可以使用 `systemctl isolate` 命令本来切换到其他目标。

```bash
[root@localhost ~]\# systemctl isolate multi-user.target
```

​隔离某个目标会停止该目标（及其依赖项）不需要的所有服务，并启动任何尚未启动的所需服务。但并非所有目标都能隔离，一般只能隔离单元文件中设置了 *A11owIsolate=yes* 的目标。例如，可以隔离图形目标，但不能隔离 *cryptsetup* 目标。

```bash
[user@localhost ~]\$ systemctl cat graphical.target
# /usr/lib/systemd/system/graphical.target
......
[Unit]
Description=Graphical Interface
Documentation=man:systemd.special (7）
Requires=multi-user.target
Wants=display-manager.service
Conflicts=rescue.service rescue.target
After=multi-user.target rescue.service rescue.target display-manager.service
AllowIsolate=yes
```

```bash
[user@localhost ~]\$ systemctl cat cryptsetup.target
#/usr/lib/systemd/system/cryptsetup.target
......
[unit]
Description=Local Encrypted Volumes
Documentation=man：systemd，special (7)
```

### 设置默认目标

系统启动时，*systemd* 会激活 *default.target* 目标。通常，*/etc/systemd/system/* 中的默认目标是指向 *graphical.target* 或 *multi-user.target* 的符号链接。`systemctl` 命令提供了两个子命令（`get-default` 和 `set-default`）用于管理该符号链接，而不是手动编辑该链接。

```bash
[root@localhost ~]\# systemctl get-default
multi-user.target
[root@localhost ~]\# systemctl set-default graphical.target
Removed /etc/systemd/system/default.target.
Created symlink /etc/systemd/system/default.target -> /usr/lib/systemd/system/graphical.target
[root@localhost ~]\# systemctl get-default
graphical.target
```

### 在启动时选择其他目标

​要在启动时选择其他目标，请从启动加载器将 *systemd.unit=target.target* 选项附加到内核命令行。

​例如，要将系统启动到救援 shell，以便能在几乎没有任何服务运行的情况下更改系统配置，可以在启动加载器将以下选项附加到内核命令行。

```ini
systemd.unit=rescue.target
```

​该配置更改仅影响单个启动，这使其成为对启动过程进行故障排除的有用工具。

​要使用这种选择其他目标的方法，可以执行以下步骤：

1. 启动或重新启动系统；
2. 按任意键（**Enter** 除外，它用于执行正常启动）中断启动加载器菜单倒计时；
3. 将光标移至要启动的内核条目；
4. 按 **e** 编辑当前条目；

5. 将光标移至以 <b>linux</b> 开头的行 —— 此为内核命令行；
6. 附加 `systemd.unit=target.target`，例如，`systemd.unit=emergency.target`；
7. 按 **Ctrl** + **x** 使用这些更改进行启动；

## 重置 ROOT 密码

​每个系统管理员都应该能完成的一项任务是重置丢失的 root 密码。如果管理员仍处于登录状态，不管是作为拥有完全 *sudo* 访问权限的非特权用户，还是作为 root 用户，这个任务都很简单。如果管理员未登录，则这个任务变得略微复杂。

​有几种方法可用于设置新的root密码。例如，系统管理员可以使用 Live CD 启动系统，从此处挂载根文件系统，然后编辑 */etc/shadow*。在本节中，我们将探讨一个无需使用外部介质的方法。

### 从启动加载器重置 ROOT 密码

​在 RHEL8 中，可以使从 *initramfs* 运行的脚本在某些点暂停，提供 root shell，然后在该 shell 存在的情况下继续。这主要是为了进行调试，但也可以使用该方法来重置丢失的 root 密码。

​要访问该 root shell，可以按照以下步骤进行操作：

1. 重新启动系统。

2. 按任意键（**Enter**除外）中断启动加载器倒计时。

3. 将光标移至要启动的内核条目。

4. 按 **e** 编辑选定的条目。

5. 将光标移到内核命令行（以linux开头的行）。

6. 附加 ``rd.break``。利用该选项，就在系统从 *initramfs* 向实际系统移交控制权前，系统将会中断。

7. 按 **Ctrl** + **x** 使用这些更改进行启动。

​此时，系统会显示 root shell，且磁盘上的实际根文件系统会在 /sysroot 中以只读方式挂载。由于进行故障排除经常要求修改根文件系统，因此需要将根文件系统更改为<b>读/写</b>模式。以下步骤说明在对 `mount` 命令使用 `remount,rw` 选项的情况下，如何利用所设置的新选项（*rw*）重新挂载文件系统。

> 预建的映像可能会在内核中放置多个 *console=* 参数，以便支持各种各样的实施场景。这些 *console=* 参数指示了用于控制台输出的设备。对于 *rd.break*，有一点需要注意的是，尽管系统会将内核消息发送给所有控制台，但提示符最终将使用最后提供的控制台。如果未获得提示符，在从启动加载器编辑内核命令行时，可能需要临时对 *console=* 参数重新进行排序。

>系统尚未启用 SELinux，因此您所创建的任何文件都没有 SELinux 上下文。有些工具（例如 `passwd` 命令）首先会创建一个临时文件，然后移动新文件以代替要编辑的文件，从而有效地创建不带 SELinux 上下文的新文件。因此，当对 `passwd` 命令使用 *rd.break* 时，*/etc/shadow* 文件并没有获得 SELinux 上下文。

​要在此时重置 root 密码，可以执行以下步骤：

1. 以 <b>读/写</b> 形式重新挂载 */sysroot*；

```bash
switch_root:/\# mount -o remount,rw /sysroot
```

2. 切换为 `chroot` 存放位置，其中 */sysroot* 被视为文件系统树的根；

```bash
switch_root:/\# chroot /sysroot
```

3. 设置新 root 密码；

```bash
sh-4.4\# passwd root
```

4. 确保所有未标记的文件（包括此时的 */etc/shadow*）在启动过程中都会重新获得标记；

```bash 
sh-4.4\# touch /.autorelabel
```

5. 键入 `exit` 命令并执行两次，分别退出 `chroot` 和 *initramfs* shell。

​此时，系统将继续进行启动，执行完整的SELinux重新标记，然后再次重新启动。

### 检查日志来排除启动问题

​查看以前启动失败时的日志会很有用。如果系统日志在重启后持久保留，则可以使用 `journalctl` 工具来检查这些日志。

​需注意，默认情况下，系统日志保存在 */run/1og/journal* 目录中，这意味着系统重启时这些日志会被清除。要将日志存储在 */var/log/journal* 目录中（可在系统重启后持久保留），请在 */etc/systemd/journald.conf* 中将 <b>Storage</b> 参数设置为 <b>persistent</b>。

```bash
[root@localhost ~]\# vim /etc/systemd/journald.conf
......
[Journal]
Storage=persistent
......
[root@localhost ~]\# systemctl restart systemd-journal.service 
```

​要检查上一次启动的日志，请对 `journalctl` 使用 *-b* 选项。如果不使用任何参数，*-b* 选项将仅显示从上一次启动以来的消息。如果参数为负数，则显示以前的启动的日志。

```bash
[root@localhost ~]\# journalctl -b -1 -p err
```

​该命令将显示上一次启动中被评为错误或更严重级别的所有消息。

### 修复 SYSTEMD 启动问题

​为了在启动时对服务启动问题进行故障排除，红帽企业 Linux 8 提供了以下工具。

#### 启用早期调试 Shell

​通过执行 `systemctl enable debug-shell.service` 来启用 *debug-shell* 服务，系统会于启动序列早期在 *TTY9*（使用**ctrl** + **Alt** + **F9** 切换）上生成一个 root shell。该 shell 会自动作为 root 登录，这样，管理员可以在操作系统仍在启动时对系统进行调试。

> 在完成调试后务必要禁用 *debug-shell* 服务。

#### 使用紧急情况和救援目标

​通过从启动加载器将 *systemd.unit=rescue.target* 或 *systemd，unit=emergency.target* 附加到内核命令行，系统将生成救援或紧急情况 shell，而不是正常启动。这两个 shell 都需要提供 root 密码。

​紧急情况目标使 root 文件系统以只读方式挂载，而救援目标会等待 sysinit.target 完成，这样系统的更多部分会进行初始化，如日志记录服务或文件系统。此时 root 用户无法更改 */etc/fstab*，直至驱动器以读写状态重新挂载（`mount -o remount,rw /`）。

​管理员可以使用这些 shell 来修复坊碍系统正常启动的任何问题；例如，服务之间的依赖关系循环，或 */etc/fstab* 中的错误条目。从这些 shell 退出后，系统会继续进行常规启动过程。

#### 识别阻塞作业

​在启动过程中，`systemd` 会生成大量作业。如果其中其些作业无法完成，则它们会纺碍其他作业运行。要检查当前作业列表，管理员可以使用 `systemctl 1ist-jobs` 命令。所有列为 <b>running</b> 的作业都必须先完成，然后列为 <b>waiting</b> 的作业才可以继续。

## 修复启动时出现的文件系统问题

​*/etc/fstab* 中的错误和损坏的文件系统可能会阻止系统启动。在大多数情况下，`systemd` 会降
至需要提供 root 密码的紧急修复 shell。

​下表列出了一些常见错误及其结果。

| 问题                                 | 结果                                                         |
| ------------------------------------ | ------------------------------------------------------------ |
| 损坏文件系统                         | systemd 尝试修复文件系统。如果问题过于严重，无法进行自动修复，系统会将用户降至紧急 shell。 |
| */etc/fstab* 中引用的设备UUID 不存在 | systemd 将等待一定的时间，等设备变得可用。如果设备未变得可用，则系统会在超时后将用户降至紧急 shell。 |
| */etc/fstab* 中的挂载点不存在        | 系统将用户降至紧急 shell。                                   |
| */etc/fstab* 中指定的挂载点错误      | 系统将用户降至紧急 shell。                                    |

​在所有情况下，管理员还都可以使用 *emergency.target* 来诊断和修复问题，因为在显示紧急 shell 之前，不会挂载任何文件系统。

> 使用紧急 shell 解决文件系统问题时，别忘了在编辑 */etc/fstab* 之后运行 `systemctl daemon-reload`。如果不重新加载，*systemd* 可能会继续使用旧版本。

​在 */etc/fstab* 文件中，条目内的 <b>nofail</b> 选项允许在该文件系统挂载不成功的情况下仍启动系统。正常情况下，请勿使用该选项。使用 <b>nofail</b> 时，应用可以从缺失的存储启动，这可能会带来严重后果。
