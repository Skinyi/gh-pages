---
title: 系统性能调优
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

​系统管理员可以基于多种用例工作负载来调整各种设备设置，以此优化系统性能。*tuned* 守护进程会利用反映特定工作负载要求的调优配置文件，以静态和动态两种方式应用调优调整。

### 静态调优

​*tuned* 守护进程会在服务启动时或选择新的调优配置文件时应用系统设置。静态调优会对配置文件中由 *tuned* 在运行时应用的预定义 *kernel* 参数进行配置。对于静态调优而言，内核参数是针对整体性能预期而设置的，不会随着活跃度的变化而进行调整。

### 动态调优

​对于动态调优而言，*tuned* 守护进程会监视系统活动，并根据运行时行为的变化来调整设置。从所选调优配置文件中声明的初始设置开始，动态调优会不断进行调优调整以适应当前工作负载。

### 配置文件

​*tuned* 应用提供的配置文件分为以下几个类别：

- 节能型配置文件；
- 性能提升型配置文件；

​性能提升型配置文件中包括侧重于以下方面的配置文件：

- 存储和网络的低延迟；
- 存储和网络的高吞吐量；
- 虚拟机性能；
- 虚拟化主机性能。

| 调优配置文件           | 用途                                                         |
| ---------------------- | ------------------------------------------------------------ |
| balanced(均衡)         | 适合需要在节能和性能之间进行折衷的系统。                     |
| desktop                | 从 balanced 配置文件衍生而来。加快交互式应用响应速度。       |
| throughput-performance | 调优系统，以获得最大吞吐量。                                 |
| latency-performance    | 适合需要牺牲能耗来获取低延迟的服务器系统。                   |
| network-latency        | 从 latency-performance 配置文件衍生而来。它可以启用额外的网络调优参数以提供低网络延迟。 |
| network-throughput     | 从 throughput-performance 配置文件衍生而来。应用其他网络调优参数，以获得最大网络吞吐量。 |
| powersave              | 调优系统，以最大程度实现节能。                               |
| oracle                 | 基于 throughput-performance 配置文件，针对 oracle 数据库负载进行优化。 |
| virtual-guest          | 当系统在虚拟机上运行时，调优系统以获得最高性能。             |
| virtual-host           | 当系统充当虚拟机宿主机时，调优系统以获得最高性能。           |

​除此之外还有：*accelerator-performance*、*hpc-compute*、*intel-sst*、*optimize-serial-console* 等配置文件。

## 从命令行管理调优配置文件

​使用 `tuned-adm` 命令来更改 *tuned* 守护进程的设置。此命令可以查询当前设置、列出可用的配置文件、为系统推荐调优配置文件、直接更改配置文件或关闭调优。

​系统管理员使用 *tuned-adm active* 来确定当前活动的调优配置文件。

```bash
[root@localhost ~]\# tuned-adm active
Current active profile: virtual-guest
```

​使用 `tuned-adm list` 命令可以列出所有可用的调优配置文件，包括内置的配置文件和系统管理员创建的自定义调优配置文件。

```bash
[root@localhost ~]\# tuned-adm list
Available profiles:
- accelerator-performance     - Throughput performance based tuning with disabled higher latency STOP states
- balanced                    - General non-specialized tuned profile
- desktop                     - Optimize for the desktop use-case
- hpc-compute                 - Optimize for HPC compute workloads
- intel-sst                   - Configure for Intel Speed Select Base Frequency
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- optimize-serial-console     - Optimize for serial console use.
- powersave                   - Optimize for low power consumption
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: virtual-guest
```

​使用 `tuned-adm profile <profilename>` 激活更加符合当前机器调优要求的其它配置文件。

```bash
[root@localhost ~]\# tuned-adm profile desktop
[root@localhost ~]\# tuned-adm active
Current active profile: desktop
```

​使用 `tuned-adm recommend` 命令来获得系统推荐的调优配置文件，该机制用于在安装后确定系统的默认配置文件。 

```bash
[root@localhost ~]\# tuned-adm recommend
virtual-guest
```

​使用 `tuned-adm off` 关闭 *tuned-adm* 调优。

```bash
[root@localhost ~]\# tuned-adm off
[root@localhost ~]\# tuned-adm active
No current active profile.
```

​如需开启调优，使用 `tuned-adm profile <profilename>` 启用调优配置文件即可。

## 调优进程调度

​计算机系统上运行的进程线程数超出了 CPU 数量，通过使用多任务技术，Linux 和其他操作系统可运行超出其处理单元数的进程。操作系统进程调度程序在单个核心上的进程之间快速切换，从而给人一种有多个进程在同时运行的印象。

