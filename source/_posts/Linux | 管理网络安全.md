---
title: 管理网络安全
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

## 管理服务器防火墙

### 防火墙原理概述

#### Netfilter

​Linux 内核中包含 *netfilter*，它是网络流量操作（如数据包过滤、网络地址转换和端口转换）的框架。通过在内核中实现拦截函数调用和消息的处理程序，*netfilter* 允许其他内核模块直接与内核的网络堆栈进行接口连接。防火墙软件使用一些 Hook 方法来注册过滤规则和数据包修改功能，以便对经过网络堆栈的每个数据包进行处理。在到达用户空间组件或应用之前，任何传入、传出或转发的网络数据包都可以通过编程方式来检查、修改、丢弃或路由。*netfilter* 是红帽企业 Linux 8 防火墙的主要组件。

#### Nftables

​Linux 内核中还包含 *nftables*，这是一个新的过滤器和数据包分类子系统，其增强了 *netfilter* 的部分代码，但仍保留了 *netfilter* 的架构，如网络堆栈 Hook、连接跟踪系统及日志记录功能。*nftables* 更新的优势在于更快的数据包处理、更快的规则集更新，以及以相同的规则同时处理 IPV4 和 IPV6。*nftables* 与原始 *netfilter* 之间的另一个主要区别是它们的接口。*Netfilter* 通过多个实用程序框架进行配置，其中包括 `iptables`、`ip6tables`、`arptables` 和 `ebtables`，这些框架现在已被弃用。*Nftables* 使用单个 `nft` 用户空间实用程序，通过一个接口来管理所有协议，由此消除了以往不同前端和多个 *netfilter* 接口引起的争用问题。

#### Firewalld

​*Firewalld* 是一个动态防火墙管理器，它是 *nftables* 框架的前端（使用 `nft` 命令）。在推出 *nftables* 之前，作为一种改进 *iptables* 服务的替代方案，*firewalld* 曾使用 `iptables` 命令来直接配置 *netfilter*。在 RHEL8 中，*firewalld* 仍然是推荐的前端，它使用 `nft` 来管理防火墙规则集。*Firewalld* 仍可以读取和管理 `iptables` 配置文件和规则集，并使用 <b>xtables-nft-multi</b> 将 `iptables` 对象直接转换为 *nftables* 规则和对象。对于 `nft` 转换过程无法正确处理现
有 `iptables` 规则集的复杂用例，您可以对 *firewalld* 进行配置，使之恢复为 `iptables` 后端，虽然强烈建议不要这样做。

​应用会使用 *D-Bus* 接口查询子系统。*firewalld* 子系统（可通过 *firewalld* RPM 软件包获得）未包含在最小安装中，但包含在基本安装中。借助 *firewalld*，可以将所有网络流量分为多个区域，从而简化防火墙管理。根据数据包源IP地址或传入网络接口等条件，流量将转入相应区域的防火墙规则。每个区域都有自己的端口和服务列表，它们处于打开或者关闭状态。

> 对于笔记本电脑或经常更改网络的其他计算机，可以使用 *NetworkManager* 自动设置连接的防火墙区域。这些区域使用适于特定连接的规则进行自定义。
>
> 当您经常在家庭、办公室和公共无线网络间切换时，此功能尤为实用。当连接到家庭网络和企业网络时，用户可能希望系统的*sshd* 服务可访问，但当用户连接到当地咖啡馆的公共无线网络时则不然。

​*Firewalld* 会检查进入系统的每个数据包的源地址。如果该源地址被分配给特定区域，则应用该区域的规则。如果该源地址未分配给某个区域，*firewalld* 就会将数据包与传入网络接口的区域相关联，并应用该区域的规则。如果出于某种原因，网络接口未与某个区域关联，则 *firewalld* 会将数据包与默认区域相关联。

​默认区域不是一个单独的区域，而是指代现有的区域。最初，*firewalld* 指定 public 区域为默认区域，并将 lo 回环接口映射至 trusted 区域。

​大多数区域会允许与特定端口和协议（例如 *631/udp*）或预定义服务（例如 *ssh* ）的列表匹配的流量通过防火墙。如果流量不与允许的端口和协议或服务匹配，则通常会被拒绝。（trusted 区域默认情况下允许所有流量，它是此规则的一个例外。）

