---
title: Linux 配置网络设置
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

## NMCLI 命令摘要

| 命令                        | 用途                                             |
| --------------------------- | ------------------------------------------------ |
| nmcli dev status            | 显示所有网络接口的 NetworkManager 状态。         |
| nmcli con show              | 列出所有连接。                                   |
| nmcli con show con-name     | 列出 con-name 连接的当前设置。                   |
| nmcli con add con-name name | 添加一个名为 name 的新连接。                     |
| nmcli con mod con-name      | 修改 con-name 连接。                             |
| nmcli con reload            | 重新加载配置文件（再手动配置编辑文件之后使用）。 |
| nmcli con up con-name       | 激活 con-name 连接。                             |
| nmcli dev dis device        | 在网络接口上停用并断开当前连接。                 |
| nmcli con del con-name      | 删除 con-name 连接及其配置文件。                 |
| nmcli gen permissions       | 查看当前用户的网络修改权限。                     |

## 从命令行配置网络

### 了解 NETWORKMANAGER 守护进程

​NetworkManager 是监控和管理网络设置的守护进程。除了该守护进程外，还有一个提供网络状态信息的 GNOME 通知区域小程序。命令行和图形工具与 NetworkManager 通信，并将配置文件保存在 */etc/sysconfig/network-scripts* 目录中。NetworkManger 的基本概念：

- 设备是网络接口；
- 连接是可以为设备配置的设置的集合；
- 对于任何一个设备，在同一时间只能有一个连接处于活动状态。可能存在多个连接，以供不同设备使用或者以便为同一设备更改配置。如果需要临时更改网络设置，而不是更改连接的配置，可以通过更改设备的哪个连接处于活动状态；
- 每个连接具有一个用于标识自身的名称或 ID；
- *nmcli* 使用程序可用于从命令行创建和编辑连接文件。

### 查看联网信息

​使用`nmcli dev[ice] status` 命令可查看所有网络设备的状态：

```bash
[root@localhost ~]\# nmcli dev status
DEVICE      TYPE      STATE                   CONNECTION 
ens160      ethernet  connected               ens160     
virbr0      bridge    connected (externally)  virbr0     
lo          loopback  unmanaged               --         
virbr0-nic  tun       unmanaged               -- 
```

​使用 `nmcli con[nect] show` 命令可显示所有连接的列表。要仅列出活动的连接，可使用 *--active* 选项。

```bash
[root@localhost ~]\# nmcli con show --active
NAME    UUID                                  TYPE      DEVICE 
ens160  8fc2fbfe-e119-43be-87dd-c45071c07bf7  ethernet  ens160 
virbr0  139c458e-0cff-4615-974c-aa119ddf87c2  bridge    virbr0 
```

### 添加网络连接

​使用 `nmcli con[nect] add` 命令添加新的网络连接。

​以下命令将为接口 *eno2* 添加一个新链接 *eno2*，此连接将使用 DHCP 获取 IPv4 联网信息并在系统启动后自动连接。此命令还将通过侦听本地链路上的路由器播发来获取 IPv6 联网设置。配置文件的名称基于的 *con-name* 选项的值 *eno2*，并保存到 */etc/sysconfig/network-scripts/ifcfg-eno2* 文件中。

```bash
# nmcli connection add COMMON_OPTIONS TYPE_SPECIFIC_OPTIONS SLAVE_OPTIONS IP_OPTIONS [-- ([+|-]<setting>.<property> <value>)+]
[root@localhost ~]\# nmcli con add con-name eno2 type ethernet ifname eno2
```

​以下命令使用静态 IPv4 地址为 *eno2* 设备来创建 *eno2* 连接，且使用 IPv4 地址和网络前缀 192.168.0.5/24 及默认网关 *192.168.0.254*，但是仍在启动时自动连接并将其配置保存到相同文件中。

```bash
[root@localhost ~]\# nmcli con add con-name eno2 type ethernet ifname eno2 \
ipv4.address 192.168.0.5/24 ipv4.gateway 192.168.0.254
```

以下命令使用静态 IPv6 和 IPv4 地址为 *eno2* 设备创建 *eno2* 连接，且使用 IPv6 地址和网络前缀 *2001:db8:0:1::c000:207/64* 及默认 IPv6 网关 *2001:db8:0:1::1*，以及 IPv4 地址和网络前缀 *192.0.2.7/24* 及默认 IPv4 网关 *192.0.2.1*，但是仍在启动时自动连接，并将其配置保存到 */etc/sysconfig/network-scripts/ifcfg-eno2*。

