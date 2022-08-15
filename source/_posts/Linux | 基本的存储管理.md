---
title: 基本的存储管理
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

## 添加分区、文件系统和持久挂载

### 磁盘分区

​通过创建磁盘分区，管理员可以使用其执行不同功能。在以下情况下，磁盘分区是必要或有益的：

- 限制应用或用户的可用空间；
- 将操作系统和程序文件与用户文件分隔开；
- 创建用于内存交换的单独区域；
- 限制磁盘空间使用，以提高诊断工具和备份映像的性能。

#### MBR 分区方案

​MBR 是一种比较传统的分区方案，其支持最多 4 个主分区。在 Linux 系统上，管理员可以使用 扩展分区和逻辑分区来创建最多 15 个分区。由于分区大小数据以 32 位值存储，使用 MBR 方案分区时，最大磁盘和分区大小为 2TB。

![MBR 分区示例](images/linux/MBR-分区示例.png)

​由于现代存储技术的不断发展，MBR 分区的 2TiB 限制已从理论转化为现在越来越频繁的现实问题，故新的 GUID 分区表（GPT 分区）正在取代 MBR 分区方案。

#### GPT 分区方案

​GPT 分区方案依赖于新的固件系统 —— UEFI。GPT 是 UEFI 标准的一部分，可以解决原有基于 MBR 的方案所带来的许多限制。

​GPT 分区方案最多可提供 128 个分区以及 8ZiB（80 亿 TiB）的分区和磁盘。GPT 分区提供了备份副本来确保数据的安全性，且提供了校验算法来检测 GPT 头和分区表中的错误和损坏。

![GPT 分区示例](images/linux/GPT-分区示例.png)

#### 使用 PARTED 管理分区

​Parted 分区实用程序同时支持管理 MBR 和 GPT 两种分区。

```bash
[root@localhost ~]\# parted -h
用法: parted [选项]... [设备 [命令 [参数]...]...]
接受以 参数s 执行的 命令s 来管理 设备.  如果未提供任何命令, 将以交互模式运行。

选项s:
  -h, --help                      显示此帮助信息
  -l, --list                      列举所有块设备的分区结构信息
  -m, --machine                   displays machine parseable output
  -s, --script                    never prompts for user intervention
  -v, --version                   displays the version
  -a, --align=[none|cyl|min|opt]  alignment for new partitions

命令s:
  align-check TYPE N                       检查分区 N 的对齐类型(min|opt)
  help [COMMAND]                           打印通用帮助信息, 或针对 COMMAND 的帮助信息
  mklabel,mktable LABEL-TYPE               创建新的磁盘标签(分区表)
  mkpart PART-TYPE [FS-TYPE] START END     创建新分区
  name NUMBER NAME                         以 NAME 命名分区 NUMBER
  print [devices|free|list,all|NUMBER]     显示 分区表|可用设备|空闲空间|所有分区|指定分区                                                (NUMBER) 
  quit                                     退出程序
  rescue START END                         恢复在 START 和 END 之间丢失的分区
  resizepart NUMBER END                    重新规划分区 NUMBER 的大小
  rm NUMBER                                删除分区 NUMBER
  select DEVICE                            选择要操作的存储设备
  disk_set FLAG STATE                      更改选定设备上的 FLAG 的状态为 STATE
  disk_toggle [FLAG]                       切换选定设备上的 FLAG 的 FLAG 为 STATE
  set NUMBER FLAG STATE                    更改分区 NUMBER 上的 FLAG 为状态 STATE
  toggle [NUMBER [FLAG]]                   切换分区 NUMBER 的 FLAG 为 STATE
  unit UNIT                                设置默认单位为 UNIT
  version                                  display the version number and copyright                                                 information of GNU Parted
```

> 默认情况下，*parted* 显示以 10 的幂次方表示的所有空间大小（KB、MB、GB）。可以使用 `unit` 子命令来更改默认设置，该子命令接受以下参数：
> 
> - *s* 表示扇区；
> - *B* 表示字节；
> - *MiB*、*GiB* 或 *TiB*（2 的幂次方）；
> - *MB*、*GB* 或 *TB* （10 的幂次方）；

```bash
[root@localhost ~]\# parted /dev/nvme0n1 unit s print
Model: NVMe Device (nvme)
Disk /dev/nvme0n1: 41943040s
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start     End        Size       Type     File system     Flags
 1      2048s     616447s    614400s    primary  xfs             boot
 2      616448s   4810751s   4194304s   primary  linux-swap(v1)
 3      4810752s  41943039s  37132288s  primary  xfs
```

