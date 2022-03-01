---
title: RHCE 考试及红帽企业 Linux 了解
lang: zh-CN
tags: 
  - Linux
  - RHEL
  - 认证考试
categories: 
  - [学习, Linux]
  - [日常记录]
  - [迁移文章]
---
> 🔴 此文章由之前的 Typora 笔记迁移过来，内容可能已经过时。

## RHCE8.0 课程及考试代码

相较于 RHCE7 ，在 EX200 考试中，RHCE8 增加了 SHELL 脚本（RH254）、系统调优（RH442）以及 RHEL8 新特性的一些考察；在 EX294 考试中，RHCE7 考察一些常见的 Service 的搭建部署使用，如：NFS、SAMBA、DNS等，而在 RHCE8 考试中则主要考察 Ansible 自动化工具的操作，不再包含常见服务。

<table>
    <tr>
       <th>课程代码</th>
       <th>考试代码</th>
       <th>考试内容</th>
       <th>考试形式</th>
       <th>考试时长</th>
    </tr>
    <tr>
        <td>RH124</td>
        <td rowspan="2">EX200</td>
        <td rowspan="2">以系统管理操作为主，如：文件系统、用户操作、权限操作、磁盘操作等</td>
        <td rowspan="3">机考实验</td>
        <td rowspan="2">2.5h</td>
    </tr>
    <tr>
        <td>RH134</td>
    </tr>
    <tr>
        <td>RH294</td>
        <td>EX294</td>
        <td>全是关于 Ansible 自动化工具的技能操作</td>
        <td>4h</td>
    </tr>
</table>

## 搭建日常练习环境

> 🟢 所需前期准备工作：
>
> - 已安装 VMWare Workstation 虚拟机，并在 bios 中开启了虚拟化相关的开关；
> - 已下载好 RHEL8 Linux 操作系统 iso 镜像；
> - 练习环境虚拟机以 NAT 模式和宿主机进行网络连接，使用的虚拟网卡是 VMNet8 。

### 虚拟机安装 RHEL8.3 操作系统

​如图，虚拟机的配置如下，可根据自己个人硬件设备情况按需调整。

![练习环境虚拟机配置](images/linux/RHEL8.3-练习环境虚拟机配置.png)

虚拟机软件选择如下：

![安装软件选择](images/linux/安装软件选择.png)

设置 root 用户和普通用户后开始安装，等待进度条跑完重启虚拟机，重启后同意 Redhat 的 EULA，选择 *FINISH CONFIGURATION* 进入图形界面。图形界面默认为刚才创建的普通用户，新安装的 RHEL 还需要进行一些系统级的配置才行。

在此之前我们需要登陆 root 账户来配置我们新安装的操作系统。下图为使用 root 用户登陆后的界面。

![Gnome 欢迎界面](images/linux/Gnome-欢迎界面.png)

### 安装完操作系统后的配置

#### 配置网络设置

​使用 `ip addr show` 命令来查看操作系统的网络配置结果如图所示：

![查看操作系统网络配置](images/linux/查看网络配置.png)

> 🟢 其它网卡设备介绍：
> 
> - lo 虚拟网卡设备， lo 是主机用于向自身发送通信的一个特殊地址（也就是一个特殊的目的地址），其 ip 为 127.0.0.1 ；
> - virbr0 是 KVM 默认创建的一个网桥，其作用是为连接其上的虚拟机网卡提供 NAT 访问外网的功能，可提供 DHCP 服务，其默认 ip 为 192.168.122.1。此虚拟设备被强制删除后重启系统还会再次创建，如需卸载须使用 KVM 虚拟化管理工具；
> - birbr0-nic, 同上由 KVM 服务创建，可以使用 `brctl` 命令进行管理。

其中 _ens160_ 才是连接外网所需要的网卡设备。可以看到此网卡没有绑定 ip 地址，但虚拟机软件是设置的 NAT 且已开启了 DHCP 服务，因此导致此问题的原因可能是我们的网卡设备没有启动或启用。需要检查此网卡设备的配置文件，其路径为：`/etc/sysconfig/network-scripts/ifcfg-ens160`（可以推断 RHEL 的网卡配置文件名称都是由 "ifcfg-" 和网卡名组成的），使用 `vim` 编辑器来编辑此文件，此文件内容如下：

```conf
TYPE=Ethernet								# 网络类型：以太网
PROXY_METHOD=none							# 网络代理方法
BROWSER_ONLY=no	
BOOTPROTO=dhcp								# 激活设备使用的地址配置协议：dhcp,static,none,bootp
DEFROUTE=yes								# ipv4 默认路由设备：yes,no
IPV4_FAILURE_FATAL=no
IPV6INIT=yes								# 初始化 ipv6 协议栈：yes,no
IPV6_AUTOCONF=yes							# 自动配置 ipv6：yes,no
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens160									# 网卡名
UUID=74e066fb-4f82-4422-ae3f-570dd2fca5a5	
DEVICE=ens160								# 应用到的设备（和 HWADDR 必须留一个，后者指网卡物理地址）
ONBOOT=no									# 开机后自动激活此设备
```

> 🟢 Linux 网络配置说明：
>
> * 网卡的相关配置文件：/etc/sysconfig/network-scripts/ifcfg-网卡名
> * 路由相关的配置文件：/etc/sysconfig/network-scripts/route-网卡名
> * 网络相关说明参考/usr/share/doc/initscripts-version/sysconfig.txt

需要将 `ONBOOT=no` 更改为 `ONBOOT=yes` 以使此网卡开机就激活启用，重启虚拟机，验证网卡是否已经自动激活（自动获得了 ip 地址则配置成功）。

![验证操作系统网络配置](images/linux/验证网络配置.png)

#### 使用 Xshell 远程登陆

​在 Xshell 里新建连接，进行配置即可远程登陆到此虚拟机。

![使用Xshell远程连接虚拟机](images/linux/使用-Xshell-进行远程登陆.png)

## RHEL 和 CentOS 的区别

|  简称  |        中文全称         | 存在付费 |   付费回报   |                    特点                     |
| :----: | :---------------------: | :------: | :----------: | :-----------------------------------------: |
|  RHEL  | 红帽企业 Linux 操作系统 |    是    | 获得技术支持 |           稳定，最先获得 Bug 修复           |
| CentOS |    社区企业操作系统     |    否    |      -       | 含有一些新特性，比较稳定，延迟获得 Bug 修复 |
