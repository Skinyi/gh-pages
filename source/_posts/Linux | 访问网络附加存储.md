---
title: 访问网络附加存储
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

## 挂载和卸载 NFS 共享

​NFS（网络文件系统）是由Linux、UNIX及类似操作系统使用的互联网标准协议，可作为它们的本地网络文件系统。它是一种仍在积极增强的开放标准，可支持本地Linux权限和文件系统功能。

​红帽企业 Linux 8 中的默认 NFS 版本为 4.2 。支持NFSV4和NFSV3的主要版本。NFSV2 已不再受支持。NFSV4 仅使用TCP 协议与服务器进行通信；较早的 NFS 版本可使用 TCP 或 UDP。

​NFS 服务器导出共享（目录）。NFS 客户端将导出的共享挂载到本地挂载点（目录），该挂载点必须存在。可以通过多种方式挂载 NFS 共享：

- 使用 `mount` 命令手动挂载；
- 使用 */etc/fstab* 条目在启动时自动挂载；
- 按需挂载：使用 *autofs* 服务或 *systemd.automount* 功能。

### 挂载 NFS 共享

​要挂载 NFS 共享，可以执行以下三个步骤：

1. <b>识别</b>：NFS 客户端系统的管理员可以通过各种方式识别可用的 NFS 共享

NFS 服务器的管理员可以提供导出详细信息，包括安全性要求。

或者，客户端管理员可以通过挂载 NFS 服务器的根目录并浏览已导出目录来识别 NFSV4 共享。以 root 用户身份执行该操作。对使用 Kerberos 安全性的共享的访问将被拒绝，但共享（目录）名称仍可见。可以浏览其他共享目录。

```bash
[user@localhost ~]\$ sudo mkdir mountpoint
[user@localhost ~]\$ sudo mount serverb:/ mountpoint
[user@localhost ~]\$ sudo ls mount point
```

2. <b>挂载点</b>：使用 `mkdir` 在合适的位置创建挂载点

```bash
[user@localhost ~]\$ mkdir -p mountpoint 
```

3. <b>挂载</b>：与分区上的文件系统一样，NFS 共享必须先进性挂载才可用。要挂载 NFS 共享，可以从以下选项中进行选择。无论在哪种情况下，都必须作为 *root* 用户登陆，或使用 `sudo` 命令，以超级用户身份运行这些命令。

- <i>临时挂载</i>：使用 `mount` 命令挂载 NFS 共享：

```bash
[user@localhost ~]\$ sudo mount -t nfs -o rw,sync serverb:/share mountpoint
```

​*-t nfs* 选项是 NFS 共享的文件系统类型（未严格要求，为完整性而显示）。*-o sync* 选项使 `mount` 立即与 NFS 服务器同步写操作（默认值为异步）。

​此命令将立即但并不持久挂载共享；下一次系统启动时，此NFS共享将不可用。对于一次性访问数据的情况，该选项很有用。它也可用于在持久提供共享之前对挂载NFS共享进行测试。

- <i>持久挂载</i>：为了确保在启动时挂载 NFS 共享，可以编辑 */etc/fstab* 文件来添加挂载条目。

```bash
[user@localhost ~]\$ sudo vim /etc/fstab
# 以下是 /etc/fstab 文件条目
serverb:/share /mountpoint nfs rw,soft 0 0
```

​接着，挂载 NFS 共享：

```bash
[user@localhost ~]\$ sudo mount /mountpoint
```

​由于 NFS 客户端服务会在 /etc/fstab 文件中查找 NFS 服务器和挂载选项，因此您无需在命令行中指定这些内容。

### 卸载 NFS 共享

​以 root 用户身份（或使用 `sudo`），使用 `umount` 命令卸载 NFS 共享。

```bash
[user@localhost ~]\$ sudo umount mountpoint
```

> 卸载共享不会删除它的 */etc/fstab* 条目。除非删除或注释掉该条目，否则在下一次系统启动或重启 NFS 客户端服务时将重新挂载 NFS 共享。

## NFSCONF 工具

​RHEL8 引入了 `nfsconf` 工具，用于管理 NFSv4 与 NFSv3 下的 NFS 客户端和服务器配置文件。使用 /etc/nfs.conf配置 nfsconf 工具（早期版本的操作系统中的 /etc/sysconfig/nfs文件现已被弃用）。使用 nfsconf 工具来获取、设置或取消设置NFS配置参数。