##### 向新磁盘写入分区表

​要对新驱动器进行分区，首先必须为其写入磁盘标签。磁盘标签指示了所用的分区方案。

> `parted` 命令的更改是即时生效的，若误用了该命令，注定会导致数据丢失。

​以 root 用户身份，使用以下命令将 MBR 磁盘标签写入磁盘：

```bash
[root@localhost ~]\# parted /dev/nvme0n1 mklabel msdos
```

​若要写入 GPT 磁盘标签，则需使用该命令：

```bash
[root@localhost ~]\# parted /dev/nvme0n1 mklabel gpt
```

> mklabel 子命令会擦除现有分区表，其效果类似于 windows 中的格式化，仅当重复利用旧磁盘而不保留其中的旧有数据时使用。

##### 创建 MBR 分区

​创建 MBR 磁盘分区包含以下几个步骤：

1. 指定要在其上创建分区的磁盘设备。
   
以 root 用户身份执行 `parted` 命令，并指定该磁盘设备名称作为参数。这样将会以交互模式启动 parted 命令并显示命令提示符。

```bash
[root@localhost ~]\# parted /dev/nvme0n1
GNU Parted 3.2
Using /dev/nvme0n1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) 
```

2. 使用 `mkpart` 子命令创建新的主分区或扩展分区。

```bash
(parted) mkpart
Partition type? primary/extended? primary    # 创建主分区
```

> 若需创建多于四个分区，可以适当将后面的分区创建为扩展分区，并在其上创建逻辑分区。

3. 指示要在分区上创建的文件系统类型，如 *xfs* 或 *ext4*，此操作不会在分区上创建文件系统，仅为指示分区类型作用。

```bash
File system type? [ext2]? xfs    # xfs 
```

​要了解支持哪些文件系统类型，可以使用以下命令进行查询。

```bash
[root@localhost ~]\# parted /dev/nvme0n1 help mkpart
```

4. 指定磁盘上新分区开始的扇区。

```bash
Start? 2048s    # 第 2948 个扇区开始
```

​也可以使用 MB、MiB 等单位，默认单位为 MB。程序会自动舍入进行对齐。对于大多数磁盘而言，起始扇区设置为 2048 的倍数较为安全。

5. 指定应结束新分区的磁盘扇区。

```bash
End? 1000MB    # 分区大小 Size = End - Start
```

​提供了结束位置后，*parted* 即可利用新分区的详细信息来更新磁盘上的分区表。

6. 退出 *parted*。

```bash
(parted) quit
Information: You may need to update /etc/fstab.

[root@localhost ~]
```

7. 运行 `udevadm settle` 命令。此命令会等待系统检测新分区并在 */dev* 目录下创建关联的设备文件，完成操作后才会返回。

```bash
[root@localhost ~] udevadm settle
[root@localhost ~]
```

​上述操作的一条命令版本为：

```bash
[root@localhost ~] parted /dev/nvme0n1 mkpart primary xfs 2048s 1000MB
```

##### 创建 GPT 分区

​创建 GPT 磁盘分区包含以下几个步骤：

1. 指定要在其上创建分区的磁盘设备。
   
以 root 用户身份执行 `parted` 命令，并指定该磁盘设备名称作为参数。这样将会以交互模式启动 parted 命令并显示命令提示符。

```bash
[root@localhost ~]\# parted /dev/nvme0n1
GNU Parted 3.2
Using /dev/nvme0n1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) 
```

2. 使用 `mkpart` 子命令创建新的主分区或扩展分区。
   
对于 GPT 方案而言，每个分区都会获得一个名称。

```bash
(parted) mkpart
Partition name? []? userdata
```

3. 指示要在分区上创建的文件系统类型，如 *xfs* 或 *ext4*。这并不会在分区上创建文件系统；它仅仅指示分区类型。

```bash
File system type? [ext2]? xfs
```

4. 指定磁盘上新分区开始的扇区。

```bash
Start? 2048s
```

5. 指定应结束新分区的磁盘扇区。

```bash
End? 1000MB
```

​一旦提供了结束位置，parted 即可利用新分区的详细信息来更新磁盘上的分区表。

6. 退出 parted。

```bash
(parted) quit
Information: You may need to update /etc/fstab.

[root@localhost ~]\#
```

7. 运行 `udevadm settle` 命令。 此命令会等待系统检测新分区并在 */dev* 目录下创建关联的设备文件。只有在完成上述操作后，他才返回。

