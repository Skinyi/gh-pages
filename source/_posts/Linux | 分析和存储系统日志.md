---
title: 分析和存储系统日志
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

## 系统日志概述

​进程和操作系统内核会以发生的具体事件记录日志，这些日志可用于系统审核和问题的故障排除。

​许多系统以文本文件的形式存储日志，它们一般都被保存在 /var/log 目录下。RHEL 内建了一个基于 Syslog 协议的标准日志记录系统。systemd-journald 和 rsyslog 服务处理红帽企业 linux 8 中的 syslog 消息。

- Systemd-journald 服务是操作系统事件日志架构的核心。它收集许多来源的事件消息，包括内核、引导过程早期阶段的输出、守护进程启动和运行时的标准输出及标准错误，以及 syslog 事件。然后，他会将它们重构为一种标准格式，并写进带有索引的结构化系统日中。默认情况下，该日志存储在系统重启后不保留的文件系统上。

- Rsyslog 服务对 syslog 消息进行排序，并将它们写入到在系统重启后不保留的日志文件中(/var/log)。rsyslog 服务会根据发送每条消息的程序类型或设备以及每条 syslog 消息的优先级，将日志消息排序到特定的日志文件。
  
以下是一些重要的日志文件：

| 日志文件                 | 描述                                                            |
| -------------------- | ------------------------------------------------------------- |
| /var/log/messages    | 包含全局系统消息，包括系统启动期间记录的消息。该文件存储邮件、作业调度、任务计划、守护程序、内核、授权、调试等模块的信息。 |
| /var/log/boot.log    | 包含与系统启动相关的非 syslog 控制台消息。                                     |
| /var/log/lastlog     | 显示最近所有用户的登录信息。只能通过 lastlog 命令来查看这个文件的内容。                      |
| /var/log/maillog     | 包含系统中运行的邮件服务器的日志信息。如 sendmail 日志包含所有已发送的邮件的信息。                |
| /var/log/btmp        | 包含失败的尝试登陆的信息。使用 *last* 命令来预览这个文件。如：`last -f /var/log/btmp     |
| /var/log/cups        | 所有和打印或打印相关的日志消息。                                              |
| /var/log/dnf.rpm.log | 记录和软件包安装的相关日志。                                                |
| /var/log/secure      | 包含认证与授权相关的信息。sshd 在此记录所有信息，包含失败的登陆。                           |
| /var/log/wtmp        | 所有登陆记录。使用 wtmp 可以指出谁登陆了系统，以及谁看了这个信息。                          |
| /var/log/audit       | 包含 Linux 授权守护进程（auditd）存储的信息。                                 |
| /var/log/samba       | 包含 samba 服务的信息。                                               |
| /var/log/sssd        | 系统安全服务守护程序用于管理远程目录和身份验证机制的访问。                                 |
| /var/log/cron        | 与调度作业执行相关的消息。                                                 |

> 🟢 有些应用不使用 syslog 管理他们的日志消息，但它们仍将其日志文件放在 /var/log 的某一子目录中。

## 查看系统日志文件

​Syslog，其进程名为 rsyslog，根据设备日志消息的严重程度将日志分了八个优先级。

| 代码  | 优先级     | 严重性      |
| --- | ------- | -------- |
| 0   | emerg   | 系统不可用    |
| 1   | alert   | 必须立即采取措施 |
| 2   | crit    | 临界情况     |
| 3   | err     | 非严重错误状况  |
| 4   | warning | 警告情况     |
| 5   | notice  | 正常但重要的事件 |
| 6   | info    | 信息性事件    |
| 7   | debug   | 调试级别消息   |

​Rsyslog 服务使用日志消息的设备和优先级来确定如何进行处理。其配置规则位于 /etc/rsyslog.conf 文件和 /etc/rsyslog.d 目录中拓展名为 .conf 的任何文件。通过在 /etc/rsyslog.d 目录中添加适当的文件，某应用的软件包可以轻松的添加属于自己的日志输出规则。

​每个控制着 syslog 消息排序方式的规则都对应了其中一个配置文件中的一行。如：

```bash
# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages
```

​其中，每行左侧表示与规则匹配的 syslog 消息的设备和严重性。每行右侧表示要将日志消息保存到的文件（或消息所要发送到的其他位置）。星号(\*)是一个匹配所有值的通配符。

> 🟢 使用 `logger` 命令可以手动往系统日志中添加信息，使用方法详见 man page: logger(1)。
> 
> ```bash
> [root@localhost ~]\# logger -p user.info "Test User Log" # -p 选项指定优先级
> [root@localhost ~]\# tail -n 1 -f /var/log/messages
> Aug 23 21:22:22 localhost root[3276]: Test User Log
> ```

## 查看系统日志条目

​Systemd-journal 服务将日志数据存储在带有索引的结构化二进制文件中，该文件称为日志。此数据包含与日志事件相关的额外信息。例如，对于 syslog 事件，这可包含原始消息的设备和优先级。

> 🟢 系统运行日志默认存储在 /run/log 目录下，该目录下的内容会在系统重启后清除。

​若要从系统运行日志中检索日志消息，可使用 journalctl 命令。可以使用此命令来查看日志中的所有消息，或根据各种选项和标准来搜索特定事件。如果以 root 身份运行该命令，则对日志具有完全访问权限。普通用户也可以使用此命令，但可能会被限制查看某些消息。

```bash
journalctl [OPTIONS...] [MATCHES] # 查询 systemd 日志
# 例子
root@localhost ~]\# journalctl    # 不带参数默认输出 system.journal 的所有内容
-- Logs begin at Mon 2021-08-23 21:08:56 PDT, end at Mon 2021-08-23 21:38:22 PDT. --
Aug 23 21:08:56 localhost.localdomain kernel: Linux version 4.18.0-240.el8.x86_64 (mockbuild@x86-vm-09.build.eng.bos.redhat.com) (gcc version 8.3.1 20191121>
Aug 23 21:08:56 localhost.localdomain kernel: Command line: BOOT_IMAGE=(hd0,msdos1)/vmlinuz-4.18.0-240.el8.x86_64 root=UUID=360e2f42-d6de-4209-8487-f36b7cc6>
Aug 23 21:08:56 localhost.localdomain kernel: Disabled fast string operations
......
```

​`journalctl` 命令突出显示重要的日志消息：优先级为 notice 或 warning 的消息显示为粗体文本，而优先级为 error 或以上的消息则显示为红色文本。

​成功利用日志进行故障排除和审核的关键在于，将日志搜索限制为仅显示相关的输出。

​默认情况下，`journalctl -n` 显示最后 10 个条目，可在参数后面指定具体显示的条数。

```bash
[root@localhost ~]\# journalctl -n 3
-- Logs begin at Mon 2021-08-23 21:08:56 PDT, end at Mon 2021-08-23 21:40:06 PDT. --
Aug 23 21:38:22 localhost.localdomain systemd-logind[1137]: New session 8 of user root.
Aug 23 21:38:22 localhost.localdomain sshd[3450]: pam_unix(sshd:session): session opened for user root by (uid=0)
Aug 23 21:40:06 localhost.localdomain chronyd[1070]: Source 2602:fcad:1::10 replaced with 124.108.20.1
```

​类似 `tail -f`，`journalctl -f` 也可以实现日志内容的实时更新：

```bash
[root@localhost ~]\# journalctl -f
-- Logs begin at Mon 2021-08-23 21:08:56 PDT. --
Aug 23 21:38:22 localhost.localdomain sshd[3441]: Accepted password for root from 172.16.1.195 port 6922 ssh2
Aug 23 21:38:22 localhost.localdomain systemd-logind[1137]: New session 7 of user root.
Aug 23 21:38:22 localhost.localdomain systemd[1]: Started Session 7 of user root.
Aug 23 21:38:22 localhost.localdomain sshd[3441]: pam_unix(sshd:session): session opened for user root by (uid=0)
Aug 23 21:38:22 localhost.localdomain sshd[3450]: Accepted password for root from 172.16.1.195 port 6925 ssh2
Aug 23 21:38:22 localhost.localdomain systemd[1]: Started Session 8 of user root.
Aug 23 21:38:22 localhost.localdomain systemd-logind[1137]: New session 8 of user root.
Aug 23 21:38:22 localhost.localdomain sshd[3450]: pam_unix(sshd:session): session opened for user root by (uid=0)
Aug 23 21:40:06 localhost.localdomain chronyd[1070]: Source 2602:fcad:1::10 replaced with 124.108.20.1
Aug 23 21:48:56 localhost.localdomain PackageKit[2105]: search-file transaction /131_bcadadca from uid 0 finished with success after 27ms
```

​使用 *-p* 选项可以根据日志的优先级过滤日志输出，其中参数可指定优先级的名称或编号，`journalctl -p` 命令会显示*该优先级及以上*的日志输出。

```bash
[root@localhost ~]\# journalctl -p err
-- Logs begin at Mon 2021-08-23 21:08:56 PDT, end at Mon 2021-08-23 21:48:56 PDT. --
Aug 23 21:09:14 localhost.localdomain kernel: piix4_smbus 0000:00:07.3: SMBus Host Controller not enabled!
```

​在查找具体事件时，可以将输出限制为某一特定的时间段。`--since` 和 `--until` 选项，它们可以将输出限制为特定的时间范围。其参数的具体格式限制为 *YYYY-MM-DDhh:mm:ss*，除了这两个选项，其还接受 *yesterday*、*today* 和 *tomorrow* 作为有效参数。

```bash
[root@localhost ~]\# journalctl --since "2021-08-23 22:00:00" --until "2021-08-23 22:05:00"
-- Logs begin at Mon 2021-08-23 21:08:56 PDT, end at Tue 2021-08-24 01:22:49 PDT. --
Aug 23 22:01:01 localhost.localdomain CROND[3760]: (root) CMD (run-parts /etc/cron.hourly)
Aug 23 22:01:01 localhost.localdomain run-parts[3763]: (/etc/cron.hourly) starting 0anacron
Aug 23 22:01:01 localhost.localdomain anacron[3769]: Anacron started on 2021-08-23
Aug 23 22:01:01 localhost.localdomain run-parts[3771]: (/etc/cron.hourly) finished 0anacron
Aug 23 22:01:01 localhost.localdomain anacron[3769]: Normal exit (0 jobs run)
```

​也可以指定相对于当前的某个时间以后的所有条目。

```bash
[root@localhost ~]# journalctl --since "-1.1 hour"
-- Logs begin at Mon 2021-08-23 21:08:56 PDT, end at Tue 2021-08-24 01:22:49 PDT. --
Aug 24 01:01:01 localhost.localdomain CROND[5331]: (root) CMD (run-parts /etc/cron.hourly)
Aug 24 01:01:01 localhost.localdomain run-parts[5334]: (/etc/cron.hourly) starting 0anacron
Aug 24 01:01:01 localhost.localdomain run-parts[5342]: (/etc/cron.hourly) finished 0anacron
Aug 24 01:01:01 localhost.localdomain anacron[5340]: Anacron started on 2021-08-24
Aug 24 01:01:01 localhost.localdomain anacron[5340]: Normal exit (0 jobs run)
```

> 🟢 --since 和 --until 支持的更复杂的时间格式可参见 systemd.time(7) man page。

​可以使用 `journalctl -o` 命令以特定的模式（verbose/export/json 等）输出。

```bash
[root@localhost ~]\# journalctl -o verbose
-- Logs begin at Mon 2021-08-23 21:08:56 PDT, end at Tue 2021-08-24 >
Mon 2021-08-23 21:08:56.476108 PDT [s=ac8c7a315ec04e1c98b39f3eba4fe1>
    _SOURCE_MONOTONIC_TIMESTAMP=0
    _TRANSPORT=kernel
    PRIORITY=5
```

​一些特定的系统日志筛选常用字段，可用于搜索与特定进程或事件相关的行。

| 筛选选项            | 作用                  |
| --------------- | ------------------- |
| \_COMM          | 指定命令的名称             |
| \_EXE           | 指定进程的可执行文件路径        |
| \_PID           | 指定进程的 PID           |
| \_UID           | 指定运行该进程的用户的 UID     |
| \_SYSTEMD\_UNIT | 指定启动该进程的 systemd 单元 |

​也可以组合多个系统日志字段以实现更加精细的搜索查询，例如：

```bash
[root@localhost ~]\# journalctl -n 3 _SYSTEMD_UNIT=sshd.service _UID=0 # 与操作
-- Logs begin at Mon 2021-08-23 21:08:56 PDT, end at Tue 2021-08-24 02:03:19 PDT. --
Aug 23 21:38:22 localhost.localdomain sshd[3441]: pam_unix(sshd:session): session opened for user root by (uid=0)
Aug 23 21:38:22 localhost.localdomain sshd[3450]: Accepted password for root from 172.16.1.195 port 6925 ssh2
Aug 23 21:38:22 localhost.localdomain sshd[3450]: pam_unix(sshd:session): session opened for user root by (uid=0)
[root@localhost ~]\# journalctl _COMM=sshd + _PID=1247 # 或操作
-- Logs begin at Mon 2021-08-23 21:08:56 PDT, end at Tue 2021-08-24 02:03:19 PDT. --
Aug 23 21:09:31 localhost.localdomain sshd[1251]: Server listening on 0.0.0.0 port 22.
Aug 23 21:09:31 localhost.localdomain sshd[1251]: Server listening on :: port 22.
Aug 23 21:11:38 localhost.localdomain cupsd[1247]: REQUEST localhost - - "POST / HTTP/1.1" 200 363 Create-Printer-Subscriptions successful-ok
```

## 保留系统日志

​默认情况下，系统日志保存在 /run/log/journal 目录中，这意味着系统重启时这些日志会被清除。可以在 /etc/systemd/journald.conf 文件中更改 systemd-journald 服务的配置设置，使日志在系统重启后保留下来。

​/etc/systemd/journald.conf 文件中的 <b>Storage</b> 参数决定系统日志以易失性方式存储，还是在系统重启后持久保留，该参数的可选值有以下几种：

| 值          | 日志存储位置                                       | 说明                                                     |
| ---------- | -------------------------------------------- | ------------------------------------------------------ |
| persistent | /var/log/journal | 持久保留，若该目录不存在，systemd-journald 服务会自动创建。                 |
| volatile   | /run/log/journal                             | 存放在内存文件系统，/run 目录下的内容重启后丢失。                            |
| auto（默认）   | 以上两者任一                                       | 默认 /run/log/journal，但若 /var/log/journal 目录存在，则存放在该目录下。 |

> 1. 持久系统日志的优点是系统启动后就可立即利用历史数据。
> 2. 即便是持久日志，并非所有数据都将永久保留，此日志项具有一个内置的日志轮转机制会每月触发。
> 3. 修改日志存储配置文件需要重新载入该服务，相关内容可参见 *13. 控制服务和守护进程* 章节。 

​默认情况下，日志的大小不能超过所处文件系统的 10%，也不能造成文件系统的可用空间低于 15%，可以在 */etc/systemd/journald.conf* 中为运行时和持久日志调整这些值。当 systemd-journald 进程启动时，会记录当前的日志大小限额。

​以下命令输出显示了反映当前大小限额的日志条目：

```bash
[root@localhost ~]\# journalctl | grep -E 'Runtime|System journal'
Aug 24 17:47:16 localhost.localdomain systemd-journald[304]: Runtime journal (/run/log/journal/6651ee1da4824a37975cd5245c8cb076) is 8.0M, max 185.4M, 177.4M free.
Aug 24 17:47:29 localhost.localdomain systemd-journald[694]: Runtime journal (/run/log/journal/6651ee1da4824a37975cd5245c8cb076) is 8.0M, max 185.4M, 177.4M free.
Aug 24 17:47:29 localhost.localdomain systemd-journald[694]: Runtime journal (/run/log/journal/6651ee1da4824a37975cd5245c8cb076) is 8.0M, max 185.4M, 177.4M free.
Aug 24 17:47:36 localhost.localdomain systemd[1]: Starting Tell Plymouth To Write Out Runtime Data...
Aug 24 17:47:36 localhost.localdomain systemd[1]: Started Tell Plymouth To Write Out Runtime Data.
```

​使用持久存储的 systemd-journald 服务会存储系统一次或多次启动后的日志，若要根据系统的每次启动筛选日志则可以使用选项 `-b`：

```bash
[root@localhost ~]\# journalctl -b 1    # 检索日志存储的第一次启动的条目
[root@localhost ~]\# journalctl -b      # 检索当前系统启动启动后的条目
[root@localhost ~]\# journalctl -b -1   # 过去一次的系统启动时的tiao
```
