---
title: 控制服务和守护进程
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

## SYSTEMCTL命令摘要

| 任务                       | 命令                               |
| ------------------------ | -------------------------------- |
| 查看有关单元状态的详细信息            | systemctl status UNIT            |
| 在运行中的系统上停止一项服务           | systemctl stop UNIT              |
| 在运行中的系统上启动一项服务           | systemctl start UNIT             |
| 在运行中的系统上重新启动一项服务         | systemctl restart UNIT           |
| 重新加载运行中服务的配置文件           | systemctl reload UNIT            |
| 彻底禁用服务，使其无法手动启动或在系统引导时启动 | systemctl mask UNIT              |
| 使屏蔽的服务变为可用               | systemctl unmask UNIT            |
| 将服务配置为在系统引导时启动           | systemctl enable UNIT            |
| 禁止服务在系统引导时启动             | systemctl disable UNIT           |
| 列出指定单元依赖的单元              | systemctl list-dependencies UNIT |

## 认识 SYSTEMD 服务

​Systemd 服务可以管理系统守护进程和网络服务，包括 Linux 的启动，一般包括服务启动和服务管理。它可在系统引导时以及运行中的系统上激活系统资源、服务器守护进程和其他进程。

​守护进程是在执行各种任务的后台等待或运行的进程。一般情况下，守护进程在系统引导时自动启动并持续运行至关机或手动停止。按照惯例，许多守护进程的名称以字母 d 结尾。

​systemd 意义上的服务通常指的是一个或多个守护进程，但启动或停止一项服务可能会对系统的状态进行一次性更改，不会留下守护进程之后继续运行（称为 oneshot）。

​在 RHEL 中，第一个启动的进程（PID 1）是 systemd。 如下是 systemd 提供的几项功能：

- 并行化功能（同时启动多个服务），它可提高系统的启动速度。

- 按需启动守护进程，而不需要单独的服务。

- 自动服务依赖关系管理，可以防止长时间超时。例如，只有在网络可用时，依赖网络的服务才会尝试启动。

- 利用 Linux 控制组一起追踪相关进程的方式。

## SYSTEMD 服务单元

​Systemd 使用单元来管理不同类型的对象。

| 单元类型          | 拓展名        | 描述                                                                                                  |
|:-------------:|:----------:|---------------------------------------------------------------------------------------------------|
| 系统服务          | .service   | 用于启动、停止、重启和重载经常访问的守护进程，如 web 服务器。                                                                   |
| 进程间通信（IPC）套接字 | .socket    | 由 Systemd 应监控的进程间通信（IPC）套接字。如果客户端连接套接字，systemd 将启动一个守护进程并将连接传递给它。套接字单元用于延迟系统启动时的服务启动，或者按需启动不常使用的服务。 |
| 路径单元          | .path      | 用于将服务的激活推迟到特定文件系统更改发生之后。这通常用于使用假脱机目录的服务，如打印系统。                                                      |
| Target        | .target    | 定义 target 信息(类似之前的 run-level)及依赖关系，一般仅包含 Unit 段                                                     |
| 设备单元          | .device    | 对于 */dev* 目录下的硬件设备，主要用于定义设备之间的依赖关系                                                                  |
| 挂载单元          | .mount     | 定义文件系统的挂载点，可以替代过去的 */etc/fstab* 配置文件                                                                |
| 自动挂载单元        | .automount | 用于控制自动挂载文件系统，相当于 SysV-init 的 autofs 服务                                                              |
| Scope         | .scope     | 非用户创建，由 Systemd 运行时产生的，描述一些系统服务的分组信息                                                                |
| Slice         | .slice     | 用于表示一个 CGroup 的树                                                                                    |
| Snapshot      | .snapshot  | 用于表示一个由 systemctl snapshot 命令创建的 Systemd Units 运行状态快照，可以切回某个快照                                      |
| 交换分区单元        | .swap      | 定义一个用户做虚拟内存的交换分区                                                                                    |
| 定时器单元         | .timer     | 用于配置在特定时间出发的任务，替代了 Crontab 的功能                                                                      |

> 若要修改某单元文件请勿在 */usr/lib/systemd/system* 目录里直接修改，而应在 */etc/systemd/system* 目录下创建该单元文件的一个副本，然后修改该副本文件。Systemd 会优先使用 */etc/systemd/system* 目录下的单元文件。 

### 查看所有已加载的服务单元

​可以使用以下命令来列出 systemd 加载的所有服务单元：