```bash
[root@localhost ~]\# nmcli con add con-name eno2 type ethernet ifname eno2 \
ipv6.address 2001:db8:0:1::c000:207/64 ipv6.gateway 2001:db8:0:1::1 \
ipv4.address 192.0.2.7/24 ipv4.gateway 192.0.2.1
```

### 控制网络连接

​使用 `nmcli con[nect] up con-name` 命令将其绑定到的网络接口上的*连接 con-name* 激活。使用 `nmcli con show` 命令显示所有可用连接的名称。

```bash
[root@localhost ~]\# nmcli con show
NAME    UUID                                  TYPE      DEVICE 
ens160  8fc2fbfe-e119-43be-87dd-c45071c07bf7  ethernet  ens160 
virbr0  139c458e-0cff-4615-974c-aa119ddf87c2  bridge    virbr0 
[root@localhost ~]\# nmcli con up ens160
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/4)
```

​使用 `nmcli dev dis[connect] device` 命令将断开与网络接口 device 的连接并将其关闭。

```bash
[root@localhost ~]\# nmcli dev dis virbr0
Device 'virbr0' successfully disconnected.
```

> 使用 `nmcli con[nection] down con-name` 虽然可以停用网络接口，但是大部分有线连接在默认情况下配置了 *autoconnect*，使用该命令停用网络接口后 *NetworkManager* 会立即将其重新开启。使用 `nmcli dev dis[connect] device` 命令将连接与接口完全断开会避免这个问题。

### 修改网络连接设置

​ *NetworkManager* 连接具有两种类型的设置。有静态连接属性，它们是由管理员配置并存储在 */etc/sysconfig/network-scripts/ifcfg-\**中的配置文件中。还可能有活动连接数据，这些数据是连接从 DHCP 服务器获取的，不会持久存储。

​使用 `nmcli con[nect] show con-name` 命令来查看某个连接的当前设置，其中 *con-name* 是连接的名称。小写的设置是静态属性，管理员可以更改全大写的设置是活动设置，临时用于此连接实例。

```bash
[root@localhost ~]\# nmcli con show ens160
connection.id:                          ens160
connection.uuid:                        8fc2fbfe-e119-43be-87dd-c45071c07bf7
connection.stable-id:                   --
connection.type:                        802-3-ethernet
connection.interface-name:              ens160
connection.autoconnect:                 yes
connection.autoconnect-priority:        0
connection.autoconnect-retries:         -1 (default)
connection.multi-connect:               0 (default)
connection.auth-retries:                -1
connection.timestamp:                   1630289327
connection.read-only:                   no
......
```

​使用 `nmcli con[nect] mod con-name` 命令更改连接的设置。这些更改将保存在连接的 */etc/sysconfig/netwrok-scripts/ifcfg-\** 文件中。

​以下命令针对 *static-ens3* 连接将 IPv4 地址设置为 *192.0.2.154* 并将默认网关设置为 *192.0.2.254*：

```bash
[root@localhost ~]\# nmcli con mod static-ens3 ipv4.address 192.0.2.154 \
ipv4.gateway 192.0.2.254
```

​以下命令针对 *static-ens3* 连接将 IPv6 地址设置为 *2001:db8:0:1::a00:1/64* 并将默认网关设置为 *2001:db8:0:1::1*。

```bash
[root@localhost ~]\# nmcli con mod static-ens3 ipv6.address 2001:db8:0:1::a00:1/64 \
ipv6.gateway 2001:db8:0:1::1
```

>之前采用 DHCPv4 服务获取 IPv4 信息的连接改为通过静态文件来获取的话需要将 *ipv4.method* 字段从 *auto* 更改为 *manual*。
>
>之前采用 SLAAC 或 DHCPv6 服务获取 IPv6 信息的连接改为通过静态文件来获取的话需要将 *ipv6.method* 字段从 *auto* 或 *dhcp* 更改为 *manual*。

### 删除网络连接

​使用 `nmcli con[nect] del con-name` 删除名为 *con-name* 的连接，同时断开它与设备的连接并删除文件 */etc/sysconfig/network-scripts/ifcfg-con-name*。

```bash
[root@localhost ~]\# nmcli con del static-ens3
```

### 网络设置修改权限说明

​root 用户可以使用 *nmcli* 对网络配置进行任何必要的更改。除此之外，普通用户：

- 在本地控制台上登陆的普通用户也可以对系统进行多项网络配置更改（必需本地登录）；

- 使用 *ssh* 登陆无权更改网络权限（除非登陆 root）。

使用 `nmcli gen[eral] permissions` 命令来查看自己的当前权限。

