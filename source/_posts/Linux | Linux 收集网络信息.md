---
title: Linux 收集网络信息
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

## 收集网络接口信息

​`ip` 命令的各种形式后面指定具体对象则查询具体对象的信息，不指定则查询所有对象的信息。

### 识别网络接口

​使用 `ip link` 命令查看系统上所有可用的网络接口：

```bash
[root@localhost ~]\# ip link show # 该命令亦可简写为 `ip l`
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:02:75:df brd ff:ff:ff:ff:ff:ff
3: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:53:ea:42 brd ff:ff:ff:ff:ff:ff
4: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN mode DEFAULT group default qlen 1000
    link/ether 52:54:00:53:ea:42 brd ff:ff:ff:ff:ff:ff
```

​以上示例中，服务器包含两个物理网络接口和一个虚拟网络接口（该接口拥有相同的 MAC 地址但有两个不同的名字），其中 *lo* 接口为回环设备，*ens160* 为虚拟机模拟的物理网卡，*virbr0* 和 *virbr0-nic* 为安装了 *libvirt* 服务自动创建的供虚拟机连接的虚拟网卡，*virbr0* 绑定一个固定的 ip 地址：192.168.122.1/24。由每个接口的 MAC 地址和绑定的 ip 地址我们就知道了每个接口连接网络的状态。

### 查询 IP 地址

​使用 `ip addr` 命令来查看网络接口的 IPv4 或 IPv6 地址。

```bash
[root@localhost ~]\# ip addr show ens160 # 可简写为 ip a s ens160
2: ens160: <BROADCAST,MULTICAST,①UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
    ②link/ether 00:0c:29:02:75:df brd ff:ff:ff:ff:ff:ff
    ③inet 192.168.224.139/24 brd 192.168.224.255 scope global dynamic noprefixroute ens160
       valid_lft 1962sec preferred_lft 1962sec
    ④inet6 240e:41d:8010:c75:683c:cc7:17fe:4c47/64 scope global dynamic noprefixroute 
       valid_lft 3282sec preferred_lft 3282sec
    ⑤inet6 fe80::8af6:d265:9b34:f1d6/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

​其中，标注各项的意义如下：

- ① *UP* 代表这是一个活动接口，即*已启用*。
- ② *link/ether* 行指定设备的硬件（MAC）地址。
- ③ *inet* 行显示 IPv4 地址、其网络前缀长度和作用域。
- ④ *inet6* 行显示 IPv6 地址、其网络前缀长度和作用域。此地址属于全局作用域，通常使用此地址。
- ⑤ 该 *inet6* 行显示接口具有链路作用域的 IPv6 地址，并且只能用于本地以太网链路上的通信。

### 显示性能统计信息

​使用 `ip -s link` 命令统计接口的性能参数。每个网络接口的计数器【包括收到（RX）和传出（TX）的数据包数、数据包错误数、丢弃的数据包数】可用于检测网络问题的存在。

```bash
[root@localhost ~]\# ip -s link show ens160 # 无法简写
2: ens160: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP mode DEFAULT group default qlen 1000
    link/ether 00:0c:29:02:75:df brd ff:ff:ff:ff:ff:ff
    RX: bytes  packets  errors  dropped overrun mcast   
    2159684    2512     0       0       0       146     
    TX: bytes  packets  errors  dropped carrier collsns 
    157533     1630     0       0       0       0  
```

## 检查网络连通性

​`ping`（IPv4）或 `ping6`（IPv6）命令可用于测试当前主机到网络上的任意主机的连通性（前提是被测试的主机开启 *ICMP 回显功能*）。不同于 windows，linux 环境下该命令将持续运行，直到按下 **ctrl** + **c** 组合键结束该命令的进程（除非指定了限制发送软件包数的选项）。

```bash
[root@localhost ~]\# ping -c 2 baidu.com # 被测试主机可以使用其域名、IP 地址
PING baidu.com (220.181.38.148) 56(84) bytes of data.
64 bytes from 220.181.38.148 (220.181.38.148): icmp_seq=1 ttl=51 time=66.2 ms
64 bytes from 220.181.38.148 (220.181.38.148): icmp_seq=2 ttl=51 time=56.4 ms