```bash
[root@localhost ~]\# systemctl list-units --type=service # 指定不同的 type 可以列出不同的单元类型
UNIT                               LOAD   ACTIVE SUB     DESCRIPTION              
abrt-journal-core.service          loaded active running Creates ABRT problems from coredumpctl messages
abrt-oops.service                  loaded active running ABRT kernel log watcher
......
```

​其中，该命令输出的各列的意义是：

| 列名          | 含义                                                 |
| ----------- | -------------------------------------------------- |
| UNIT        | 单元名称                                               |
| LOAD        | systemd 是否正确解析了单元的配置并将该单元加载到内存中                    |
| ACTIVE      | 单元的高级别激活状态。此信息表明单元是否已成功启动                          |
| SUB         | 单元的低级别激活状态。此信息指示有关该单元的更多详细信息。信息视单元类型、状态以及单元的执行方式而异 |
| DESCRIPTION | 单元的简短描述                                            |

​默认情况下，``systemctl list-units --type=service` 命令仅列出激活状态为 *active* 的服务单元。指定 *--all* 选项可列出所有服务单元，不论激活状态如何：

```bash
[root@localhost ~]\# systemctl list-units --type=service --all
  UNIT                                   LOAD      ACTIVE   SUB     DESCRIPTION                                                                  
  abrt-ccpp.service                      loaded    inactive dead    Install ABRT coredump hook                                                   
  abrt-journal-core.service              loaded    active   running Creates ABRT problems from coredumpctl messages
```

​使用 *--state=* 选项可按照 LOAD、ACTIVE 或 SUB 字段中的值进行筛选。

```bash
[root@localhost ~]\# systemctl list-units --type=service --state=dead | head -n 3
  UNIT                                   LOAD      ACTIVE   SUB  DESCRIPTION                                                     
  abrt-ccpp.service                      loaded    inactive dead Install ABRT coredump hook                                      
  abrt-vmcore.service                    loaded    inactive dead Harvest vmcores for ABRT
```

​不带任何参数运行 systemctl 命令可以列出已加载和活动的单元。

```bash
[root@localhost ~]\# systemctl
UNIT                                                                                                     LOAD   ACTIVE SUB       DESCRIPTION                                               
proc-sys-fs-binfmt_misc.automount                                                                        loaded active waiting   Arbitrary Executable File Formats File System Automount Po>
sys-devices-pci0000:00-0000:00:11.0-0000:02:00.0-usb2-2\x2d2-2\x2d2.1-2\x2d2.1:1.0-bluetooth-hci0.device loaded active plugged   /sys/devices/pci0000:00/0000:00:11.0/0000:02:00.0/usb2/2-2>
......
-.mount                                                                                                  loaded active mounted   Root Mount                                                
boot.mount                                                                                               loaded active mounted   /boot

