---
title: 管理逻辑卷
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

## 逻辑卷管理 (LVM) 概述

​逻辑卷和逻辑卷管理有助于更加轻松地管理磁盘空间。如果托管逻辑卷的文件系统需要更多空间，可以将其卷组中的可用空间分配给逻辑卷，并且可以调整文件系统的大小。如果磁盘开始出现错误，可以将替换磁盘注册为物理卷放入卷组中，并且逻辑卷的区块可迁移到新磁盘。

| LVM 概念  | 定义                                                                                                                   |
| ------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| 物理设备    | 物理设备是用于保存逻辑卷中所存储数据的存储设备。它们是块设备，可以是磁盘分区、整个磁盘、RAID 阵列或 SAN 磁盘。设备必须初始化为 LVM 物理卷，才能与 LVM 结合使用。整个设备将用作一个物理卷。                                          |
| 物理卷（PV） | 在 LVM 系统中使用设备之前，必须将设备初始化为物理卷。LVM 工具会将物理卷划分为物理区块（PE），它们是充当物理卷上最小存储块的小块数据。                                                                         |
| 卷组（VG）  | 卷组是存储池，由一个或多个物理卷组成。它在功能上与基本存储中的整个磁盘相当。一个 PV 只能分配给一个 VG。VG 可以包含未使用的空间和任意数目的逻辑卷。                                                                   |
| 逻辑卷（LV） | 逻辑卷根据卷组中的空闲物理区块创建，提供应用、用户和操作系统所使用的 “存储” 设备。LV 是逻辑区块（LE）的集合，LE 映射到物理区块（PV 的最小存储块）。默认情况下，每个 LE 将映射到一个 PE。设置特定 LV 选项将会更改此映射；例如，镜像会导致每个 LE 映射到两个 PE。 |

## 实施 LVM 存储

​创建 LVM 存储需要几个步骤。第一步是确定要使用的物理设备。在组装完一组合适的设备之后，系统会将它们初始化为物理卷，以便将它们识别为属于 LVM。这些物理卷随机被合并到卷组中。此时将会创建一个磁盘空间池，从中可以分配逻辑卷。利用卷组的可用空间创建的逻辑卷可以格式化为文件系统、作为交换空间激活，也可以实现持久挂载或激活。

![Linux 逻辑卷概念抽象](images/linux/Linux-逻辑卷概念抽象.png)

​LVM 提供了一组全面的命令行工具，用于实施和管理 LVM 存储。这些命令行工具可用在脚本中，从而使它们更适于自动化。

### 创建逻辑卷

​创建逻辑卷的步骤如下：

1. 准备物理设备

​使用 `parted`、`gdisk` 或 `fdisk` 创建新分区，以便与 LVM 结合使用。在 LVM 分区上，始终将分区类型设置为 Linux LVM；对于 MBR 分区，使用 *0x8e*。如有必要，使用 `partprobe` 向内核注册新分区。

​也可以使用完整磁盘、RAID 阵列或 SAN 磁盘。

​只有当没有已准备好的物理设备并且需要需要新物理卷来创建或扩展卷组时，才需要准备物理设备。

```bash
[root@localhost ~]\# parted -s /dev/nvme1n0 mkpart primary 1MiB 769MiB
[root@localhost ~]\# parted -s /dev/nvme1n0 mkpart primary 770MiB 1026MiB
[root@localhost ~]\# parted -s /dev/nvme1n0 mkpart set 1 lvm on
[root@localhost ~]\# parted -s /dev/nvme1n0 mkpart set 2 lvm on
```

2. 创建物理卷

​使用 `pvcreate` 将分区（或其他物理设备）标记为物理卷。`pvcreate` 命令会将物理卷分成若干固定大小的物理区块（PE），如 4MiB 块。

```bash
[root@localhost ~]\# pvcreate /deb/nvme0n1p2 /dev/nvme0n1p1
```