--- baidu.com ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 2ms
rtt min/avg/max/mdev = 56.371/61.274/66.177/4.903 ms
[root@localhost ~]\# ping6 ff02::1 # ping 命令其实也可以兼容 IPv6 地址
PING ff02::1(ff02::1) 56 data bytes
64 bytes from fe80::8af6:d265:9b34:f1d6%ens160: icmp_seq=1 ttl=64 time=0.423 ms
64 bytes from fe80::f4ac:5eff:fe0b:c1d2%ens160: icmp_seq=1 ttl=64 time=2.91 ms (DUP!)
^C
--- ff02::1 ping statistics ---
1 packets transmitted, 1 received, +1 duplicates, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.423/1.668/2.913/1.245 ms
```

​当网络中存在不止一块网卡且连接不止一个网络时，使用 `ping` 命令测试本地链路地址和本地链路全节点多播组（ff02::1）时，必须使用作用域标识符来显示指定要使用的网络接口。若遗漏，则将显示错误 *connect: Invalid argument*。

​对 *ff02::1* 执行 *ping* 可能有助于找到本地网络上的其他 IPv6 节点。如上例中找打了具有 IPv6 本地链路地址  *fe80::f4ac:5eff:fe0b:c1d2* 的其他主机。

```bash
[root@localhost ~]\# ping fe80::f4ac:5eff:fe0b:c1d2%ens160
PING fe80::f4ac:5eff:fe0b:c1d2%ens160(fe80::f4ac:5eff:fe0b:c1d2%ens160) 56 data bytes
64 bytes from fe80::f4ac:5eff:fe0b:c1d2%ens160: icmp_seq=1 ttl=64 time=3.19 ms
64 bytes from fe80::f4ac:5eff:fe0b:c1d2%ens160: icmp_seq=2 ttl=64 time=4.11 ms
^C
--- fe80::f4ac:5eff:fe0b:c1d2%ens160 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 3ms
rtt min/avg/max/mdev = 3.186/3.645/4.105/0.463 ms

```

​IPv6 本地链路地址可以由同一链路上的其他主机使用。可以尝试使用 IPv6 本地链路地址 *ssh* 连接到你的虚拟机或同一网络下的其他主机。

## 路由故障排查

### 显示路由表

​使用 `ip route` 命令选项来显示路由信息。

```bash
[root@localhost ~]\# ip route
default via 192.168.224.44 dev ens160 proto dhcp metric 100 
192.168.122.0/24 dev virbr0 proto kernel scope link src 192.168.122.1 linkdown 
192.168.224.0/24 dev ens160 proto kernel scope link src 192.168.224.139 metric 100 
```

​以上命令仅显示了 IPv4 路由表。使用 `ip -6 route` 命令即可显示 IPv6 路由表。

```bash
[root@localhost ~]\# ip -6 route
::1 dev lo proto kernel metric 256 pref medium
240e:41d:8010:c75::/64 dev ens160 proto ra metric 100 pref medium
fe80::/64 dev ens160 proto kernel metric 100 pref medium
default via fe80::f4ac:5eff:fe0b:c1d2 dev ens160 proto ra metric 100 pref high
```

​一个网络接口同时具有两个 IPv6 地址，即一个对外网络全局地址和一个本地链路地址。

### 追踪流量采用的路由

​使用 `traceroute` 或 `tracepath` 命令来追踪网络流量通过多少个路由器来到达远程主机以及其采用的路径。可以通过这两个命令检查网络中的路由器配置是否存在问题。

​默认情况下，两个命令都使用 UDP 数据包来追踪数据路径，但是许多网络阻止 UDP 和 ICMP 流量。`traceroute` 命令拥有可以追踪 UDP（默认）、ICMP（*-I*）或 TCP（*-T*）数据包路径的选项。不过，默认情况下通常不安装 `traceroute` 命令。

```bash
[root@localhost ~]\# tracepath bilibili.com
 1?: [LOCALHOST]                      pmtu 1500
 1:  _gateway                                              2.647ms 
 1:  _gateway                                              2.826ms 
 2:  _gateway                                              3.126ms pmtu 1410
 2:  no reply
 3:  183.29.251.1                                         67.107ms 
 4:  no reply
 5:  183.29.0.5                                           51.596ms 
 ......