```bash
[root@localhost ~]\# udevadm settle
[root@localhost ~]\#
```

​作为交互模式的替代方法，您也可以按如下方式创建分区：

```bash
[root@localhost ~]\# parted /dev/nvme0n1 mkpart userdata xfs 2048s 1000MB
```

##### 删除分区

​以下步骤适用于 MBR 和 GPT 两种分区方案。

1. 指定包含要删除的分区的磁盘。
   
作为 root 用户，以磁盘设备作为唯一参数执行 `parted` 命令，从而通过命令提示符在交互模式下启动 parted。

```bash
[root@localhost ~]\# parted /dev/nvme0n1
GNU Parted 3.2
Using /dev/nvme0n1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted)
```

2. 确定要删除的分区的分区编号。

```bash
(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/nvme0n1: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number    Start    End        Size    File system        Name        Flags
  1        1049kB    1000MB    999MB    xfs                userdata       

```

3. 删除分区。

```bash
(parted) rm 1
```

​`rm` 子命令会立即从磁盘上的分区表中删除该分区。

4. 退出 `parted`。

```bash
(parted) quit
Information: You may need to update /etc/fstab.

[root@localhost ~]\#
```

##### 创建文件系统

​创建块设备后，下一步是向其中添加文件系统。红帽企业 Linux 支持许多不同的文件系统类型，其中两种常见的类型是 XFS 和 ext4。红帽企业 Linux 的安装程序 Anaconda 默认使用 XFS。

​以 root 用户身份，使用 `mkfs.xfs` 命令为块设备应用 XFS 文件系统。对于 ext4，可以使用 `mkfs .ext4`。

```bash
[root@localhost ~]\# mkfs.xfs /dev/nvme0n1
meta-data=/dev/nvme0n1        isize=512    account=4, agsize=60992 blks
         =                    sects=512    attr=2,    projid32bit=1
         =                    crc=1        finobt=1, sparse=1, rmapbt=0
         =                    reflink=1
data     =                     bsize=4096  blocks=243968, imaxpct=25
         =                    sunit=0        swidth=0 blks
naming   =version 2            bsize=4096    ascii-ci=0, ftype=1
log         =internal log        bsize=4096  blocks=1566, version=2
         =                    sectsz=512    sunit=0 blks, lazy-count=1
realtime =none                extsz=4096  blocks=0, rtextents=0
```

### 挂载文件系统

​关于手动挂载文件系统的方法已在 *23. 访问 Linux 文件系统* 中阐述，本节重点说明持久挂载文件系统。持久挂载是指系统开机或重启进入系统后自动挂载某块设备而无需进行手动挂载，此行为跟图形用户界面中的自动挂载有区别。

​要实现持久化挂载可已修改 */etc/fstab* 文件，向其添加新的条目。修改完毕后使用 `systemctl daemon-reload` 命令来实现修改生效。

```bash
[root@localhost ~]\# cat /etc/fstab

#
# /etc/fstab
# Created by anaconda on Wed Jun  2 22:59:16 2021
#
# Accessible filesystems, by reference, are maintained under '/dev/disk/'.
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info.
#
# After editing this file, run 'systemctl daemon-reload' to update systemd
# units generated from this file.
#
UUID=360e2f42-d6de-4209-8487-f36b7cc69cc2 /        xfs     defaults        0 0
UUID=529dd49b-7b72-4394-8fbb-98eb7f24de1e /boot    xfs     defaults        0 0
UUID=bb97577f-bf23-4a75-8ff1-46b7dce21eb9 none     swap    defaults        0 0
```

​fstab 文件的条目的第一个字段指定设备。本示例使用 UUID 来指定设备（推荐）。创建文件系统时会在其超级块中创建和存储 UUID。或者可以使用设备文件。