​此命令会将设备 /dev/nvme0n1p2 和 /dev/nvme0n1p1 标记为 PV，准备好分配到卷组中。

​仅当没有空闲的 PV 可以创建或扩展 VG 时，才需要创建 PV。

3. 创建卷组

​使用 `vgcreate` 将一个或多个物理卷结合为一个卷组。卷组在功能上与硬盘相当；利用卷组中的可用物理区块池可以创建逻辑卷。

​`vgcreate` 命令行由卷组名后跟一个或多个要分配给此卷组的物理卷组成。

```bash
[root@localhost ~]\# vgcreate vg01 /dev/nvme0n1p1 /dev/nvme0n1p2
```

​此命令将创建名为 `vg01` 的 VG，他的大小是 /dev/nvme0n1p1 和 /dev/nvme0n1p2 两个 PV 的大小之和（以 PE 单位计）。

​仅当 VG 尚不存在时，才需要创建 VG。可能会出于管理原因创建额外的 VG，用于管理 PV 和 LV 的使用。否则，可在需要时扩展现有 VG 以容纳新的 LV。

4. 创建逻辑卷

​使用 lvcreate 可根据卷组中的可用物理区块创建新的逻辑卷。`lvcreate` 命令中至少包含用于设置 LV 名称的 *-n* 选项、用于设置 LV 大小（以字节为单位）的 *-L* 选项或用于设置 LV 大小（以区块数为单位）的 *-l* 选项，以及托管此逻辑卷的卷组的名称。

```bash
[root@localhost ~]\# lvcreate -n 1v01 -L 700M vg01
```

​这会在 VG vg01 中创建一个名为 lv01、大小为 700 MiB 的LV。针对所请求的大小，如果卷组没有足够数量的可用物理区块，此命令将失败。另外请注意，如果大小无法完全匹配，则将四舍五入为物理区块大小的倍数。

​您可以使用 *-L* 选项来指定大小，它预期大小单位为字节、兆字节（二进制兆字节，1048576 字节）、千兆字节（二进制千兆字节）等等。或者，您可以使用 *-l* 选项，它预期大小指定为若干物理区块。

​以下列表提供了一些创建 LV 的示例：

- `lvcreate -L 128M`：将逻辑卷的大小确定为正好 128MiB。
- `lvcreate -l 128`：将逻辑卷的大小确定为正好 128 个区块。字节总数取决于基础物理卷上物理区块块的大小。

> 不同的工具将使用传统名称 /dev/vgname/lvname 或内核设备映射程序名 /dev/mapper/vgname-lvname，显示逻辑卷名。

5. 添加文件系统
   
使用 `mkfs` 在新逻辑卷上创建 *xfs* 文件系统。或者，根据首选文件系统创建文件系统，例如 ext4。

```bash
[root@localhost ~]\# lvcreate -n 1v01 -L 700M vg01
```

​要使文件系统在重新启动后依然可用，可以执行以下步骤：

- 使用 mkdir 创建挂载点。

```bash
[root@localhost ~]\# mkdir /mnt/data
```

- 向 /etc/fstab 文件中添加以下条目：

```bash
/dev/vg01/lv01    /mnt/data    xfs        defaults    1 2
```

> 按名称挂载逻辑卷等同于按 UUID 进行挂载，因为即使最初按名称将它们添加到卷组中，LVM 也会根据 UUID 查找物理卷。

- 执行 `mount /mnt/data`，挂载刚刚在 `/etc/fstab` 中添加的文件系统。

```bash
[root@localhost ~]\# mount /mnt/data
```

### 删除逻辑卷

​删除逻辑卷的步骤如下：

1. 准备文件系统

​将必须保留的所有数据移动到另一个文件系统。使用 `umount` 命令卸载文件系统，然后删除与该文件系统关联的所有 /etc/fstab 条目。

```bash
[root@localhost ~]\# umount /mnt/data
```

> 此操作会破坏所存储的数据，一定要注意备份或移动数据。

