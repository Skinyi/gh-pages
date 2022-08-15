---
title: 设置任务计划
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

## 设置一次性任务

​使用 `at` 命令可以设置在将来某个设定的点运行一个命令或一组命令。此命令的执行和生效依赖于 *atd* 服务，要使用该命令，请确保已经安装该软件包且 *atd* 服务已经启动。

​使用 *at* 设置的任务会进入 *a* 到 *z* 26 个队列，作业按字母顺序排列，队列越往后则优先级越低。

### 用法

​使用 `at TIMESPEC` 来设置新的作业：

```bash
[root@localhost ~]\# at now + 5min
warning: commands will be executed using /bin/sh
at> # 接着输入自己想设置的命令，按 Ctrl + D 来完成输入
```

​对于复杂的长串命令，使用重定向导入要比手动输入更方便，如：

```bash
[root@localhost ~]\# at now + 5min < myscript.txt # 其中 myscript.txt 中已设置好要执行的命令
```

​其中 *TIMESPEC* 的形式较为灵活，以下列出了一些可用的形式：

- now + 5min;
- teatime tomorrow;（下午茶时间为 16:00）

- noon + 4days;
- 5pm august 3 2021。

### 管理计划的任务

​使用命令 `atq` 或 `at -l` 来获得当前用户的待处理作业的列表。

```bash
[root@localhost ~]\# at -l
3	Fri Sep 17 11:04:00 2021 a root
# 编号 日期 时间 所处的队列编号 作业所有者即执行者
```

​使用命令 `at -c JOBNUMBER` 命令查看指定作业编号的计划作业的环境和执行内容。

```bash
[root@localhost ~]\# at -c 3
#!/bin/sh
# atrun uid=0 gid=0
# mail root 0
umask 22
......
```

​使用命令 `atrm JOBNUMBER` 命令来删除计划的作业。

```bash
[root@localhost ~]\# atrm 3
Cannot find jobid 3
```

> 使用 `watch` 命令可以设置每一段时间执行命令并将其结果输出到标准输出设备上。

## 设置周期性作业

​使用 `crontab` 命令可以设置周期性执行的作业，此命令的设置和执行依赖于 *crond* 守护进程，*crond* 守护进程会读取多个配置文件（每个用户对应一个配置文件）以及一组系统范围内的文件。这些配置文件使用户和管理员拥有细微的控制权，可以控制应执行周期作业的时间。

### 用法

| 命令             | 用途                                                         |
| ---------------- | ------------------------------------------------------------ |
| crontab -l       | 列出当前用户的计划作业                                       |
| crontab -r       | 删除当前用户的所有作业                                       |
| crontab -e       | 编辑当前用户的作业                                           |
| crontab filename | 删除所有作业，并替换为从 filename 读取的作业。如果没有指定文件，则使用 stdin。 |

> Root 用户可以使用 `crontab -u <USER>` 命令来管理其他用户的作业，不应以 root 用户直接设置系统作业，而应通过修改 /etc/crontab 配置文件。

​`crontab -e` 命令会默认调用 vim 编辑器（除非 EDITOR 环境变量指定其他设置）来编辑任务计划设置文件，每行只输入一个作业，除此之外也可使用注释、变量以及空行。其中常见变量的设置包括 SHELL 变量和 MAILTO 变量。

​Crontab 文件中的字段按以下顺序显示：分钟、小时、日、月、星期、命令。

> 若指定的日和星期都不是 *\**，则该命令会在每个指定的日期或星期执行。

​前五个字段全部使用相同的语法规则：

- *\** 表示 “无关紧要” 或始终；

- *数字* 可用于指定不同时间概念，特别是 1-6 分别表示星期一至星期六，0 或 7 都可以代表星期天；

- *x\-y* 表示范围，(x，y]；

- *x,y* 表示列表，列表也可包含范围，如分钟列可以设置成 *5, 10-13, 17*；

- *\*/x* 表示每 x 时间的间隔，如指定分钟列 \*/7 代表每 7 分钟运行一次作业。

​此外，月份和星期也可以使用三个字母的缩写来表示，如：Jan、Feb、Mar...以及 Mon、Tue、Wed...。