​*/etc/nfs.conf* 配置文件由多个部分组成，它的开头是一个位于方括号中的关键字（<b>[keyword]</b>）以及这一部分的值分配。对于 NFS 服务器，请配置 *[nfsd]* 部分。值的分配（或键）由值的名称、等号和值的设置组成，例如 *vers4.2=y*。以 “#” 或 “;” 开头的行将被忽略，就像空白行一样。

```bash
[user@localhost ~]\$ sudo cat /etc/nfs.conf
```

```ini
[nfsd]
# debug=0
# threads=0
# host=
# port=0
# grace-time=90
# lease-time=90
# tcp=y
# vers2=n
# vers3=y
# vers4=y
# vers4.0=y
# vers4.1=y
# vers4.2=y
# rdma=n
# ......
```

​默认情况下，<b>[nfsd]</b> 部分的键值对会被注释掉。不过，注释中会显示在不做更改的情况下将会生效的默认选项。这为配置 NFS 提供了很好的起点。

​使用 `nfsconf --set section key value` 来设置指定部分的键值。

```bash
[user@localhost ~]\$ sudo nfsconf --set nfsd vers4.2 y
```

​此命令将更新 */etc/nfs.conf* 配置文件：

```bash
[user@localhost ~]\$ sudo cat /etc/nfs.conf
```

```ini
[nfsd]
# ......
vers4.2 = y
# ......
```

​使用 `nfsconf --get section key` 来检索指定部分的键值：

```bash
[user@localhost ~]\$ sudo nfsconf --get nfsd vers4.2
y
```

​使用 `nfsconf --unset section key` 来取消设置指定部分的键值：

```bash
[user@localhost ~]\$ sudo nfsconf --unset nfsd vers4.2
```

### 配置一个仅限使用 NFSv4 的客户端

​通过在 */etc/nfs.conf* 配置文件中设置以下值，可以配置一个仅限使用 NFSv4 的客户端。

​首先禁用 UDP 以及其他与 NFSv2 和 NFSv3 有关的键：

```bash
[user@localhost ~]\$ sudo nfsconf --set nfsd udp n
[user@localhost ~]\$ sudo nfsconf --set nfsd vers2 n
[user@localhost ~]\$ sudo nfsconf --set nfsd vers3 n
```

​启用 TCP 和 NFSv4 相关键：

```bash
[user@localhost ~]\$ sudo nfsconf --set nfsd tcp y
[user@localhost ~]\$ sudo nfsconf --set nfsd vers4 y
[user@localhost ~]\$ sudo nfsconf --set nfsd vers4.0 y
[user@localhost ~]\$ sudo nfsconf --set nfsd vers4.1 y
[user@localhost ~]\$ sudo nfsconf --set nfsd vers4.2 y
```

​和以前一样，*/etc/nfs.conf* 配置文件中会显示所做的更改：

```bash
[user@localhost ~]\$ cat /etc/nfs.conf
```

```ini
[nfsd]
udp = n
vers2 = n
vers3 = n
tcp = y
vers4 = y
vers4.0 = y
vers4.1 = y
vers4.2 = y
# ......
```

## 自动挂载网络附加存储

### 使用自动挂载器挂载 NFS 共享

​自动挂载器是一种服务（*autofs*），它可以 “根据需要” 自动挂载NFS共享，并将在不再使用 NFS 共享时自动卸载这些共享。

​<b>自动挂载服务的优势：</b>

- 用户无需具有root特权就可以运行 mount 和 umount 命令。
- 自动挂载器中配置的 NFS 共享可供计算机上的所有用户使用，受访问权限约束。
- NFS 共享不像 /etc/fstab 中的条目一样永久连接，从而可释放网络和系统资源。
- 自动挂载器在客户端配置，无需进行任何服务器端配置。
- 自动挂载器与 mount 命令使用相同的选项，包括安全性选项。
- 自动挂载器支持直接和间接挂载点映射，在挂载点位置方面提供了灵活性。
- autofs 可创建和删除间接挂载点，从而避免了手动管理。
- NFS 是默认的自动挂载器网络文件系统，但也可以自动挂载其他网络文件系统。
- autofs 是一种服务，其管理方式类似于其他系统服务。

### 配置自动挂载 NFS 共享

​配置自动挂载包含以下步骤：

1. 安装 *autofs* 软件包；

```bash
[user@localhost ~]\$ sudo yum install autofs
```

​此软件包包含使用自动挂载器挂载NFS共享所需的所有内容。