2. 删除逻辑卷

​使用 `lvremove DEVICE_NAME` 删除不再需要的逻辑卷。 

```bash
[root@localhost ~]\# lvremove /dev/vg01/lv01
```

​运行此命令之前，卸载 LV 文件系统。在删除 LV 之前，该命令会提示进行确认。LV 的物理区块会被释放，并可用于分配给卷组中的现有 LV 或新 LV。

3. 删除卷组

​使用 `vgremove VG_NAME` 删除不再需要的卷组。

```bash
[root@localhost ~]\# vgremove vg01
```

​VG 的物理卷会被释放，并可用于分配给系统中的现有 VG 或新 VG。

4. 删除物理卷

​使用 `pvremove` 删除不再需要的物理卷。使用空格分隔的 PV 设备列表同时删除多个 PV。此命令将从分区（或磁盘）中删除 PV 元数据。分区现已空闲，可重新分配或重新格式化。

```bash
[root@localhost ~]\# pvremove /dev/nvme0n1p2 /dev/nvmen1p1
```

## 查看 LVM 状态信息

### 物理卷

​使用 `pvdisplay` 显示有关物理卷的信息。要列出有关所有物理卷的信息，可以使用不带参数的命令。要列出有关特定物理卷的信息，可以将相应的设备名称传给该命令。

```bash
[root@localhost ~]\# vgdisplay vg01
--- Volume group ---
VG Name                    vg01
System ID
Format                    lvm2
Metadata Areas            2
Metadata Sequence No     2
VG Access                read/write
VG Status                resizable
MAX LV                    0
Cur    LV                    1
Open LV                    1
Max PV                    0
Cur PV                    2
Act    PV                    2
VG Size                    1016.00 MiB
PE Size                    4.00MiB
Total PE                254
Alloc PE / Size            175 / 700.00 MiB
Free PE / Size            79 / 316.00 MiB
VG UUID                    3snNw3-CF71-CcYG-Llk1-p6EY-rHEv-xFUSez
```

- *VG Name* 卷组的名称；
- *VG Size* 是存储池可用于逻辑卷分配的总大小；
- *Total PE* 是以 PE 单位表示的总大小；
- *Free PE / Size* 显示 VG 中有多少空闲空间可用于分配给新 LV 或扩展现有 LV。

### 逻辑卷

​使用 `lvdisplay` 显示有关逻辑卷的信息。如果未向命令提供任何参数，则将显示有关所有 LV 的信息；如果提供了 LV 设备名称作为参数，此命令将显示有关该特定设备的信息。

```bash
[root@localhost ~]\# lvdisplay /dev/vg01/lv01
--- Logical volume ---
LV Path                    /dev/vg01/lv01
LV Name                 lv01
VG Name                    vg01
LV UUID                    5IyRea-WBZw-xLHK-3h2a-IuVN-YaeZ-i3IRrN
LV Write Access            read / write
LV Creation host, time    host.lab.example.com, 2019-03-28 17:17:47 -0400
LV Status                available
# open                    1
LV Size                    700MiB
Current LE                175
Segments                1
Allocation                inherit
Read ahead sectors        auto
- current set to         256
Block device             252:0
```

- *LV Path* 显示逻辑卷的设备名称；
  
某些工具可能会将设备名报告为 /dev/mapper//vgname-lvname；两个名称都表示同一 LV。

- *VG Name* 显示从其分配 LV 的卷组。

- *LV Size* 显示 LV 的总大小。使用文件系统工具确定可用空间和数据存储的已用空间。

- *Current LE* 显示此 LV 使用的逻辑区块数。LE 通常映射到 VG 中的物理区块，并因此映射到物理卷。

## 扩展逻辑卷

### 扩展和缩减卷组

​扩展卷组指可以通过添加额外的物理卷并为逻辑卷分配新的物理区块来为卷组增加更多磁盘空间。