> 命令字段中若包含未转义的 *%* 符号，则该百分比符号将被当作换行字符，且百分比之后的所有内容将传给 *stdin* 中的命令。

### 作业示例

​以下作业将在每年 2 月 2 日上午 9 点准点执行命令 /usr/local/bin/yearly_backup。

```bash
0 9 2 2 * /usr/local/bin/yearly_backup
```

​以下作业将在每个工作日午夜前的两分钟运行命令 /usr/local/bin/daily_report。

```bash
58 23 * * 1-5 /usr/local/bin/daily_report
```

​以下作业将在每个工作日（周一到周五）上午 9 点执行 mutt 命令，从而将邮件消息 Checking in 发送给收件人 boss@example.com 。

```bash
0 9 * * 1-5 mutt -s "Check in" boss@example % Hi there boss, just checking in.
```

## 周期性系统作业

​这部分内容可以参考文章：https://cloud.tencent.com/developer/article/1619542，写的非常详细。

​系统管理员经常需要运行周期性作业，最佳的做法是从系统账户而不是从用户账户运行这些作业，即不用使用 `crontab` 命令来安排这些作业，而是要使用系统范围的 crontab 文件。

​/etc/crontab 文件的语法跟普通用户创建 crontab 任务类似，crontab 文件中需额外指定一个用户字段：

```bash
[root@localhost ~]\# cat /etc/crontab | grep ^# 
# For details see man 4 crontabs
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

​周期性系统作业可以在两个位置定义：*/etc/crontab* 文件和 */etc/cron.d* 目录中的自定义文件（推荐）。除此之外，crontab 系统中还包含需要每小时、每天、每周和每月运行的脚本的存储库，这些存储库分别对应于名为 */etc/cron.hourly*、*/etc/cron.daily*、*/etc/cron.weekly*、*/etc/cron.monthly* 的目录。这些目录中存放的是脚本文件而不是 crontab 配置文件。

> 这些文件夹中的脚本文件需要被授予执行权限，否则将不会触发。

​为了处理由于系统关机等某些原因导致的一些任务计划由于超出时间范围而未被 crontab 执行的问题，anacron 被引入，anacron 每小时被 crond 服务执行一次以检查一些任务计划是否在设定的时间执行，若有则执行该任务，待执行完毕或者无任何逾期的任务计划时就会结束 anacron 进程。

​在 /etc/cron.hourly 可以找到 anacron 的配置执行文件：

```bash
#!/bin/sh
# Check whether 0anacron was run today already
if test -r /var/spool/anacron/cron.daily; then
    day=`cat /var/spool/anacron/cron.daily`
fi
if [ `date +%Y%m%d` = "$day" ]; then
    exit 0
fi

# Do not run jobs when on battery power
online=1
for psupply in AC ADP0 ; do
    sysfile="/sys/class/power_supply/$psupply/online"

    if [ -f $sysfile ] ; then
        if [ `cat $sysfile 2>/dev/null`x = 1x ]; then
            online=1
            break
        else
            online=0
        fi
    fi
done
if [ $online = 0 ]; then
    exit 0
fi
/usr/sbin/anacron -s
```

## SYSTEMD 定时器

### 一些操作系统设定的计时器

```bash
[root@localhost etc]\# systemctl status *timer
● dnf-makecache.timer - dnf makecache --timer
   Loaded: loaded (/usr/lib/systemd/system/dnf-makecache.timer; enabled; vendor preset>
   Active: active (waiting) since Fri 2021-09-17 09:07:25 CST; 6h ago
  Trigger: Fri 2021-09-17 16:23:21 CST; 36min left

Sep 17 09:07:25 localhost.localdomain systemd[1]: Started dnf makecache --timer.
......
```

### 自定义定时器

​在 */etc/systemd/system* 下添加自定义定时器单元配置文件，并添加如下配置：

```ini
[Unit]
Description=”对该定时器功能的描述“

[Timer]
# 任务执行的时间
OnCalendar=*-*-* *:*:00	
# 精度（秒），允许浮动的偏差
AccuracySec=24h
# 是否
Persistent=true
# 最大随机延迟时间
#RandomizedDelaySec=14400
# 定时器被激活时执行的服务
Requires=MyService.service

[Install]
WantedBy=timers.target
```
