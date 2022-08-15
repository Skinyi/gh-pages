---
title: 监控和管理 Linux 进程（一）
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

## 了解 Linux 进程

### 什么是进程

​进程是已启动的可执行程序的运行中实例，其由以下组成部分组成：

- 已分配内存的地址空间；
- 安全属性；
- 程序代码的一个或多个运行线程；
- 进程状态；

​进程的环境包括：

- 本地和全局变量；

- 当前调度上下文；

- 分配的系统资源，如文件描述符和网络端口；

​现有的（父）进程复制自己的地址空间（fork）来创建一个新的（子）进程结构。每个新进程分配有一个唯一进程 ID(PID) ，满足跟踪和安全性的需要。PID 和父进程 ID(PPID) 是新进程环境的元素。任何进程都可创建子进程。所有进程都是第一个系统进程的后代，在 RHEL8 系统上，第一个系统进程是 systemd。

​通过fork例程，子进程继承安全性身份、过去和当前的文件描述符、端口和资源特权、环境变量，以及程序代码。随后，子进程可能 exec 其自己的程序代码。通常，父进程在子进程运行期间处于睡眠状态，设置一个在子进程完成时发出信号的请求 wait ）。在退出时，子进程已经关闭或丢弃了其资源和环境。唯一剩下的资源称为僵停，是进程表中的一个条目。父进程在子进程退出时收到信号而被唤醒，清理子条目的进程表，由此释放子进程的最后一个资源。然后，父进程继续执行自己的程序代码。

### 进程的状态

​在多任务处理操作系统中，每个 CPU（或 CPU 核心）在一个时间点上处理一个进程。在进程运行时，它对 CPU 时间和资源分配的直接要求会有变化。进程分配有一个状态，它随着环境的要求而改变。

![Linux 进程状态](images/linux/Linux-进程的状态.png)

​上图展示了 Linux 进程运行过程中的各种状态，下表是对以上状态的说明：

<table>
    <tr>
        <th>名称</th>
        <th>标志</th>
        <th>内核定义的状态名称</th>
        <th>描述</th>
    </tr>
    <tr>
        <td>运行</td>
        <td>R</td>
        <td>TASK_RUNNING</td>
        <td>进程正在 CPU 上执行，或者正在等待运行。处于运行中（或可运行）状态时，进程可能正在执行用户例程或内核例程（系统调用），或者已排队并就绪。</td>
    </tr>
    <tr>
        <td rowspan="4">睡眠</td>
        <td>S</td>
        <td>TASK_INTERRUPTIBLE</td>
        <td>进程正在等待某一条件：硬件请求、系统资源访问或信号。当事件或信号满足该条件时，该进程将返回到运行中。</td>
    </tr>
    <tr>
        <td>D</td>
        <td>TASK_UNINTERRUPTIBLE</td>
        <td>此进程也在睡眠，但与 S 状态不同，不会响应信号。仅在进程中断可能会导致意外设备状态的情况下使用。</td>
    </tr>
    <tr>
        <td>K</td>
        <td>TASK_KILLABLE</td>
        <td>与不可中断的 D 状态相同吧，但有所修改，允许等待中的任务响应要被中断（彻底退出）的信号。实用程序通常将可中断的进程显示为 K 状态。</td>
    </tr>
    <tr>
        <td>I</td>
        <td>TASK_REPORT_IDLE</td>
        <td>D 状态的一个子集。在计算负载平均值时，内核不会统计这些进程。用于内核线程。设置了 TASK_UNINTERRUPTABLE 和 TASK_NOLOAD 标志。类似于 TASK_KILLABLE，也是 D 状态的一个子集。它接受致命信号。</td>
    </tr>
    <tr>
        <td rowspan="2">已停止</td>
        <td>T</td>
        <td>TASK_STOPPED</td>
        <td>进程已被停止（暂停），通常是通过用户或其他进程发出的信号。进程可以通过另一信号返回到运行中状态，继续执行（恢复）。</td>
    </tr>
    <tr>
        <td>T</td>
        <td>TASK_TRACED</td>
        <td>正在被调试的进程也会临时停止，并且共享同一个 T 状态标志。</td>
    </tr>
    <tr>
        <td rowspan="2">僵停</td>
        <td>Z</td>
        <td>EXIT_ZOMBIE</td>
        <td>子进程在退出时向父进程发出信号。除进程身份（PID）之外的所有资源都已释放。</td>
    </tr>
    <tr>
        <td>X</td>
        <td>EXIT_DEAD</td>
        <td>当父进程清理（获取）剩余的子进程结构时，进程现在已彻底释放。此状态从不会在进程列出实用程序中看到。</td>
    </tr>
</table>

​在对系统进行故障排除时，了解内核如何与进程通信以及进程如何相互通信非常重要。在创建进程时，系统会为进程分配一个状态。top命令的 S 列或 ps 的 STAT 列显示每个进程的状态。在单 CPU 系统上，一次只能运行一个进程。可以看到多个状态为 R 的进程。然而，并非所有这些进程都在持续运行，其中一些将处于等待状态。