```bash
[student@localhost ~]\$ nmcli gen permissions # 普通用户 ssh 登陆
PERMISSION                                                        VALUE 
org.freedesktop.NetworkManager.checkpoint-rollback                auth  
org.freedesktop.NetworkManager.enable-disable-connectivity-check  no    
org.freedesktop.NetworkManager.enable-disable-network             no    
org.freedesktop.NetworkManager.enable-disable-statistics          no    
org.freedesktop.NetworkManager.enable-disable-wifi                no    
org.freedesktop.NetworkManager.enable-disable-wimax               no    
org.freedesktop.NetworkManager.enable-disable-wwan                no    
org.freedesktop.NetworkManager.network-control                    auth  
org.freedesktop.NetworkManager.reload                             auth  
org.freedesktop.NetworkManager.settings.modify.global-dns         auth  
org.freedesktop.NetworkManager.settings.modify.hostname           auth  
org.freedesktop.NetworkManager.settings.modify.own                auth  
org.freedesktop.NetworkManager.settings.modify.system             auth  
org.freedesktop.NetworkManager.sleep-wake                         no    
org.freedesktop.NetworkManager.wifi.scan                          auth  
org.freedesktop.NetworkManager.wifi.share.open                    no    
org.freedesktop.NetworkManager.wifi.share.protected               no    
```

## 编辑网络配置文件

​除了通过 `nmcli con mod con-name` 命令更改网络配置 */etc/sysconfig/network-scripts/ifcfg-con-name*来实现对网络配置的修改，我们还可以使用文本编辑器来手动编辑此文件。手动更改该文件后，执行 `nmcli con reload` 命令来使 NetworkManager 进程重新读取该配置文件。

​nmcli 设置和 ifcfg 文件中的名称和语法不同，下表整理了部分常见配置的区别。

| NMCLI CON MOD      | IFCFG-\* 文件  | 作用           |
| ------------- | ------------------ | -------------- |
| ipv4.method manual | BOOTPROTO=none | IPv4 静态配置 |
| ipv4.method auto | BOOTPROTO=dhcp | IPv4 动态获取地址，若配置了静态设置，则 DHCPv4 获取成功之前不会激活这些静态设置。 |
| ipv4.address "192.168.2.1/24 192.0.2.254" | IPADDR0=192.0.2.1 PREFIX0=24 GATEWAY=192.0.2.254 | 设置静态 IPv4 地址、网络前缀和默认网关。如果为连接设置了多个，则 ifcog-\* 指令将以 1、2、3 等结尾，而不是以 0 结尾。 |
| ipv4.dns 8.8.8.8 | DNS0=8.8.8.8 | 修改 /etc/resolve.conf 以使用此域名服务器。 |
| ipv4.dns-search example.com | DOMAIN=example.com | 修改 /etc/resolv.conf，以在 search 指令中使用这个域。 |
| ipv4.ignore-auto-dns true | PEERDNS=no | 忽略来自 DHCP 服务器的 DNS 服务器信息。 |
| ipv6.method manual | IPV6_AUTOCONF=no | IPv6 静态配置。 |
| ipv6.method auto | IPV6_AUTOCONF=yes | 使用路由器播发中的 SLAAC 来配置网络设置。 |
| ipv6.method dhcp | IPV6_AUTOCONF=no DHCPV6C=yes | 使用 DHCPv6 （而不使用 SLAAC）来配置网络设置。 |
| ipv6.addresses "2001:db8::a/64 2001:db8::1" | IPV6ADDR=2001:db8::a/64 IPV6_DEFAULTGW=2001:db8::1 | 设置静态 IPv6 地址、网络前缀默认网关。如果为连接设置了多个地址，IPV6_SECONDARIES 将采用空格分割的地址/前缀定义的双引号列表。 |
| ipv6.dns 2400:3200::1 | DNS0=2400:3200::1 | 修改 /etc/resolv.conf 以使用此域名服务器。与 IPv4 完全相同。 |
| ipv6.dns-search example.com | DOMAIN=example.com | 修改 /etc/resolv.conf，以在 search 指令中使用这个域。与 IPv4 完全相同。 |
| ipv6.ignore-auto-dns true | IPV6_PEERDNS=no | 忽略来自 DHCP 服务器的 DNS 服务器消息。 |
| connection.autoconnection yes | ONBOOT=yes | 在系统引导时自动激活此连接。 |
| connection.id ens3 | NAME=ens3 | 此连接的名称。 |
| connection.interface-name ens3 | DEVICE=ens3 | 连接与具有此名称的网络接口绑定。 |
| 802-3-ethernet.mac-address ... | HWADDR=... | 连接与具有此 MAC 地址的网络接口绑定。 |