......
cups.path                                                                                                loaded active running   CUPS Scheduler                                            
systemd-ask-password-plymouth.path                                                                       loaded active waiting   Forward Password Requests to Plymouth Directory Watch 
......
init.scope                                                                                               loaded active running   System and Service Manager                                
session-2.scope                                                                                          loaded active running   Session 2 of user student 
......
...
```

### 查看所有已安装的单元文件的状态

​*systemctl list-units* 命令显示 *systemd* 服务尝试解析并加载到内存中的单元；它不显示已安装但未启用的单元。可以使用以下命令来查看所有已安装的单元文件的状态：

```bash
[root@localhost ~]\# systemctl list-unit-files
UNIT FILE                                                                 STATE    
proc-sys-fs-binfmt_misc.automount                                         static   
-.mount                                                                   generated
```

​*--type=* 以及 *state=* 选项同样适用于此命令。其中 STATE 的可选值为 [enabled | disabled | static | masked]。

### 3. 查看服务状态

​使用 `systemctl status name.type` 命令来查看特定单元的状态，如果未提供单元类型，则 systemctl 将显示服务单元的状态（如果存在）。

```bash
[root@localhost ~]\# systemctl status cups
● cups.service - CUPS Scheduler
   Loaded: loaded (/usr/lib/systemd/system/cups.service; enabled; vendor preset: e>
   Active: active (running) since Mon 2021-08-09 17:08:41 PDT; 1h 11min ago
     Docs: man:cupsd(8)
 Main PID: 1213 (cupsd)
   Status: "Scheduler is running..."
    Tasks: 1 (limit: 23503)
   Memory: 3.7M
   CGroup: /system.slice/cups.service
           └─1213 /usr/sbin/cupsd -l

Aug 09 17:08:40 localhost.localdomain systemd[1]: Starting CUPS Scheduler...
Aug 09 17:08:41 localhost.localdomain systemd[1]: Started CUPS Scheduler.
......
[root@localhost ~]\# systemctl status cups.path
● cups.path - CUPS Scheduler
   Loaded: loaded (/usr/lib/systemd/system/cups.path; enabled; vendor preset: enab>
   Active: active (running) since Mon 2021-08-09 17:08:29 PDT; 1h 13min ago

Aug 09 17:08:29 localhost.localdomain systemd[1]: Started CUPS Scheduler.
......
```

​一些重要字段的含义：

| 字段       | 描述                      |
| -------- | ----------------------- |
| Loaded   | 服务单元是否已加载到内存中。          |
| Active   | 服务单元是否正在运行，若是，他已经运行了多久。 |
| Main PID | 服务的主进程 ID, 包括命令名称。      |
| Status   | 有关该服务的其他信息。             |

​*Status* 字段的不同值的含义：

| 值               | 描述                    |
| --------------- | --------------------- |
| loaded          | 单元配置文件已加载处理。          |
| active(running) | 正在通过一个或多个持续进程运行。      |
| active(exited)  | 已成功完成一次性配置。           |
| active(waiting) | 运行中，但正在等待事件。          |
| inactive        | 不在运行。                 |
| enabled         | 在系统引导时启动。             |
| disabled        | 未设为在系统引导时启动。          |
| static          | 无法启用，但可以由某一启用的单元自动启动。 |

### 验证服务的状态

​systemctl 命令提供了一些方法来验证服务的具体状态。

```bash
[root@localhost ~]\# systemctl is-active sshd # 是否活动
active    # inactive
[root@localhost ~]\# systemctl is-enabled sshd    # 是否自启 
enabled    # disabled
[root@localhost ~]\# systemctl is-failed sshd    # 是否启动失败
active    # failed，若已被停止，则 unknown 或 inactive
```

​列出所有启动失败的单元：

```bash
[root@localhost ~]\# systemctl --failed --type=service
0 loaded units listed. Pass --all to see loaded but inactive units, too.
To show all installed unit files use 'systemctl list-unit-files'.
```

## 控制系统服务

### 手动启停服务

​手动启停服务的原因：1. 更新服务；2. 更改配置文件；3. 卸载服务；4. 启动不常使用的服务。

​启动服务前，可以先用 `systemctl status` 命令验证服务是否未在运行。启停服务需要 root 权限进行操作。以 cups 服务为例，以下命令展示了启停服务的命令：

```bash
# 验证服务是否已经启动
[root@localhost ~]\# systemctl status cups
● cups.service - CUPS Scheduler
   Loaded: loaded (/usr/lib/systemd/system/cups.service; enabled; vendor preset: e>
   Active: active (running) since Mon 2021-08-09 17:08:41 PDT; 2h 0min ago
     Docs: man:cupsd(8)
 Main PID: 1213 (cupsd)
   Status: "Scheduler is running..."
    Tasks: 1 (limit: 23503)
   Memory: 3.7M
   CGroup: /system.slice/cups.service
           └─1213 /usr/sbin/cupsd -l

Aug 09 17:08:40 localhost.localdomain systemd[1]: Starting CUPS Scheduler...
Aug 09 17:08:41 localhost.localdomain systemd[1]: Started CUPS Scheduler.
# 停止 cups 守护进程服务并查看状态
[root@localhost ~]\# systemctl stop cups.service # .service 拓展可以省略，下同
[root@localhost ~]\# systemctl is-active cups    # 验证 cups 服务是否在活动
inactive    # 服务已停止
# 启动 cups 守护进程服务并查看状态
[root@localhost ~]\# systemctl start cups
[root@localhost ~]\# systemctl status cups
● cups.service - CUPS Scheduler
   Loaded: loaded (/usr/lib/systemd/system/cups.service; enabled; vendor preset: e>
   Active: active (running) (thawing) since Mon 2021-08-09 19:15:01 PDT; 1min 17s >
     Docs: man:cupsd(8)
 Main PID: 4284 (cupsd)
   Status: "Scheduler is running..."
    Tasks: 1 (limit: 23503)
   Memory: 1.9M
   CGroup: /system.slice/cups.service
           └─4284 /usr/sbin/cupsd -l

Aug 09 19:15:01 localhost.localdomain systemd[1]: Starting CUPS Scheduler...
Aug 09 19:15:01 localhost.localdomain systemd[1]: Started CUPS Scheduler.
```

### 手动重启服务和重新加载服务

​重启 = 停止 + 启动，因此重启后的服务进程 ID 会改变，并且在启动期间会关联新的进程 ID。若只是因为修改了某项支持重新加载的服务的配置文件，则可以直接重新加载该服务而无需重启该服务。

```bash
[root@localhost ~]\# systemctl restart cups
[root@localhost ~]\# systemctl status cups
● cups.service - CUPS Scheduler
   Loaded: loaded (/usr/lib/systemd/system/cups.service; enabled; vendor preset: e>
   Active: active (running) (thawing) since Mon 2021-08-09 19:24:21 PDT; 10s ago
     Docs: man:cupsd(8)
 Main PID: 4384 (cupsd)
   Status: "Scheduler is running..."
    Tasks: 1 (limit: 23503)
   Memory: 2.0M
   CGroup: /system.slice/cups.service
           └─4384 /usr/sbin/cupsd -l

Aug 09 19:24:21 localhost.localdomain systemd[1]: cups.service: Succeeded.
Aug 09 19:24:21 localhost.localdomain systemd[1]: Stopped CUPS Scheduler.
......
[root@localhost ~]\# systemctl reload cups # cups 服务不支持重载
Failed to reload cups.service: Job type reload is not applicable for unit cups.service.
[root@localhost ~]\# systemctl reload firewalld # firewalld 服务支持重载
[root@localhost ~]\# systemctl status firewalld # 最底下详情记录了重载的信息
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; enabled; vendor preset: enabled)
   Active: active (running) (thawing) since Mon 2021-08-09 17:08:40 PDT; 2h 19min ago
     Docs: man:firewalld(1)
  Process: 4443 ExecReload=/bin/kill -HUP $MAINPID (code=exited, status=0/SUCCESS)
 Main PID: 1116 (firewalld)
    Tasks: 3 (limit: 23503)
   Memory: 35.2M
   CGroup: /system.slice/firewalld.service
           └─1116 /usr/libexec/platform-python -s /usr/sbin/firewalld --nofork --nopid

Aug 09 17:08:38 localhost.localdomain systemd[1]: Starting firewalld - dynamic firewall daemon...
Aug 09 17:08:40 localhost.localdomain systemd[1]: Started firewalld - dynamic firewall daemon.
Aug 09 17:08:41 localhost.localdomain firewalld[1116]: WARNING: AllowZoneDrifting is enabled. This is considered an insecure configuration option. It will be removed in a future release. >
Aug 09 19:27:17 localhost.localdomain systemd[1]: Reloading firewalld - dynamic firewall daemon.
Aug 09 19:27:17 localhost.localdomain systemd[1]: Reloaded firewalld - dynamic firewall daemon.
Aug 09 19:27:17 localhost.localdomain firewalld[1116]: WARNING: AllowZoneDrifting is enabled. This is considered an insecure configuration option. It will be removed in a future release. 
```

​若不确定某服务是否具有重新加载配置文件更改的功能，则可以使用 `reload-or-restart` 选项来运行 systemctl 命令。如果重新加载功能可用，该命令将重新加载配置更改，否则，该命令将重新启动服务以实施新的配置更改：

```bash
[root@localhost ~]\# systemctl status NetworkManager # 支持重载
● NetworkManager.service - Network Manager
   Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled; vendor preset: enabled)
   Active: active (running) (thawing) since Mon 2021-08-09 17:08:40 PDT; 2h 28min ago
     Docs: man:NetworkManager(8)
  Process: 4531 ExecReload=/usr/bin/busctl call org.freedesktop.NetworkManager /org/freedesktop/NetworkManager org.freedesktop.NetworkManager Reload u 0 (code=exited, status=0/SUCCESS)
 Main PID: 1190 (NetworkManager)
    Tasks: 3 (limit: 23503)
   Memory: 10.7M
   CGroup: /system.slice/NetworkManager.service
           └─1190 /usr/sbin/NetworkManager --no-daemon

Aug 09 19:20:04 localhost.localdomain NetworkManager[1190]: <info>  [1628562004.4206] dhcp4 (ens160): option requested_subnet_mask => '1'
Aug 09 19:20:04 localhost.localdomain NetworkManager[1190]: <info>  [1628562004.4206] dhcp4 (ens160): option requested_time_offset => '1'
Aug 09 19:20:04 localhost.localdomain NetworkManager[1190]: <info>  [1628562004.4206] dhcp4 (ens160): option requested_wpad       => '1'
Aug 09 19:20:04 localhost.localdomain NetworkManager[1190]: <info>  [1628562004.4206] dhcp4 (ens160): option routers              => '192.168.30.2'
Aug 09 19:20:04 localhost.localdomain NetworkManager[1190]: <info>  [1628562004.4206] dhcp4 (ens160): option subnet_mask          => '255.255.255.0'
Aug 09 19:20:04 localhost.localdomain NetworkManager[1190]: <info>  [1628562004.4206] dhcp4 (ens160): state changed bound -> extended
Aug 09 19:36:30 localhost.localdomain systemd[1]: Reloading Network Manager.
Aug 09 19:36:30 localhost.localdomain NetworkManager[1190]: <info>  [1628562990.2331] audit: op="reload" arg="0" pid=4531 uid=0 result="success"
Aug 09 19:36:30 localhost.localdomain NetworkManager[1190]: <info>  [1628562990.2339] config: signal: SIGHUP (no changes from disk)
Aug 09 19:36:30 localhost.localdomain systemd[1]: Reloaded Network Manager.
[root@localhost ~]\# systemctl reload-or-restart NetworkManager
[root@localhost ~]\# systemctl status NetworkManager
● NetworkManager.service - Network Manager
   Loaded: loaded (/usr/lib/systemd/system/NetworkManager.service; enabled; vendor preset: enabled)
   Active: active (running) (thawing) since Mon 2021-08-09 17:08:40 PDT; 2h 28min ago
     Docs: man:NetworkManager(8)
  Process: 4547 ExecReload=/usr/bin/busctl call org.freedesktop.NetworkManager /org/freedesktop/NetworkManager org.freedesktop.NetworkManager Reload u 0 (code=exited, status=0/SUCCESS)
 Main PID: 1190 (NetworkManager)
    Tasks: 3 (limit: 23503)
   Memory: 10.8M
   CGroup: /system.slice/NetworkManager.service
           └─1190 /usr/sbin/NetworkManager --no-daemon

Aug 09 19:20:04 localhost.localdomain NetworkManager[1190]: <info>  [1628562004.4206] dhcp4 (ens160): option subnet_mask          => '255.255.255.0'
Aug 09 19:20:04 localhost.localdomain NetworkManager[1190]: <info>  [1628562004.4206] dhcp4 (ens160): state changed bound -> extended
Aug 09 19:36:30 localhost.localdomain systemd[1]: Reloading Network Manager.
Aug 09 19:36:30 localhost.localdomain NetworkManager[1190]: <info>  [1628562990.2331] audit: op="reload" arg="0" pid=4531 uid=0 result="success"
Aug 09 19:36:30 localhost.localdomain NetworkManager[1190]: <info>  [1628562990.2339] config: signal: SIGHUP (no changes from disk)
Aug 09 19:36:30 localhost.localdomain systemd[1]: Reloaded Network Manager.
Aug 09 19:37:18 localhost.localdomain systemd[1]: Reloading Network Manager.
Aug 09 19:37:19 localhost.localdomain NetworkManager[1190]: <info>  [1628563039.0110] audit: op="reload" arg="0" pid=4547 uid=0 result="success"
Aug 09 19:37:19 localhost.localdomain NetworkManager[1190]: <info>  [1628563039.0114] config: signal: SIGHUP (no changes from disk)
Aug 09 19:37:19 localhost.localdomain systemd[1]: Reloaded Network Manager.
```

### 列出单元依赖项

​某些服务要求首先运行其他服务，从而创建对其他服务的依赖项。其他服务并不在系统引导时启动，而是仅在需要时启动。在这两种情况下， systemd 和 systemctl 根据需要启动服务，不论是解决依赖项，还是启动不经常使用的服务。

​使用 `systemctl list-dependencies` 命令可以列出指定服务的依赖项树，如：

```bash
[root@localhost ~]\# systemctl list-dependencies cups # 列出 cups 服务的依赖项
cups.service
● ├─cups.path
● ├─cups.socket
● ├─system.slice
● └─sysinit.target
●   ├─dev-hugepages.mount
●   ├─dev-mqueue.mount
......
```

### 屏蔽未屏蔽的服务

​目的：屏蔽服务可防止管理员意外启动与其他服务冲突的服务。

​原理：在配置目录中创建指向 */dev/null* 文件的链接，该文件可阻止服务启动。

```bash
[root@localhost ~]\# systemctl mask cups    # 使用 mask 选项屏蔽某服务
Created symlink /etc/systemd/system/cups.service → /dev/null.
[root@localhost ~]\# systemctl status cups
● cups.service
   Loaded: masked (Reason: Unit cups.service is masked.)
   Active: active (running) (thawing) since Mon 2021-08-09 19:45:56 PDT; 11min ago
 Main PID: 4653 (cupsd)
   Status: "Scheduler is running..."
    Tasks: 1 (limit: 23503)
   Memory: 1.9M
   CGroup: /system.slice/cups.service
           └─4653 /usr/sbin/cupsd -l

Aug 09 19:45:56 localhost.localdomain systemd[1]: Starting CUPS Scheduler...
Aug 09 19:45:56 localhost.localdomain systemd[1]: Started CUPS Scheduler.
Aug 09 19:56:23 localhost.localdomain systemd[1]: cups.service: Current command va>
[root@localhost ~]\# systemctl stop cups
[root@localhost ~]\# systemctl status cups
● cups.service
   Loaded: masked (Reason: Unit cups.service is masked.)
   Active: inactive (dead) (thawing) since Mon 2021-08-09 19:57:53 PDT; 3s ago
 Main PID: 4653 (code=exited, status=0/SUCCESS)
   Status: "Scheduler is running..."

Aug 09 19:45:56 localhost.localdomain systemd[1]: Starting CUPS Scheduler...
Aug 09 19:45:56 localhost.localdomain systemd[1]: Started CUPS Scheduler.
Aug 09 19:56:23 localhost.localdomain systemd[1]: cups.service: Current command va>
Aug 09 19:57:53 localhost.localdomain systemd[1]: Stopping cups.service...
Aug 09 19:57:53 localhost.localdomain systemd[1]: cups.service: Succeeded.
Aug 09 19:57:53 localhost.localdomain systemd[1]: Stopped cups.service.
[root@localhost ~]\# systemctl start cups
Failed to start cups.service: Unit cups.service is masked.
[root@localhost ~]\# systemctl list-unit-files --type=service | grep masked
cups.service                               masked  
systemd-timedated.service                  masked  
[root@localhost ~]\# systemctl unmask cups    # 使用 unmask 接触对某服务的屏蔽
Removed /etc/systemd/system/cups.service.
[root@localhost ~]\# systemctl start cups
[root@localhost ~]\# systemctl status cups
● cups.service - CUPS Scheduler
   Loaded: loaded (/usr/lib/systemd/system/cups.service; enabled; vendor preset: e>
   Active: active (running) (thawing) since Mon 2021-08-09 19:59:12 PDT; 4s ago
     Docs: man:cupsd(8)
 Main PID: 4833 (cupsd)
   Status: "Scheduler is running..."
    Tasks: 1 (limit: 23503)
   Memory: 1.9M
   CGroup: /system.slice/cups.service
           └─4833 /usr/sbin/cupsd -l

Aug 09 19:59:12 localhost.localdomain systemd[1]: Starting CUPS Scheduler...
Aug 09 19:59:12 localhost.localdomain systemd[1]: Started CUPS Scheduler.
```

> 🟢 mask 选项并不会自动停止已经启动的服务，而且只能屏蔽对服务的启动操作。

### 设置某服务开启或关闭开机自启

```bash
[root@localhost ~]\# systemctl is-enabled cups    # 查看 cups 服务是否是开机自启的
enabled
[root@localhost ~]\# systemctl disable cups    # 关闭 cups 开机自启
Removed /etc/systemd/system/multi-user.target.wants/cups.service.
Removed /etc/systemd/system/multi-user.target.wants/cups.path.
Removed /etc/systemd/system/sockets.target.wants/cups.socket.
Removed /etc/systemd/system/printer.target.wants/cups.service.
[root@localhost ~]\# systemctl is-enabled cups    # 验证
disabled
[root@localhost ~]\# systemctl enable cups    # 开启 cups 开机自启
Created symlink /etc/systemd/system/printer.target.wants/cups.service → /usr/lib/systemd/system/cups.service.
Created symlink /etc/systemd/system/multi-user.target.wants/cups.service → /usr/lib/systemd/system/cups.service.
Created symlink /etc/systemd/system/sockets.target.wants/cups.socket → /usr/lib/systemd/system/cups.socket.
Created symlink /etc/systemd/system/multi-user.target.wants/cups.path → /usr/lib/systemd/system/cups.path.
[root@localhost ~]\# systemctl is-enabled cups    # 验证
enabled
```

​设置开机自启某服务并不会立即启动该服务。