```bash
[root@localhost ~]\# ps aux
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root           1  0.0  0.3 245648 14544 ?        Ss   Apr25   0:16 /usr/lib/systemd/systemd --switched-root --system --deserialize 18
root           2  0.0  0.0      0     0 ?        S    Apr25   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   Apr25   0:00 [rcu_gp]
root           4  0.0  0.0      0     0 ?        I<   Apr25   0:00 [rcu_par_gp]
......
```

​可以使用信号暂停、停止、恢复、终止和中断进程，信号可以由其他进程、内核本身或登陆系统的用户使用。以下会有详细介绍。

### 列出进程

​`ps` 命令可以用于列出当前的进程，它可以提供详细的进程信息，包括：

- 用户标识符（UID），它确定进程的特权；
- 唯一进程识别符（PID）；
- CPU 和已经花费的实时时间；
- 进程在各种位置上分配的内存数量；
- 进程 stdout 的位置，称为控制终端；
- 当前的进程状态。

> 🟢 常见选项组合
>
> - aux: 显示包括五控制终端的进程在内的所有进程，注意 aux != -aux；
>
> USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
>
> root           1  0.0  0.3 245648 14544 ?        Ss   Apr25   0:16 > /usr/lib/systemd/systemd --switched-root --system --deserialize 18
>
> - lax: 长列表提供更多技术详细信息，但可以通过避免查询用户名来加快显示。
>
> F   UID     PID    PPID PRI  NI    VSZ   RSS WCHAN  STAT TTY        TIME COMMAND
>
> 4     0       1       0  20   0 245648 14544 do_epo Ss   ?          0:16 > /usr/lib/systemd/systemd --switched-root --system --deserialize 18
>
> - \-ef: 
>
> UID          PID    PPID  C STIME TTY          TIME CMD
>
> root           1       0  0 Apr25 ?        00:00:16 /usr/lib/systemd/systemd --> switched-root --system --deserialize 18
>
>- 没有选项：选择具有与当前用户相同的有效用户 ID(EUID) 并于调用 ps 所处统一终端关联的所有进程。
>
> PID TTY          TIME CMD
>
> 19667 pts/0    00:00:00 bash

> 🟢 使用 `ps -ef | grep [PROC_NAME]` 时注意此命令会将 grep 命令自身的进程也列出一份。

> 🟢 其他事项
>
> - 方括号中的进程（通常位于列表顶部）为调度的内核线程；
> - 僵停列为 exiting 或 defunct；
> - ps 的输出仅显示一次。使用 top 来获得动态更新的进程显示；
> - ps 可以采用树形显示格式（f 选项），以便查看父进程和子进程之间的关系，（更直观的可以使用 pstree 命令）；
> - 默认输出按进程 ID 编号排序。

## 作业控制

### 进程组、作业、会话

​Linux 下进程不仅有自己独一无二的 PID，还有其属于哪个进程组的标识 PGID。如果某进程的 PGID 等于其自身的 PID，那么此进程就可以被称为与它具有同一 PGID 的进程组的组长进程。当一个程序创建一个进程组时，其创建进程组的这个进程就为组长进程。程序实例的生命周期只与进程组中的最后一个进程有关。Shell 的作业控制功能控制的对象是作业或者进程组而不是进程。每个终端前台只能运行一个进程组或一个组或一个作业，而每个终端后台可以运行多个进程组和多个作业。会话是一个或多个进程组的集合。

```bash
[root@localhost ~]\# ps -ajx | grep 19667
   PPID     PID    PGID     SID TTY        TPGID STAT   UID   TIME COMMAND
  19658   19667   19667   19667 pts/0      20514 Ss       0   0:00 -bash					#			同一			
  19667   20514   20514   19667 pts/0      20514 R+       0   0:00 ps -ajx					# 同一个	  个会
  19667   20515   20514   19667 pts/0      20514 S+       0   0:00 grep --color=auto 19667	# 进程组	  话
```

​ps 命令在 TTY 列中显示进程的控制终端的设备名称。某些进程（如系统守护进程）由系统启动，并不是从 shell 提示符启动。这些进程没有控制终端，也不是作业的成员，无法转至前台。ps 命令在 TTY 列中针对这些进程显示一个问号。

### 查看当前会话的作业列表

```bash
jobs [-lnprs] [jobspec ...] # 显示当前会话所有（或提供 jobspec 参数指定的）作业的状态
# 选项：
-l, # 额外列出进程的 PID
-n, # 列出最后一次通知后改变运行状态的进程
-p, # 只列出进程的 PID
-r, # 仅限输出正在运行的作业
-s, # 仅限输出已经停止的作业
```

### 将作业放在前台执行

- 对于将要运行的程序：在命令行的结尾处不附加符号 `&`，这样程序所对应的作业直接在前台下运行。

- 把后台作业拉到前台运行：执行命令

  ~~~ bash
  fg [%]作业号	# [%] 表示 % 为可选
  ~~~

