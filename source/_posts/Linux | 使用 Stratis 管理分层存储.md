---
title: 使用 Stratis 管理分层存储
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

## STRATIS 架构概述

​RHEL 当前的本地存储解决方案中包含许多成熟、稳定的技术，其中包括设备映射器（dm）、逻辑卷管理器（LVM）及 XFS 文件系统。这些组件提供的功能包括可大规模扩展的文件系统、快照、冗余（RAID）逻辑设备、多路径、精简配置、缓存、重复数据删除，以及对虚拟机和容器的支持。每个存储堆栈层（dm、LVM 和 XFS）均使用专门面向层的命令和实用程序进行管理，这就要求系统管理员将物理设备、固定大小的卷和文件系统作为独立的存储组件进行管理。

​在 RHEL 8 中，红帽推出了 Stratis 存储管理解决方案。Straits 以管理物理存储设备池的服务形式运行，并透明地为所创建的文件系统创建和管理卷。由于 Stratis 使用现有的存储驱动程序和工具，因此 Stratis 也支持当前在 LVM、XFS 和设备映射器中使用的所有高级存储功能。

​在卷管理文件系统中，文件系统借助一个名为精简配置的概念内置于磁盘设备的共享池中。Straits 文件系统没有固定大小，也不再预分配未使用的快空间。尽管文件系统仍构建在隐藏的 LVM 卷上，但 Stratis 会管理基础卷，并可在需要时对其进行扩展。文件系统的 “使用中” 大小可视作所含文件占用的实际块数量。文件系统的可用空间就是它所驻留的池设备中仍未使用的空间量。多个文件系统可以驻留在同一磁盘设备池中，共享可用空间，但文件系统也可以保留池空间，以便在需要时保证可用性。

​Straitis 使用存储的元数据来识别所管理的池、卷和文件系统。因此，绝不应该对 Stratis 创建的文件系统进行手动重新格式化或重新配置；只应使用 Straitis 工具和命令对它们进行管理。手动配置 Stratis 文件系统可能会导致该元数据丢失，并阻止 Stratis 识别它已创建的文件系统。

​可以使用不同组的块设备来创建多个池。在每个池中，可以创建一个或多个文件系统。目前，每个池最多可以创建 2<sup>24</sup> 个文件系统。下图说明了 Stratis 存储管理解决方案的元素是如何定位的。

![Stratis 存储示例一](images/linux/Stratis-存储示例（一）.png)

​存储池可将块设备分组到数据层，或分组到缓存层。数据层侧重于灵活性和完整性，而缓存层则侧重于提高性能。由于缓存层旨在提高性能，因此应使用具有更高每秒输入、输出操作次数（IOPS）的块设备，如 SSD。

![Stratis 存储示例二](images/linux/Stratis-存储示例（二）.png)

​在内部，Stratis 使用 *Backstore* 子系统来管理块设备，并使用 Thinpool 子系统来管理池。*Backstore* 有一个数据层，负责维护块设备磁盘上的元数据，以及检测和纠正数据损坏。缓存层使用高性能块设备，作为数据层之上的缓存。*Thinpool* 子系统管理与 *Straitis* 文件系统关联的精简部署卷。该子系统使用 *dm-thin* 设备映射器驱动程序取代 LVM 进行虚拟卷大小调整和管理。*dm-thin* 可以创建虚拟大小比较大、采用 XFS 格式，但物理大小比较小的卷。当物理大小快要满时，Stratis 会自动将其扩大。

![Stratis 存储示例三](images/linux/Stratis-存储示例（三）.png)

## 管理精简配置的文件系统

​要使用 Stratis 存储管理解决方案来管理精简配置的文件系统，可以安装 stratis-cli 和 stratisd 软件包。stratis-cli 软件包中提供了 *stratis* 命令，它通过 *D-Bus* API 将用户请求转换为 *stratisd* 服务。*stratisd* 软件包中提供了 *stratisd* 服务，它实现 *D-Bus* 接口并管理和监控 Stratis 的元素，如块设备、池和文件系统。当 stratisd 服务处于运行状态时，可以使用 *D-Bus* API。

​使用常用工具安装并激活 Stratis：

1. 使用 `yum install` 命令安装 *stratis-cli* 和 *stratisd*；

```bash
[root@localhost ~]\# yum install stratis-cli stratisd
......
Is this ok [y/N]: y
......
complete!
```

2. 使用 `systemctl` 命令激活 *stratisd* 服务；

```bash
[root@localhost ~]\# systemctl enable --now stratisd
```

​使用 *stratis* 存储管理解决方案执行常见管理操作：

1. 使用 `stratis pool create` 命令来创建包含一个或多个块设备的池；

```bash\
[root@localhost ~]\# stratis pool create pool1 /dev/vdb
```

​每个池都是 */stratis* 目录下的一个子目录。使用 `stratis pool list` 命令查看可用池的列表：

```bash
[root@localhost ~]\# stratis pool list
Name	Total Physical Size		Total	Physical Used
pool1				  5 GiB					   52 MiB
```

2. 使用 `stratis pool add-data` 命令向池中添加额外的块设备；

```bash
[root@localhost ~]\# stratis pool add-data pool1 /dev/vdc
```

​使用 `stratis pool list` 命令查看可用池的块设备：

```bash
[root@localhost ~]\# stratis pool list pool1
Pool Name	Device Node		Physical Size		State	Tier
pool1		   /dev/vdb		  		5 GiB	   In-use	Data
pool1		   /dev/vdc				5 GiB	   In-use	Data
```

3. 使用 `stratis filesystem create` 命令为池创建动态、灵活的文件系统；

```bash
[root@localhost ~]\# stratis filesystem create pool1 filesystem1
```