#### 预定义区域

​*Firewalld* 上有一些预定义区域，可分别进行自定义。默认情况下，如果传入流量属于系统启动的通信的一部分，则所有区域都允许这些传入流量和所有传出流量。下表详细介绍了这些初始区域配置。

| 区域名称 | 默认配置                                                     |
| -------- | ------------------------------------------------------------ |
| trusted  | 允许所有传入流量。                                           |
| home     | 除非与传出流量相关，或与 *ssh*、*mdns*、*ipp-client*、*samba-client* 或 *dhcpv6-client* 预定义服务匹配，否则拒绝传入流量。 |
| internal | 除非与传出流量相关，或与 *ssh*、*mdns*、*ipp-client*、*samba-client* 或 *dhcpv6-client* 预定义服务匹配，否则拒绝传入流量。（一开始与 <b>home</b> 区域相同） |
| work     | 除非与传出流量相关，或与 *ssh*、*ipp-client* 或 *dhcpv6-client* 预定义服务匹配，否则拒绝传入流量。 |
| public   | 除非与传出流量相关，或与 *ssh* 或 *dhcpv6-client* 预定义服务匹配，否则拒绝传入流量。新添加的网络接口的默认区域。 |
| external | 除非与传出流量相关，或与 *ssh* 预定义服务匹配，否则拒绝传入流量。通过此区域转发的 IPV4 传出流量将进行伪装，以使其看起来像是来自传出网络接口的 IPV4 地址。 |
| dmz      | 除非与传出流量相关，或与 *ssh* 预定义服务匹配，否则拒绝传入流量。 |
| block    | 除非与传出流量相关，否则拒绝所有传入流量。                   |
| drop     | 除非与传出流量相关，否则丢弃所有传入流量（甚至不产生包含 ICMP 错误的响应）。 |

#### 预定义服务

​*Firewald* 上有一些预定义服务。这些服务定义可帮助您识别要配置的特定网络服务。例如，可指定预构建的 *samba-client* 服务来配置正确的端口和协议，而无需研究 *samba-client* 服务的相关端口。下表列出了初始防火墙区域配置中使用的预定义服务。

| 服务名称      | 配置                                                         |
| ------------- | ------------------------------------------------------------ |
| SSH           | 本地 SSH 服务器。到 22/tcp 的流量。                          |
| dhcpv6-client | 本地 DHCPv6 客户端。到 fe80::/64 IPV6 网络中 546/udp 的流量。 |
| ipp-client    | 本地 IPP 打印。到 631/udp 的流量。                           |
| samba-client  | 本地 Windows 文件和打印共享客户端。到 137/udp 和 138/udp 的流量。 |
| MDNS          | 多播 DNS（mDNS）本地链路名称解析。到 5353/udp 指向 224.0.0.251（IPV4）或 f02:fb（IPV6）多播地址的流量。 |

> 许多预定义服务都包含在 *firewalld* 软件包中。使用 `firewall-cmd --get-services` 可以列出这些预定义服务。预定义服务的配置文件位于 */usr/1ib/firewalld/services* 中，其格式由 firewalld.zone（5）定义。
>
> 可以使用预定义服务，或者直接指定所需的端口和协议。可以使用Web控制台图形界面来查看预定义服务和定义更多服务。

### 配置防火墙

​系统管理员可通过三种主要方式与firewalld交互：

- 直接编辑/etc/firewalld/中的配置文件（本章中不讨论）；

- Web控制台图形界面（略）；

- firewall-cmd命令行工具。

  本章重点关注从命令行配置防火墙。

#### 从命令行配置防火墙

​`firewall-cmd` 命令将会与 *firewalld* 动态防火墙管理器进行交互。它是作为主 firewalld 软件包的一部分安装的，可用于倾向于使用命令行的管理员，在没有图形环境的系统上工作，或编写有关防火墙设置的脚本。

​下表列出一些常用 `firewall-cmd` 命令及其说明。请注意，除非另有指定，否则几乎所有命令都作用于运行时配置，当指定*--permanent*选项时除外。如果指定了 *--permanent* 选项，还必须通过运行 `firewall-cmd -reload` 命令来激活设置，它将读取当前的永久配置并将其作为新的运行时配置来应用。列出的许多命令都采用 *--zone=ZONE* 选项来确定所影响的区域。如果需要子网拖码，请使用 <b>CIDR</b> 表示法，如 *192.168.1/24*。