​缩减卷组指将未使用的物理卷从卷组中删除。首先，使用 `pvmove` 命令将数据从一个物理卷上的区块移动到卷组中其他物理卷上的区块。通过这种方式，可以将新磁盘添加到现有卷组，将数据从较旧或较慢的磁盘移动到新磁盘，并将旧磁盘从卷组中删除。可在卷组中的逻辑卷正在使用时执行这些操作。

> 以下操作以 /dev/nvme0n1 为例，命令参数需以实际操作的磁盘为准。

#### 扩展卷组

​扩展卷组包含以下步骤：

1. 准备物理设备并创建物理卷
   
像创建新卷组一样，如果还没有准备好物理卷，则必须创建新分区，并准备好将其用作物理卷。

```bash
[root@localhost ~]\# parted -s /dev/nvme0n1 mkpart primary 1027MiB 1539MiB
[root@localhost ~]\# parted -s /dev/nvme0n1 set 3 lvm on
[root@localhost ~]\# pvcreate /dev/nvme0n1p3
```

​仅当没有空闲的 PV 可以扩展 VG 时，才需要创建 PV。

2. 扩展卷组
   
使用 `vgextend` 向卷组中添加新物理卷。使用 VG 名称和 PV 设备名称作为 `vgextend` 的参数。

```bash
[root@localhost ~]\# vgextend vg01 /dev/nvme0n1p3
```

​此命令会对 vg01 VG 进行扩展，扩展幅度为 /dev/nvme0n1p3 PV 的大小。

3. 验证新空间是否可用
   
使用 `vgdisplay` 确认额外的物理区块是否可用。检查输出中的 *Free PE / Size*。他不应当为零。

```bash
[root@localhost ~]\# vgdisplay vg01
--- Volume group ---
VG Name                vg01
......
Free PE / Size        178 / 712.00 MiB
......
```

#### 缩减卷组

​缩减卷组包含以下步骤：

1. 移动物理区块
   
使用 `pvmove PV_DEVICE_NAME` 将要删除的物理卷中的所有物理区块都重新放置到卷组中的其他物理卷上。其他物理卷中必须有足够数量的空闲区块来容纳这些移动内容。仅当 VG 中存在足够的空闲区块，且所有这些区块都来自其他 PV 时，才能执行此操作。

```bash
[root@localhost ~]\# pvmove /dev/nvme0n1p3
```

​此命令会将 PE 从 /dev/nvme0n1p3 移动到同一 VG 中具有空闲 PE 的 PV。

> 使用 `pvmove` 前需备份卷组中所有逻辑卷上存储的数据。如果操作期间意外断电，可能会导致卷组状态不一致。这可能导致卷组中逻辑卷上的数据丢失。

2. 缩减卷组
   
使用 `vgreduce VG_NAME PV_DEVICE_NAME` 从卷组中删除物理卷。

```bash
[root@localhost ~]\# vgreduce vg01 /dev/nvme0n1p3
```

​它将从 vg01 VG 中删除 /dev/nvme0n1p3 PV，并可以添加到其他 VG。或者，也可以使用 `pvremove` 永久停止将设备用作 PV。

#### 扩展逻辑卷和 XFS 文件系统

​逻辑卷的一个优势在于能够在不停机的情况下增加其大小。可将卷组中的空闲物理区块添加到逻辑卷以扩展其容量，然后可使用逻辑卷扩展所包含的文件系统。

##### 扩展逻辑卷

​缩减逻辑卷包含以下步骤：

1. 验证卷组是否具有可用的空间
   
使用 `vgdisplay` 验证是否有足够的物理区块可供使用。

```bash
[root@localhost ~]\# vgdisplay vg01
--- Volume group ---
VG Name                vg01
......
Free PE / Size         178 / 712.00 MiB
......
```

2. 扩展逻辑卷
   
使用 `lvextend LV_DEVICE_NAME` 将逻辑卷扩展为新的大小。

```bash
[root@localhost ~]\# lvextend -L +300M /dev/vg01/lv01
```