> 🟢 当在前台下运行一个作业时，shell 会被拉到后台，因为前台只能运行一个作业或者进程组。

### 将作业放在后台执行

- 对于将要运行的程序：在命令行的结尾处附加符号 `&`。

- 把一个正在前台运行的程序放到后台运行：先使用 `Ctrl` + `Z` 快捷键将前台的作业放到后台，此时该后台作业是处于暂停状态的，使用以下命令将其从停止状态恢复到运行状态：

~~~bash
bg [%]作业号	# [%] 表示 % 为可选
~~~

### 使作业在关闭终端后也能运行

```bash
nohup COMMAND [ARG]... [> FILE] 	# 执行命令，忽略挂起信号（一般后面追加个 & 配合使用） 
```

### 终止当前运行的前台任务

​使用 `Ctrl` + `C` 快捷键。

## 中断进程

### 使用信号控制进程

​信号是传递至进程的软件中断。信号向执行中的程序报告事件。生成信号的事件可以是错误或外部事件（I/O请求或定时器过期），或者来自于显式使用信号发送命令或键盘序列。以下是一些常见的中断信号的介绍（更详细的可参见 manpage signal(7)）：

| 信号编号 |  名称  | 定义 | 用途                                                         |
| :------: | :----: | :--: | ------------------------------------------------------------ |
|    1     | SIGHUP | 挂起 | 用于报告终端控制进程的终止。也用于请求进程重新初始化（重新加载配置）而不终止。 |
|    2     | SIGINT | 键盘中断 | 导致程序终止。可以被拦截或处理。通过按 INTR 键盘序列（CTRL + C）发送。 |
|    3     | SIGQUIT | 键盘退出 | 与 SIGINT 相似；在终止时添加进程转储。通过按 QUIT 键序列（CTRL + \）发送。 |
|    9     | SIGKILL | 中断，无法拦截 | 导致立即终止程序。无法被拦截、忽略或处理；总是致命的。 |
| 15默认 | SIGTERM | 终止 | 导致程序终止。和 SIGKILL 不同，可以被拦截、忽略或处理。要求程序终止的“友好”方式；允许自我清理。 |
| 18 | SIGCONT | 继续 | 发送至进程使其恢复（若已停止）。无法被拦截。即使被处理，也始终恢复进程。 |
| 19 | SIGSTOP | 停止，无法拦截 | 暂停进程。无法被拦截或处理。 |
| 20 | SIGTSTP | 键盘停止 | 和SIGSTOP不同，可以被拦截、忽略或处理。通过按 SUSP 键序列（CTRL + Z）发送。 |

> 🟢 信号编号跟硬件平台有关，上表中的编号以 x86_64 系统为例。作为命令使用时建议使用信号名称作为参数。

### 向作业发送信号

```bash
kill [-s sigspec | -n signum | -sigspec] pid | jobspec ... # 向作业发送信号，未指定信号的话则默认发送 SIGTERM 信号
kill -l [sigspec] # 列出所有可用的中断名称或指定 sigspec 的中断号
# 常见选项：
-s SIG # SIG 是信号名称
-n SIGNUM # SIGN 是信号编码
```

```bash
killall [-s sigspec | -sigspec] NAME... # 向多个作业发送信号，未指定信号的话则默认发送 SIGTERM 信号
# 该命令还有一些筛选选项，具体用到时可参见其 --help 选项的说明
```

```bash
pkill [-<SIG> | --signal <SIG>] [OTHER_OPTIONS] PATTERN # 向多个作业发送信号，未指定信号的话则默认发送 SIGTERM 信号
# 此命令的筛选条件更详细，且进程名都是可以使用模式匹配的
```

### 以管理员的身份注销其他用户的登陆

​要注销某个用户，首先确定要终止的登录会话。使用 `w` 命令列出用户登陆和当前运行的进程。记录 TTY 和 FROM 列，以确定要关闭的会话。

```bash
[root@localhost ~]\# w
 11:06:17 up 1 day,  2:29,  2 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
student  tty2     tty2             15Apr21 10days  5:04   1.20s /usr/libexec/track
root     pts/0    192.168.30.1     07:35    1.00s  0.25s  0.01s w
```

> 🟢 TTY 列：pts/N 代表图形界面终端窗口或远程登录会话相关联的伪终端；ttyN 代表用户位于一个系统控制台、替代控制台或其他直接连接的终端设备上。

​使用 pgrep 确定要中断的 PID 编号：

```bash
[root@localhost ~]\# pgrep -l -u student	# pgrep 与 pkill 命令的用法类似，作用不同
2238 systemd
2248 (sd-pam)
2257 pulseaudio
2263 gnome-keyring-d
......
[root@localhost ~]\# pkill -SIGKILL -u student	# 杀死 student 用户的所有进程
[root@localhost ~]\# pgrep -l -u student
```

> 🟢 通常建议不要直接使用 *SIGKILL* 杀死进程，而是在尝试 *SIGTERM* 和 *SIGINT* 不起作用之后使用。
