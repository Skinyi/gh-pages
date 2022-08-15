---
title: 监控和管理 Linux 进程（二）
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

## 监控进程活动

### Linux 负载平均值

​负载平均值是 Linux 内核提供的一种度量方式，它可以简单地表示一段时间内感知的系统负载。这个数值可以用来粗略衡量待处理的系统资源请求数量，并确定系统负载是随时间增加还是减少。

​根据处于可运行和不可中断状态的进程数，内核会每五秒钟收集一次当前的负载数。通过汇总这些数值，可以得到最近一分钟、五分钟和十五分钟内的指数移动平均值。

* 负载数基本上是根据准备运行的进程数（进程状态为 `R`）和等待 `I/O` 完成的进程数（进程状态为 `D`）而得到的；
* 一些 UNIX 系统仅考虑 CPU 使用率或运行队列长度来指示系统负载。Linux 还包含磁盘或网络利用率，因为他们与 CPU 负载一样会对系统性能产生重大影响。遇到负载平均值很高但 CPU 活动很低时，请检查磁盘和网络活动。

​负载平均值可粗略衡量在可以执行其他任何作业之前，有多少进程当前在等待请求完成。请求可能是用于运行进程的 CPU 时间。或者，请求可能是让关键磁盘 I/O 操作完成；在请求完成之前，即使 CPU 空闲，也不能在 CPU 上运行该进程。无论是哪种方式，都会影响系统负载；系统的运行看起来会变慢，因为有进程正在等待运行。

### 使用 `uptime` 命令查看系统当前负载平均值

```bash
uptime # 显示系统运行时长及过去 1，5，15 分钟的平均负载
-p, --pretty # 只显示开机运行时长
-s, --since # 只显示开机运行的时间点
# 例子：
[student@localhost ~]$ uptime -s
2021-06-02 16:30:42
[student@localhost ~]$ uptime -p
up 17 minutes
[student@localhost ~]$ uptime
16:48:01 up 17 min,  1 user,  load average: 0.13, 0.14, 0.13
```

### 使用 `w` 命令查看系统当前负载

```bash
w # 查看已登录的用户及其当前活动
-h, --no-header # 不要打印标题行
-s, --short # 简洁模式
-f, --from # 打印用户的登陆终端信息
-i, --ip-addr # 在 from 列打印 IP 地址而不是主机名
-u, --no-current # 当指出当前进程和 cpu 时间时忽略用户名
# 例子：
[student@localhost ~]$ w 
 16:58:40 up 27 min,  1 user,  load average: 0.04, 0.03, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
student  tty2     tty2             16:31   27:49  49.47s  0.15s /usr/libexec/track
[student@localhost ~]$ w -h
student  tty2     tty2             16:31   28:16  50.82s  0.15s /usr/libexec/track
[student@localhost ~]$ w -s
 16:59:27 up 28 min,  1 user,  load average: 0.07, 0.04, 0.06
USER     TTY      FROM              IDLE WHAT
student  tty2     tty2             28:36  /usr/libexec/tracker-miner-fs
```

### 使用 `top` 命令查看系统当前负载

```bash
top # 动态列出 linux 进程
[student@localhost ~]$ top
top - 17:00:39 up 29 min,  1 user,  load average: 0.11, 0.07, 0.07
Tasks: 329 total,   1 running, 328 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.8 us,  2.5 sy,  0.0 ni, 94.3 id,  0.0 wa,  1.2 hi,  0.2 si,  0.0 st
MiB Mem :   3709.4 total,   1835.5 free,   1239.4 used,    634.4 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   2217.5 avail Mem 
    
PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND     
 11 root      20   0       0      0      0 I   0.3   0.0   0:01.85 rcu_sched 
...
# 进程号 所属用户             虚拟内存 常驻内存   进程状态            CPU时间 进程命令名称
```

​ top 命令常见按键操作：

| 按键  | 用途  |
| --- | --- |
| ? or h | 关于快捷键操作的帮助信息。 |
| l、t、m | 切换到负载、线程和内存标题行。 |
| 1   | 标题中切换显示单独 CPU 信息或所有 CPU 的汇总。 |
| s \* | 更改刷新（屏幕）率，以带小数的秒数表示（如0.5、1、5）。 |
| b   | 切换反向突出显示 `运行中` 的进程；默认为仅粗体。 |
| shift + b | 在显示中使用粗体，用于标题以及运行中的进程。 |
| shift + h | 切换线程；显示进程摘要或单独线程。 |
| u、shift + u | 过滤任何用户名称（有效、真实）。 |
| shift + m | 按照内存使用率降序排列进程列表。 |
| shift + p | 按照处理器使用率降序排列进程列表。 |
| k \* | 中断进程。若有提示，输入 PID ，再输入 signal。 |
| r \* | 调整进程的 nice 值。若由提示，输入 PID，再输入 nice_value。 |
| shift + w | 写入（保存）当前的显示配置，以便在下一次重新启动 top 时使用。 |
| q   | 退出。 |
| f   | 通过启用或禁用字段的方式来管理列。同时还允许您为 top 设置排序字段。 |
| \* 注： | 如果 top 在安全模式中启动，则不可用。 |