> 列出块设备的 UUID 可以使用 `lsblk` 或 `blkid` 命令：
> 
> ```bash
> [student@localhost ~]\$ lsblk -fs
> NAME      FSTYPE LABEL UUID                                 MOUNTPOINT
> sr0                                                         
> nvme0n1p1 xfs          529dd49b-7b72-4394-8fbb-98eb7f24de1e /boot
> └─nvme0n1                                                   
> nvme0n1p2 swap         bb97577f-bf23-4a75-8ff1-46b7dce21eb9 [SWAP]
> └─nvme0n1                                                   
> nvme0n1p3 xfs          360e2f42-d6de-4209-8487-f36b7cc69cc2 /
> └─nvme0n1 
> [student@localhost ~]\$ sudo blkid
> [sudo] password for student: 
> /dev/nvme0n1: PTUUID="3c479f3a" PTTYPE="dos"
> /dev/nvme0n1p1: UUID="529dd49b-7b72-4394-8fbb-98eb7f24de1e" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="3c479f3a-01"
> /dev/nvme0n1p2: UUID="bb97577f-bf23-4a75-8ff1-46b7dce21eb9" TYPE="swap" PARTUUID="3c479f3a-02"
> /dev/nvme0n1p3: UUID="360e2f42-d6de-4209-8487-f36b7cc69cc2" BLOCK_SIZE="512" TYPE="xfs" PARTUUID="3c479f3a-03"
> ```

​第二个字段指定挂载点，通过它可以访问目录结构中的块设备，挂载点必须存在于父文件系统中。

​第三个字段包含文件系统类型，如 *xfs*、*ext4*、*swap*（交换分区）等。

​第四个字段的值可以取以逗号分隔的多个应用于设备的选项。常见值的意义如下：

| 选项       | 描述                        |
| -------- | ------------------------- |
| rw       | 按可读可写权限挂载。                |
| suid     | 允许文件或目录的所属用户位和所属组位生效。     |
| exec     | 允许执行二进制程序。                |
| dev      | 解释字符或屏蔽特殊设备于文件系统上。        |
| auto     | 可以使用 `mount -a` 手动挂载上。    |
| nouser   | 不允许普通用户手动挂载此设备。           |
| async    | 异步存储（性能较佳），将内容写入日志在保存到磁盘。 |
| defaults | 以上选项的合集。                  |

​第五个字段的值表示 ”指定分区是否被 `dump` 程序备份“，0 代表不备份，1 代表备份，2 代表不定期备份。

​第六个字段的值表示 ”指定分区是否被 `fsck` 程序检测“, 0 代表不检测，其他数字代表检测的优先级，递增优先级越低。

> 添加新条目于 */etc/fstab* 中后需使用 `mount -a` 选项来测试下挂载是否成功，确保无误后再重启测试（有可能会导致系统无法启动）。
> 
> 修改完 */etc/fstab* 中的条目需使用 `mount -o remount` 命令来重新挂载（`mount -a` 命令对已挂载上的同名设备不会重新挂载）。

## 管理交换空间

​交换空间是受 Linux 内核内存管理子系统控制的磁盘区域。内核通过使用交换空间将不活动的内存页暂存在交换空间中来节省 RAM，交换空间一般以磁盘分区的形式存在，称为 *交换分区*。系统内存和交换分区组合在一起称为虚拟内存。由于交换分区位于磁盘上，其读取和写入速度较慢。

​关于交换分区的划定大小有下表的建议：

| RAM                | 交换分区大小      | 允许休眠功能时的交换内存大小 |
| ------------------ | ----------- | -------------- |
| 2GiB 或 以下          | RAM 大小 x 2  | RAM 大小 x 3     |
| 介于 2GiB 和 8GiB 之间  | 与 RAM 大小 相同 | RAM 大小 x 2     |
| 介于 8GiB 和 64GiB 之间 | 至少 4GiB     | RAM 大小 x 1.5   |
| 64 GiB 以上          | 至少 4GiB     | 不建议开启休眠功能      |

​休眠：指关闭计算机电源之前将内存内容转储到交换分区（注意：硬盘是非易失性存储设备），重新打开计算机后系统会从交换分区中恢复内存上的内容，无需完全启动。

### 创建交换空间

​创建交换空间的基本操作可概述为：创建文件系统类型为 linux-swap 的分区、为设备放置交换签名。

​以前 `parted` 工具可以根据分区文件系统类型来确定是否应激活设备，但现在即使该程序不再使用分区文件系统类型，设置此类型也可以使管理员快速确定该分区的用途。

```bash
[root@localhost ~]\# parted /dev/nvme0n1
GNU Parted 3.2
Using /dev/nvme0n1
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/nvme0n1: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number    Start    End        Size    File system        Name    Flags
  1        1049kB    1001MB    1000MB                    data

(parted) mkpart
Partition name? []? swap1
File system type? [ext2]? linux-swap
Start? 1001MB
End? 1257MB
(parted) print
Model: Virtio Block Device (virtblk)
Disk /dev/nvme0n1: 5369MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number    Start    End        Size    File system        Name    Flags
  1        1049kB    1001MB    1000MB                    data
  2        1001MB    1257MB    256MB    linux-swap(v1)    swap

(parted) quit
Information: You may need to update /etc/fstab.

[root@localhost ~]\#
```