```

​`traceroute` 输出中的每一行表示数据包在来源和最终目标之间所经过的路由器或跃点，也提供了其他信息，如往返用时（RTT）和最大传输单元（MTU）大小中的任何变化等。*asymm* 表示流量使用了不同的（非对称）路由到达该路由器的流量和从该路由器返回。显示的路由器是用于出站流量的路由器，而不是返回流量。

​`tracepath6` 和 `tracepath -6` 命令等效于 IPv6 版本的 `tracepath` 和 `traceroute` 命令。

```bash
[root@localhost ~]\# tracepath6 fe80::70b8:51ff:fe39:4e7f # tracepath 命令也兼容 IPv6
 1?: [LOCALHOST]                        0.027ms pmtu 1410
 1:  fe80::%ens160                                         2.619ms rea
 1:  fe80::%ens160                                         5.508ms rea
     Resume: pmtu 1410 hops 1 back 1 
```

### 端口和服务故障排除

​TCP 服务使用套接字作为通信的端点，其由 IP 地址、协议和端口号组成。服务通常侦听标准端口，而客户端则使用随机的可用端口。*/etc/services* 文件中列出了标准端口的常用名称。

​`ss` 命令可用于显示套接字统计信息。`ss` 命令旨在替换 *net-tools* 软件包中所包含的较旧工具 `netstat`。

```bash
[root@localhost ~]\# ss -ta # -t 只显示 TCP 连接，-a 显示所有（监听中和已建立的）套接字 
State	Recv-Q  Send-Q                  Local Address:Port       Peer Address:Port
LISTEN	0       128                           0.0.0.0:sunrpc          0.0.0.0:*   
LISTEN  0       32                      192.168.122.1:domain          0.0.0.0:*   
LISTEN  0       128                           0.0.0.0:ssh             0.0.0.0:*
LISTEN  0       5                           127.0.0.1:ipp             0.0.0.0:*
LISTEN  0       128                         127.0.0.1:x11-ssh-offset  0.0.0.0:* 
LISTEN  0       128                              [::]:sunrpc             [::]:*
LISTEN  0       128                              [::]:ssh                [::]:* 
LISTEN  0       5                               [::1]:ipp                [::]:*
LISTEN  0       128                             [::1]:x11-ssh-offset     [::]:* 
ESTAB   0       0  [fe80::8af6:d265:9b34:f1d6]%ens160:ssh                       [fe80::cdbe:da6f:b9b7:a07d]:4499    
ESTAB   0       52 [fe80::8af6:d265:9b34:f1d6]%ens160:ssh                          [fe80::cdbe:da6f:b9b7:a07d]:14318
```

​`ss` 命令和 `netstat` 命令的常用选项：

| 选项    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| -n      | 显示接口和端口的编号，而不显示名称。                         |
| -t      | 显示 TCP 套接字。                                            |
| -u      | 显示 UDP 套接字。                                            |
| -l      | 仅显示侦听中的套接字。                                       |
| -a      | 显示所有（侦听中和已建立的）套接字。                         |
| -p      | 显示使用套接字的进程。                                       |
| -A inet | 对于 inet 地址，显示活动的连接（但不显示侦听套接字），也就是说，忽略本地 UNIX 域套接字。对于 `ss`，同时显示 IPv4 和 IPv6连接。对于 `netstat`，仅显示 IPv4 连接（`netstat -A inet6` 显示IPv6，`netstat -46` 则同时显示 IPv4 和 IPv6）。 |
