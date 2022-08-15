---
title: 系统时间维护
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

​为了确保系统日志和日志记录的事件有正确的时间戳以及计算机的正常使用，我们需要通过 NTP 服务确保系统时间与互联网时间同步。

## 设置本地时钟和时区

​通过网络时间协议（NTP）从互联网上的公共 NTP 服务器上获取正确的时间信息是确保本地系统时间正确的一种标准方法。

​使用 `timedatectl` 命令可以简要的显示出当前的时间相关的系统设置，如系统的当前时间、时区和 NTP 同步设置。

```bash
[root@localhost ~]\# timedatectl
               Local time: Tue 2021-08-24 18:30:15 PDT
           Universal time: Wed 2021-08-25 01:30:15 UTC
                 RTC time: Wed 2021-08-25 01:30:15
                Time zone: America/Los_Angeles (PDT, -0700)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

​查看系统时区数据库的内容：

```bash
[root@localhost ~]\# timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
......
```

​若不知道自己该使用的时区配置信息，则可以通过 `tzselect` 命令来交互式的推断出适合自己的时区配置信息。该命令并不会对系统的时区设置进行任何更改。

​更改当前时区需要 root 权限，可以使用 `timedatectl set-timezone` 命令更改系统设置更新当前时区。

```bash
[root@localhost ~]\# timedatectl set-timezone Asia/Shanghai
[root@localhost ~]\# timedatectl
               Local time: Wed 2021-08-25 09:47:26 CST
           Universal time: Wed 2021-08-25 01:47:26 UTC
                 RTC time: Wed 2021-08-25 01:47:25
                Time zone: Asia/Shanghai (CST, +0800)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

> 🟢 若要使用 UTC 可以使用 `timedatectl set-timezone UTC` 命令将系统当前时区设置为 UTC。

​开启或禁用 NTP 服务：

```bash
[root@localhost ~]\# timedatectl set-ntp false # 关闭 NTP，true 为开启
```

## 配置和监控 CHRONYD

本地 linux 系统通过 chronyd 服务与配置的 NTP 服务器同步来保证不准确的本地硬件时钟（RTC）保持正确运行。若没有可用的网络连接，chronyd 将计算 RTC 时钟漂移，记录在 /etc/chrony.conf 配置文件指定的 *driftfile* 变量中。

​默认情况下不需要配置 chronyd 服务使用的授时服务器配置，但当本地系统处于孤立网络中时，就可能需要更改 NTP 服务器。

​NTP 时间源的 *stratum* 决定其质量。stratum 确定计算机与高性能参考时钟偏离的跃点数。参考时钟是 stratum 0 的时间源。与之直接关联的 NTP 服务器是 stratum 1，而与该 NTP 服务器同步时间的计算机则是 stratum 2 时间源。

​在 /etc/chrony.conf 配置文件中我们可以配置从两种时间源类别进行同步，它们分别是 server 和 peer。Server 比本地 NTP 服务器高一个级别，而 peer 则属于同一级别。可以指定多个 server 和多个 peer，每行指定一个。

​添加一个授时服务器的具体格式如下：

```ini
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
# ......
server ip或者域名 选项[常用 iburst]
```

> 🟢 修改 *chronyd* 配置文件后应该重新启动该服务，该服务不支持 reload。

​`chronyc` 命令充当 chronyd 服务的客户端。设置好 NTP 同步后，应该使用 `chronyc sources` 命令来验证本地系统是否使用 NTP 服务器无缝同步系统时钟，使用 *-v* 选项可以获得更加详细的输出。

```bash
[root@localhost ~]\# chronyc sources -v
210 Number of sources = 4

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^+ time.cloudflare.com           3   6   377    35  -8470us[  -12ms] +/-  139ms
^- time.cloudflare.com           3   6    37    34  -6035us[-9624us] +/-  128ms
^* 139.199.215.251               2   6   173    32    -16ms[  -20ms] +/-   78ms
^- ntp7.flashdance.cx            2   6   162   162  -7138us[  -13ms] +/-  163ms
``