| firewall-cmd 命令                         | 说明                                                         |
| ----------------------------------------- | ------------------------------------------------------------ |
| --get-default-zone                        | 查询当前默认区域。                                           |
| --set-default-zone=ZONE                   | 设置默认区域。此命令会同时更改运行时配置和永久配置。         |
| --get-zones                               | 列出所有可用区域。                                           |
| --get-active-zones                        | 列出当前正在使用的所有区域（具有关联的接口或源）及其接口和源信息。 |
| --add-source=CIDR [--zone=ZONE]           | 将来自 IP 地址或网络/子网掩码的所有流量路由到指定区域。如果未提供 *--zone=* 选项，则使用默认区域。 |
| --remove-source=CIDR [--zone=ZONE]        | 从区域中删除用于路由来自 IP 地址或网络/子网拖码的所有流量的规则。如果未提供 *-zone=* 选项，则使用默认区域。 |
| --add-interface=INTERFACE [--zone=ZONE]   | 将来自 INTERFACE 的所有流量路由到指定区域。如果未提供 *--zone=* 选项，则使用默认区域。 |
| --change-interface=INTERFACE [zone=ZONE]  | 将接口与 ZONE 而非其当前区域关联。如果未提供 *--zone=* 选项，则使用默认区域。 |
| --list-all [--zone=ZONE]                  | 列出 ZONE 的所有已配置接口、源、服务和端口。如果未提供 *--zone=* 选项，则使用默认区域。 |
| --list-all-zones                          | 检索所有区域的所有信息（接口、源、端口、服务）。             |
| --add-service=SERVICE [--zone=ZONE]       | 允许到 SERVICE 的流量。如果未提供 *--zone=* 选项，则使用默认区域。 |
| --add-port=PORT/PROTOCOL [--zone=ZONE]    | 允许到 PORT/PROTOCOL 端口的流量。如果未提供 *--zone=* 选项，则使用默认区域。 |
| --remoce-service=SERVICE [--zone=ZONE]    | 从区域的允许列表中删除 SERVICE。如果未提供 *--zone=* 选项，则使用默认区域。 |
| --remove-port=PORT/PROTOCOL [--zone=ZONE] | 从区域的允许列表中删除 PORT/PROTOCOL 端口。如果未提供 *--zone=* 选项，则使用默认区域。 |
| --reload                                  | 丢弃运行时配置并应用持久配置。                               |

​以下命令示例会将默认区域设置为 *dmz*，将来自 *192.168.9.0/24* 网络的所有流量都分配给 *internal* 区域，并在*internal* 区域上打开用于 *mysql* 服务的网络端口。

```bash
[root@localhost ~]\# firewall-cmd --set-default-zone=dmz
[root@localhost ~]\# firewall-cmd --permanent --zone=internal --add-source=192.168.0.0/24
[root@localhost ~]\# firewall-cmd --permanent --add-service=mysql
[root@localhost ~]\# firewall-cmd --reload
```

> 对于 *firewalld* 的基本语法不够的情况，还可以添加富规则（一种更具表达力的语法）来编写复杂的规则。如果富规则语法也不够，您还可以使用直接配置规则，这是与 *firewalld* 规则混合使用的原始 `nft` 语法。
> 这些高级模式超出了本章的讨论范围。

## 控制 SELinux 端口标记

### SELinux 端口标记概述

​SELinux 不仅仅是进行文件和进程标记。SELinux 策略还严格实施网络流量。SELinux 用来控制网络流量的其中一种方法是标记网络端口；例如，在 *targeted* 策略中，端口 *22/TCP* 具有标签 *ssh_port_t*与其相关联。默认 HTTP 端口 *80/TCP* 和 *443/TCP* 具有标签 *http_port_t* 与其相关联。当某个进程希望侦听端口时，SELinux将检查是否允许与该进程（域）相关联的标签绑定该端口标签。这可以阻止恶意服务控制本应由其他（合法）网络服务使用的端口。

### 管理 SELinux 端口标记