​创建分区后，运行 `udevadm settle` 命令。此命令会等待系统检测新分区并在 */dev* 中创建关联的设备文件。只有在完成上述操作后，它才返回。

```bash
[root@localhost ~]\# udevadm settle
[root@localhost ~]\#
```

#### 格式化设备

​`mkswap` 命令向设备应用交换签名。与其他格式化实用程序不同，`mkswap` 在设备开头写入单个数据块，而将设备的其余部分保留为未格式化，这样内核就可以使用它来存储内存页。

```bash
[root@localhost ~]\# mkswap /dev/nvme0n1
Setting up swapspace version 1, size = 244 MiB(255848448 bytes) no label, UUID=39e2667a-9458-42fe-9665-c5c854605881

```

#### 激活交换空间

​可以使用 swapon 命令激活已格式化的交换空间。

​使用 `swapon` 并将设备作为参数，或者使用 `swapon -a` 来激活 */etc/fstab* 文件中列出的所有交换空间。使用 `swapon --show` 和 `free` 命令检查可用的交换空间。

```bash
[root@localhost ~]\# free
        total    used    free    shared    buff/cache    available
Mem:    1873036    134688    1536436    16748    201912        1576044
Swap:    0        0        0
[root@localhost ~]\# swapon /dev/nvme0n1
[root@localhost ~]\# free
        total    used    free    shared    buff/cache    available
Mem:     1873036 135044    1536040    16748    201952        1575690
Swap:    249852    0        249852
```

​可以使用 swapoff 命令停用交换空间。如果交换空间具有写入的页面，swapoff 会尝试将这些页面移动到其他活动交换空间或将其写回到内存中。如果无法将数据写入到其他位置，则 `swapoff` 命令会失败，并显示错误，而交换空间仍保持活动。

#### 持久激活交换空间

​要想在每次启动时都激活交换空间，请在 */etc/fstab* 文件中放置一个条目。基于上面创建的交换空间，以下示例显示了 */etc/fstab* 中比较典型的一行。

```bash
  UUID=39e2667a-9458-42fe-9665-c5c854605881    swap    swap    defaults    0    0
```

​该示例使用 UUID 作为第一个字段。格式化设备时，`mkswap` 命令会显示该 UUID。如果丢失了 `mkswap` 的输出，请使用 `lsblk --fs` 命令。作为替代方法，您也可以在第一个字段中使用设备名称。 

​第二个字段通常为 mount point 保留。但是，由于交换设备无法通过目录结构访问，因此该字段取占位符值为 *swap* 或者 *none*。

​第三个字段是文件系统类型。交换空间的文件系统类型是 *swap*。

​第四个字段是选项，本示例中使用了 *defaults* 选项，关于此选项的介绍可以参见上文。

​最后两个字段是 *dump* 标志和 *fsck* 顺序，这两个字段的值默认为 0 就行。

​在 /etc/fstab 文件中添加或删除条目时，可以运行 `systemctl daemon-reload` 命令或重启服务器，以便让 systemd 注册新配置。

```bash
[root@localhost ~]\# systemctl daemon-reload
```

#### 设置交换空间优先级

​默认情况下，系统会按顺序使用交换空间，即内核先使用第一个已激活交换空间，直至其空间已满，然后开始使用第二个交换空间。不过，您也可以为每个交换空间定义一个优先级，从而强制按该顺序使用交换空间。

​要设置优先级，请在 */etc/fstab* 中使用 *pri* 选项。内核会首先使用优先级最高的交换空间。默认优先级为 -2。

​以下示例显示了 */etc/fstab* 中定义的三个交换空间。内核首先使用最后一个条目，其优先级为 *pri=10*。当该空间已满时，他将使用第二个条目，其优先级为 *pri=4*。最后，它将使用第一个条目，其默认优先级为 -2。

```bash
  UUID=a2345678-9458-42fe-9665-c5c854605881    swap    swap    defaults    0    0
  UUID=b3245c23-9458-42fe-9665-c5c854605881    swap    swap    defaults    0    0
  UUID=39e2667a-9458-42fe-9665-c5c854605881    swap    swap    defaults    0    0
```

​使用 swapon --show 可以显示交换空间的优先级。

​当交换空间具有相同的优先级时，内核会以轮询方式向其中写入。