​以下是我的虚拟机的网络配置文件内容：

```ini
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens160
UUID=8fc2fbfe-e119-43be-87dd-c45071c07bf7
DEVICE=ens160
ONBOOT=yes
```

​修改了配置文件后，使用以下命令来使配置生效。

```bash
[root@localhost ~]\# nmcli con reload
[root@localhost ~]\# nmcli con down "con-name"
[root@localhost ~]\# nmcli con up "con-name"
```

## 配置主机名和域名解析

### 更改系统主机名

​使用 `hostname` 命令显示或临时修改系统的完全限定主机名。

```bash
[root@localhost ~]\# hostname # 不跟选项或参数为显示主机名
localhost.localdomain
[root@localhost ~]\# hostname skinyi.mydomain # 后跟内容则临时修改主机名
[root@localhost ~]\# hostname 
skinyi.mydomain
```

​使用 `hostnamectl` 命令修改配置文件 */etc/hostname* 来实现永久更改主机名，该命令也可用于查看系统的完全限定主机名的状态。

```bash
[root@localhost ~]\# hostnamectl set-hostname vmrhel@skinyi.me
[root@localhost ~]\# hostnamectl status
   Static hostname: vmrhelskinyi.me
   Pretty hostname: vmrhel@skinyi.me
         Icon name: computer-vm
           Chassis: vm
        Machine ID: 6651ee1da4824a37975cd5245c8cb076
           Boot ID: 95270889df844cc2911c26503ef2b0df
    Virtualization: vmware
  Operating System: Red Hat Enterprise Linux 8.3 (Ootpa)
       CPE OS Name: cpe:/o:redhat:enterprise_linux:8.3:GA
            Kernel: Linux 4.18.0-240.el8.x86_64
      Architecture: x86-64
[root@localhost ~]\# cat /etc/hostname
vmrhelskinyi.me
```

### 配置域名解析

​根解析器用于将域名转换为 IP 地址，反之亦可。他将根据 */etc/nsswitch.conf* 文件的配置来确定查找位置。默认情况下，先检查 */etc/hosts* 文件的内容。

​使用 `getent hosts hostname` 命令可以模拟在 */etc/hosts* 文件中查找域名记录的过程（关于 `getent` 命令的详细介绍见 *09. 管理本地用户和组* 一章）：

```bash
[root@localhost ~]\# getent hosts localhost  # DNS 反查
::1             localhost localhost.localdomain localhost6 localhost6.localdomain6
```

​若在本地 hosts 文件中找不到相关记录，默认情况下，根解析器会尝试使用 DNS 服务来查询主机名。*/etc/resolv.conf* 配置文件控制如何进行查询：

- search：对于较短的主机名尝试搜索的域名列表。不应该在同一文件中设置此参数和 domain，如果在同一文件中设置它们，最后一项设置将覆盖另外一项；
- nameserver：要查询的域名服务器的 IP 地址。可以指定最多三个域名服务器指令，以在其中一个域名服务器停机时提供备用域名服务器。

```bash
[root@vmrhelskinyi ~]\# cat /etc/resolv.conf
# Generated by NetworkManager
search me
nameserver 192.168.251.86
nameserver 240e:41d:8010:c75::6f
nameserver 240e:41d:8010:c75::b8
```

​向连接添加 DNS 服务器地址配置：

```bash
[root@vmrhelskinyi ~]\# nmcli con mod CON-NAME +ipv4.dns IP # 向 CON-NAME 连接增加一条 DNS 配置记录 
```

​NetworkManager 会根据连接配置文件更新 */etc/resolv.conf* 文件。

​使用 `host` 命令测试域名解析：

 ```bash
 [root@vmrhelskinyi ~]\# host kernel.org	# 测试域名解析
 kernel.org has address 198.145.29.83
 kernel.org mail is handled by 10 mail.kernel.org.
 [root@vmrhelskinyi ~]\# host 198.145.29.83 # 测试域名反查
 83.29.145.198.in-addr.arpa domain name pointer kernel.org.
 ```

> DHCP 设置会在接口启动时自动根据获取的信息重写 /etc/resolv.conf 文件，如果不想使用 DHCP 更新 DNS 服务器设置，可以在接口配置文件中配置：PEERDNS=no。使用 `nmcli` 命令设置如下：
>
> ```bash
> [root@vmrhelskinyi ~]\# nmcli con mod CON_NAME ipv4.ignore-auto-dns yes 
> ```