​不同进程的重要程度不同，进程调度程序可以配置为针对不同的进程采用不同的调度策略。常规系统上运行的大多数进程所使用的调度策略称为 SCHED_OTHER（也称为SCHED_NORMAL），但还有其他一些策略可满足各种各样的工作负载需求。为进程设置静态的 *nice* 值可以设置进程的静态优先度。Linux 的 nice 值范围为 -20 到 19，<b>数值越大则优先级越低</b>，默认情况下，进程将继承其父进程的 nice 级别，<b>通常为 0</b>，优先级较高的进程更加容易获得 CPU 资源。

### 查看进程的 nice 级别

​使用 `top` 命令可通过交互方式查看和管理进程。该命令的输出默认显示 nice 级别（*NI*列）和优先级（*PR*列），在 *top* 界面中，nice 级别映射至内部系统优先级的关系是：NI值 + 20 = PR值。

```bash
[root@localhost ~]\# top
top - 14:55:11 up  6:11,  2 users,  load average: 0.05, 0.01, 0.00
Tasks: 333 total,   1 running, 332 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.5 us,  0.2 sy,  0.0 ni, 99.0 id,  0.0 wa,  0.2 hi,  0.2 si,  0.0 st
MiB Mem :   3709.4 total,   1792.8 free,   1289.7 used,    626.9 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   2169.3 avail Mem 

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND        
   2374 student   20   0 2854124 180868  92620 S   0.7   4.8   0:22.67 gnome-shell    
   2010 gdm       20   0 2789052 159172  89064 S   0.3   4.2   0:13.35 gnome-shell    
      1 root      20   0  180464  14660   9212 S   0.0   0.4   0:06.62 systemd        
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.09 kthreadd       
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp         
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp
      . ......
```

​使用 `ps` 命令搭配适当的选项也可以查看到进程的 nice 值，该命令结果中值为 *-* 的 nice 值代表该进程按照其他调度策略运行，并被调度程序解读为具有较高的优先级。进程采用的进程调度方式可在 *CLS* 列查看，其中 *TS* 代表采用了 SCHED_NORMAL 调度策略运行。

```bash
[root@localhost ~]\# ps axo pid,comm,cls,nice --sort=-nice
    PID COMMAND         CLS  NI
     34 khugepaged       TS  19
   1017 alsactl         IDL   -
   2763 tracker-miner-a IDL   -
   2764 tracker-miner-f  TS  19
     33 ksmd             TS   5
   1034 rtkit-daemon     TS   1
   .... ......
```

### 以指定 nice 值启动某进程

​使用 `nice` 命令来以指定 nice 值（默认为 10）来启动一个程序。

```bash
[root@localhost ~]\# nice sha1sum /dev/zero &
[1] 6788
[root@localhost ~]\# ps -o pid,comm,nice 6788
    PID COMMAND          NI
   6788 sha1sum          10
```

​可以使用 *-n* 选项来指定 nice 值。

```bash
[root@localhost ~]\# nice -n 20 sha1sum /dev/zero &	# 指定 nice 超出边界则会以边界值为准
[1] 6808
[root@localhost ~]\# ps -o pid,comm,nice 6808
    PID COMMAND          NI
   6808 sha1sum          19
```

### 更改运行中的进程的 nice 值

​使用 `renice` 命令更改运行中的进程的 nice 值。

```bash
[root@localhost ~]\# renice -n 15 6808
6808 （process ID）old priority 19, new priority 15
```

​也可以搭配 `top` 命令，在 *top* 界面中，按下 *r* 选项会提示：*PID to renice [default pid = \*\*\*\*]*，然后输入要更改的进程的 pid 和 nice 级别。

```bash
top - 15:19:05 up  6:35,  2 users,  load average: 0.00, 0.02, 0.04
Tasks: 333 total,   1 running, 332 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.1 us,  0.1 sy,  0.0 ni, 99.6 id,  0.0 wa,  0.1 hi,  0.1 si,  0.0 st
MiB Mem :   3709.4 total,   1792.2 free,   1290.1 used,    627.1 buff/cache
MiB Swap:   2048.0 total,   2048.0 free,      0.0 used.   2168.9 avail Mem 
PID to renice [default pid = 2522] 
    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND        
   2522 root      20   0  205208  32316  10372 S   0.2   0.9   0:49.24 sssd_kcm       
   1039 root      20   0  369624  12628  10504 S   0.1   0.3   0:34.17 vmtoolsd       
   2739 student   20   0  534444  38860  31804 S   0.1   1.0   0:34.34 vmtoolsd       
   6904 root      20   0   65620   5364   4328 R   0.1   0.1   0:00.04 top            
   .... ......
```