​Stratis 文件系统的链接位于 */stratis/pool1* 目录中。

4. Stratis 支持通过 `stratis filesystem snapshot` 命令创建文件系统快照。快照独立于源文件系统。

```bash
[root@localhost ~]\# stratis filesystem snapshot pool1 filesystem1 snapshot1
```

5. 使用 `stratis filesystem list` 命令查看可用文件系统的列表。

```bash
[root@localhost ~]\# stratis filesystem list
......
```

​为了确保持久挂载 *Stratis* 文件系统，请编辑 */etc/fstab* 并指定文件系统的详细信息。以下命令显示文件系统 UUID，在 */etc/fstab* 中应使用该 UUID 来识别文件系统。 

```bash
[root@localhost ~]\# lsblk --output=UUID /stratis/pool1/filesystem1
UUID	31b9363b-add8-4b46-a4bf-c199cd478c55
```

​以下是 */etc/fstab* 文件中用于持久挂载 Stratis 文件系统的条目示例。

```bash
UUID=31b9363b-add8-4b46-a4bf-c199cd478c55	/dir1 xfs defaults,x-systemd.requires=stratisd.service 0 0
```

​*x-systemd.requires=stratisd.service* 挂载选项可延迟挂载文件系统，直到 systemd 在启动过程中启动 `stratisd.service` 为止。

>如果在 */etc/fstab* 中未对 stratis 文件系统使用 *x-systemd.requires=stratisd.service* 挂载选项，将会导致计算机在下一次重启时引导至 emergency.target。

## 使用 VDO 压缩存储和删除重复数据

​红帽企业 Linux8 包含虚拟数据优化器（VDO）驱动程序，可以优化块设备上数据的空间占用。VDO 是一个 Linux 设备映射器驱动程序，它可以减少块设备上的磁盘空间使用，同时最大限度减少数据重复，从而节省磁盘空间，甚至提高数据吞吐量。VDO 包括两个内核模块：kvdo 模块用于以透明的方式控制数据压缩，uds 则可用于重复数据删除。

​VDO 层位于现有块存储设备（如 RAID设备或本地磁盘）的顶部。这些块设备也可以是加密设备。存储层（如 LVM逻辑卷和文件系统）位于 VDO 设备之上。下图显示了在一个由使用优化存储设备的 KVM 虚拟机构成的基础架构中，VDO 所处的位置。

![vdo 存储概念图](images/linux/vdo-存储概念图.png)

​VDO 会按以下顺序对数据实施三个阶段的处理，以减少存储设备上的空间占用：

1. 零块消除将过滤掉仅包含零（0）的数据块，且仅在元数据中记录这些块的信息。非零数据块随即被传递到下一个处理阶段。该阶段将启用VDO设备中的精简配置功能。

2. 重复数据删除将去除几余的数据块。在创建相同数据的多个副本时，VDO 会检测重复数据块并更新元数据，以便使用这些重复块来引用原始数据块，而不会创建几余数据块。通用重复数据删除服务（UDS）内核模块将通过其维护的元数据来检查数据的几余。该内核模块是作为 VDO 的一部分而提供的。

3. 最后一个阶段是压缩。kvdo 内核模块使用 LZ4 压缩对块进行压缩，并以 4KB 块进行分组。

### 实施虚拟数据优化器

​利用 VDO 创建的逻辑设备被称为 VDO 卷。VDO 卷与磁盘分区类似；您可以将这些卷格式化为所需的文件系统类型，并像常规文件系统那样进行挂载。此外，您还可以将 VDO 卷用作LVM物理卷。

​要创建 VDO 卷，请指定块设备以及 VDO 向用户显示的逻辑设备的名称。您可以指定 VDO 卷的逻辑大小（可选）。VDO 卷的逻辑大小可以大于实际块设备的物理大小。

​由于 VDO 卷采用了精简配置，因此用户只能看到正在使用的逻辑空间，而无法了解实际可用的物理空间。如果在创建卷时未指定逻辑大小，则 VDO 会将实际物理大小视为卷的逻辑大小。这种采用 1:1 的比率映射逻辑大小与物理大小的方式有利于提高性能，但同时也会降低存储空间的使用效率。应根据您的基础架构要求来确定是优先考虑性能还是空间效率。

​当 VDO 卷的逻辑大小超过实际物理大小时，应使用 `vdostats--verbose` 命令主动监控卷统计信息，以查看实际使用情况。

#### 启用 VDO

​安装 *VDO* 和 *kmod* 软件包，以便在系统中启用 VDO。

```bash
[root@localhost ~]\# yum install vdo kmod-kvdo
......
Is this ok [y/N]: y
......
complete!
```

#### 创建 VDO

​要创建 VDO 卷，请运行 `vdo create` 命令。

```bash
[root@localhost ~]\# vdo create --name=vdo1 --device=/dev/vdd --vdoLogicalSize=50G
......
```

​如果省略逻辑大小，则生成的VDO卷将与其物理设备的大小相同。

​当 VDO 卷就位时，可以将它格式化为所选的文件系统类型并挂载于系统的文件系统层次结构下。

#### 分析 VDO 卷

​要分析 VDO 卷，请运行 `vdostatus` 命令。此命令将以 YAML 格式显示有关 VDO 系统的报告以及 VDO 卷的状态。此外，它还显示 VDO 卷的属性。使用 *--name=* 选项可指定特定卷的名称。如果省略特定卷的名称，`vdostatus` 命令的输出中将显示所有VDO卷的状态。

```bash
[root@localhost ~]\# vdo status --name=vdo1
......
```

​`vdolist` 命令显示当前启动的 VDO 卷的列表。您可以分别使用 `vdostart` 和 `vdostop` 命令来启动和停止 VDO 卷。