2. 向 */etc/auto.master.d* 添加一个主映射文件。此文件确定用于挂载点的基础目录，并确定用于创建自动挂载的映射文件；

```bash
[user@localhost ~]\$ sudo vim /etc/auto.master.d/demo.autofs 
```

​主映射文件的名称是任意的（尽管通常会有一定的含意），但为了让子系统能够识别，它必须以 *.autofs* 作为扩展名。可以在一个主映射文件中放置多个条目；或者可以创建多个主映射文件，且每个文件的条目都进行逻辑分组。

​在此例中，为简介映射的挂载添加主映射条目：

```bash
/shares		/etc/auto.demo
```

3. 创建映射文件。每个映射文件确定一组自动挂载的挂载点、挂载选项及挂载的源位置；

```bash
[user@localhost ~]\$ sudo vim /etc/auto.demo
```

​映射文件的命名规则是 */etc/auto.name*, 其中 name 反映了映射内容。

```bash
work	-rw,sync	serverb:/shares/work
```

​条目的格式为挂载点、挂载选项和源位置。此示例显示基本的间接映射条目。本节稍后部分将讨论直接映射和使用通配符的间接映射。

- 挂载点在 man page 被称为 “密钥”，它由 autofs 服务自动创建和删除。在此例中，完全限定挂载点是 /shares/work（请参阅主映射文件）。autofs 服务将根据需要创建和删除 /shares 目录和 /shares/work 目录。

在此例中，本地挂载点将镜像服务器的目录结构，但这不是必需的；本地挂载点可以随意命名。autofs 服务不会在客户端上强制执行特定的命名结构。

- 挂载选项以短划线字符 *-* 开头，并使用逗号分隔，不带空格。自动挂载时，可以使用相应的挂载选项来手动挂载文件系统。在此例中，自动挂载器将挂载具有<b>读/写访问权限</b>的共享内容（*rw*选项），并且在写入操作期间服务器会立即同步（*sync* 选项）。

有用的自动挂载器特定选项包括 *-fstype=* 和 *-strict*。使用 *fstype* 指定文件系统类型，如 nfs4 或 xfs；挂载文件系统时，使用 *strict* 可将错误视为严重。

- NFS 共享的源位置遵循 *host:/pathname* 模式；在此示例中为 <b>serverb:/shares/work</b>。<b>为了确保此次自动挂载成功，NFS 服务器 serverb 必须以读/写访问权限导出目录，而请求访问的用户必须对该目录具有标准 Linux 文件权限。如果 serverb 以只读访问权限导出目录，那么客户端也仅获得只读访问权限，即使它要求读/写访问权限</b>。

 4. 启动并启用自动挂载器服务；

使用 `systemctl` 启动并启用 *autofs* 服务。

```bash
[user@localhost ~]\$ sudo systemctl enable --now autofs
Created symlink /etc/systemd/system/multi-user.target.wants/autofs.service -> /usr/lib/systemd/system/autofs.service.
```

### 直接映射

​直接映射用于将NFS共享映射到现有的绝对路径挂载点。

​要使用直接映射的挂载点，主映射文件可能如下所示：

```bash
/-		/etc/auto.direct
```

​所有直接映射条目都使用 */-* 作为基础目录。在此例中，包含挂载详细信息的映射文件是 */etc/auto,direct*。

​*/etc/auto.direct* 文件的内容可能如下所示：

```bash
/mnt/docs -rw,sync serverb:/shares/docs
```

​挂载点（或密钥）始终为绝对路径。映射文件的其余部分使用相同的结构。

​在此例中，存在 */mnt* 目录，它由 *autofs* 管理。*autofs* 服务将自动创建和删除整个 */mnt/docs* 目录。

### 间接通配符映射

​当NFS服务器导出一个目录中的多个子目录时，可将自动挂载程序配置为使用单个映射条目访问这些子目录其中的任何一个。

​继续前面的示例，如果 *serverb:/shares* 导出两个或多个子目录，并且能够使用相同的挂载选项访问这些子目录，则 */etc/auto.demo* 文件的内容可能如下所示：

```bash
*	-rw,sync	serverb:/shares/&
```

​挂载点（或密钥）是星号字符 *\**，而源位置上的子目录是 *\&* 符号。条日中的所有其他内容都相同。

​当用户尝试访问 */shares/work* 时，密钥 \*（此例中为 work ）将代替源位置中的 & 符号，并挂载 serverb:/shares/work。对于间接示例，autofs 将自动创建和删除 work 目录。