​此命令会将逻辑卷 lv01 的大小增加 300 MiB。请注意大小前面的加号（+），它表示向现有大小增加此值；如无该符号，该值定义 LV 的最终大小。

​和 `lvcreate` 一样，存在不同的方法来指定大小：*-l* 选项预期以物理区块数作为参数。*-L* 选项则预期以大小（单位为字节、兆字节、千兆字节等等）作为参数。

​以下列表提供了一些扩展 LV 的示例。

| 命令                   | 结果                       |
| -------------------- | ------------------------ |
| lvextend -l 128      | 将逻辑卷大小调整为正好 128 个区块。     |
| lvextend -l +128     | 向逻辑卷的当前大小添加 128 个区块。     |
| lvextend -L 128M     | 将逻辑卷的大小调整为正好 128 MiB。    |
| lvextend -L +128M    | 向逻辑卷的当前大小添加 128 MiB。     |
| lvextend -l +50%FREE | 向 LV 添加 VG 中当前可用空间的 50%。 |

3. 扩展文件系统
   
使用 `xfs_growfs mountpoint` 可以扩展文件系统以占用已扩展的 LV。使用 `xfs_growfs` 命令时，必须挂载目标文件系统。在调整文件系统大小时，可以继续使用该文件系统。

```bash
[root@localhost ~]\# xfs_growfs /mnt/data
```

> 常见错误是运行 `lvextend` 但忘记运行 `xfs_growfs`。连续运行两个步骤的一种替代方法是在 `lvextend` 命令中包含 *-r* 选项。这将使用 *fsadm* 在扩展 LV 后调整文件系统的大小。它可以用于多种不同的文件系统。

​验证已挂载文件系统的新大小：

```bash
[root@localhost ~]\# df -h /mountpoint
```

#### 扩展逻辑卷和 EXT4 文件系统

​同样需要先扩展逻辑卷，参考 *c. 扩展逻辑卷和 XFS 文件系统* 中的 *扩展逻辑卷* 一节。调整文件系统大小时与 *XFS* 不同。

1. 验证卷组是否具有可用的空间
   
使用 `vgdisplay VGNAME` 验证卷组中是否有足够数量的物理区块可供使用。

2. 扩展逻辑卷
   
使用 `lvextend -l +extents /dev/vgname/lvname` 对逻辑卷 */dev/vgname/lvname* 进行扩展，扩展的幅度为 extents 值。

3. 扩展文件系统
   
使用 `resize2fs /dev/vgname/lvname` 可以扩展文件系统已占用新扩展的 LV。运行扩展命令时，可以挂载并使用文件系统，可以包含 *-p* 选项以监控调整大小操作的进度。

```bash
[root@localhost ~]\# resize2fs -p /dev/vgname/lvname
```

> `xfs_growfs` 和 `resize2fs` 传递的参数类型不同，前者传挂载点、后者传逻辑卷名称。

#### 扩展逻辑卷和交换空间

​格式化为交换空间的逻辑卷必须卸载后才能进行扩展。

1. 验证卷组是否具有可用的空间
   
使用 `vgdisplay VGNAME` 验证卷组中是否有足够数量的物理区块可供使用。

2. 停用交换空间
   
使用 `swapoff -v /dev/vgname/lvname` 可以停用逻辑卷上的交换空间。

> 系统必须有足够内存或交换空间以便被停用的逻辑卷上的交换空间上的内容可以转移到它们上。

3. 扩展逻辑卷
   
使用 `lvextend -l +extents /dev/vgname/lvname` 可对逻辑卷 /dev/vgname/lvname 进行扩展，扩展的幅度为 extents 值。

4. 将逻辑卷格式化为交换空间
   
使用 `mkswap /dev/vgname/lvname` 可将整个逻辑卷格式化为交换空间。

5. 激活交换空间
   
使用 `swapon -va /dev/vgname/lvname` 可以激活逻辑卷上的交换空间。