​如果决定在非标准端口上运行服务，SELinux 肯定会拦截此流量。在这种情况下，必须更新 SELinux 端口标签。在某些情况下，*targeted* 策略已经通过可以使用的类型标记了端口；例如，由于端口 *8008/TCP* 通常用于Web应用程序，此端口已使用*http_port_t*（Web 服务器的默认端口类型）进行标记。

### 列出端口标签

​要获取所有当前端口标签分配的概述，请运行 `semanage port -l` 命令。*-l* 选项将以下列形式列出所有当前分配：

```txt
port_label_t tcp|udp comma,separated,list,of,ports
```

​输出示例：

```bash
[root@localhost ~]\# semanage port -l
......
http_cache_port_t	tcp	8080, 8118, 8123, 10001-10010
http_cache_port_t	udp	8130
http_port_t			tcp	80, 81, 443, 488, 8008, 8009, 8443, 9000
......
```

​要优化搜索，可以使用 `grep` 命令：

```bash
[root@localhost ~]\# semanage port -l | grep ftp
ftp_data_port_t		tcp		20
ftp_port_t			tcp		21, 989, 990
ftp_port_t			udp		989, 990
tftp_port_t			udp		69
```

​一个端口标签可能会在输出中出现两次，一次是针对 TCP ，一次是针对 UDP。

### 添加端口标签

​使用 `semanage` 命令可以分配新端口标签、删除端口标签或修改现有端口标签。

>Linux 发行版中的大多数标准服务都提供了一个 SELinux 策略模块，用于在端口上设置标签。其中无法使用 `semanage` 来更改这些端口上的标签；要更改这些标签则需要替换策略模块。编写和生成策略模块在此节不涉及。

​要向现有端口标签（类型）中添加端口，请使用以下语法。*-a* 将添加新端口标签，*-t* 表示类型，*-p* 表示协议。

```bash
[root@localhost ~]\# semanage port -a -t port_label -p tcp|udp PORTNUMBER
```

​例如，要允许 gopher 服务侦听端口 *71/TCP*：

```bash
[root@localhost ~]\# semanage port -a -t gopher_port_t -p tcp 71
```

​要查看对默认策略的本地更改，管理员可以在 `semanage` 命令中添加 *-C* 选项。

```bash
[root@localhost ~]\# semanage port -l -C
SELinux Port Type	Proto	Port Number
......
gopher_port_t		tcp		71
```

> *targeted* 策略随附了大量端口类型。在 *selinux-policy-doc* 软件包中可以找到特定于服务的 SELinux man page，其中包含了有关 SELinux 类型、布尔值和端口类型的文档。如果系统上尚未安装这些 man page，可以避循以下步骤进行安装：
>
> ```bash
> [root@localhost ~]\# yum -y install selinux-policy-doc
> [root@localhost ~]\# man -k _selinux
> ```

### 删除端口标签

​删除自定义端口标签的语法与添加端口标签的语法相同，但不是使用 *-a* 选项（表示添加），而是使用 *-d* 选项（表示删除）。

​例如，要删除端口 <b>71/TCP</b> 与 *gopher_port_t* 的绑定：

```bash
[root@localhost ~]\# semanage port -d -t gopher_port_t -p tcp 71
```

### 修改端口标签

​要更改端口绑定（可能是因为需求发生改变），可以使用 *-m*（修改）选项。这种流程比删除旧绑定并添加新绑定更高效。

​例如，要将端口 <b>71/TCP</b> 从 *gopher_port_t* 修改为 *http_port_t*，管理员可以使用以下命令：

```bash
[root@localhost ~]\# semanage port -m -t http port_t -p tcp 71
```

​和以前一样，使用 `semanage` 命令可以查看修改内容。

```bash
[root@localhost ~]\# semanage port -l -C 
SELinux Port Type	Proto	Port Number
......
http_port_t			tcp		71
[root@localhost ~]\# semanage port -l | grep http
http_cache_port_t		tcp		8080，8118，8123，10001-10010
http_cache_port_t 		udp		3130
http_port_t 			tcp		71，80，81，443，488，8008，8009，8443，9000
pegasus_http_port_t		tcp		5988
pegasus_https_port_t 	tcp		5989
```
