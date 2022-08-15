---
title: 访问 Linux 文件系统
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

## 文件系统和存储设备

​Linux 服务器上的文件是按照文件系统层次结构（一棵颠倒的目录树）访问的，不同目录下的文件可能存储在不同的存储设备下。某个存储设备的文件系统要想接入当前系统的文件系统中需要将其挂载到当前系统文件系统中的某个空目录上，该目录被称为 “挂载点”。进行挂载后便可以访问该设备存储的文件，该挂载点相当于该存储设备的根目录。

### 文件系统、存储和块设备

​在 Linux 中，对存储设备的低级别访问是由一种称为块设备的特殊类型文件提供的。在挂载这些设备前，必须先使用文件系统对其进行格式化。

​块设备文件和其他的设备文件一起存储在 */dev* 目录中。设备文件是由操作系统自动创建的。在红帽企业 Linux 中，检测到的第一个 SATA/PATA 、SAS、SCSI 或 USB 硬盘驱动器被称为 */dev/sda*，第二个被称为 */dev/sdb*，以此类推。这些名称代表整个硬盘驱动器。

| 设备类型                              | 设备命名模式               |
| ------------------------------------- | -------------------------- |
| SATA/SAS/USB 附加存储                 | /dev/sda、/dev/sdb...      |
| virtio-blk 超虚拟化存储（部分虚拟机） | /dev/nvme0、/dev/nvme1     |
| NVMe 附加存储（很多 SSD）             | /dev/nvme0、/dev/nvme1     |
| SD/MMC/eMMC 存储（SD卡）              | /dev/mmcblk0、/dev/mmcblk1 |

> 最新的 virtio-scsi 超虚拟化存储技术对应的命名形式为 /dev/sd*。

### 磁盘分区和逻辑卷

​一块磁盘可能会被分为若干个分区以实现不同用途的存储需求，如系统分区、用户主目录分区、交换分区、启动分区等。这些分区的本质也是块设备，其命名形式为：sd\**n*、nvme*m*p*n*（n 代表第几个分区）等。

​整理磁盘和分区的另一种方式是通过*逻辑卷管理*（LVM）。通过 *LVM*，一个或多个块设备可以汇集为一个存储池，称为卷组。然后，卷组中的磁盘空间被分配到一个或多个逻辑卷，它们的功能等同于驻留在物理磁盘上的分区。LVM 系统在创建时为卷组和逻辑卷分配名称。LVM 在 */dev* 中创建一个名称与组名匹配的目录，然后在该新目录中创建一个与逻辑卷同名的符号链接。之后，可以挂载该逻辑卷文件。例如，如果一个卷组名 myvg，其中有一个名为 mylv 的逻辑卷，那么其逻辑卷设备文件的完整路径名为 */dev/myvg/mylv*。 

>逻辑卷设备的命名形式实际上是建立与实际设备文件的符号链接，以此来访问该文件，其名称在每次启动时可能会有所不同。还有一种逻辑卷设备的命名形式，那就是与常用的 */dev/mapper* 中的文件建立链接，也是一种与实际设备文件的符号链接。 

## 检查文件系统

​使用 `df` 命令对本地和远程文件系统设备及可用空间大小可以有个简略了解。

```bash 
[root@localhost ~]\# df
Filesystem     1K-blocks    Used Available Use% Mounted on
devtmpfs         1880292       0   1880292   0% /dev
tmpfs            1899208       0   1899208   0% /dev/shm
tmpfs            1899208    9884   1889324   1% /run
tmpfs            1899208       0   1899208   0% /sys/fs/cgroup
/dev/nvme0n1p3  18555904 4878480  13677424  27% /
/dev/nvme0n1p1    301728  178008    123720  59% /boot
tmpfs             379840    1168    378672   1% /run/user/42
tmpfs             379840       0    379840   0% /run/user/0
```

​其中 *tmpfs* 和 *devtmpfs* 设备是系统内存中的文件系统，在系统重启后，写入 *tmpfs* 或 *devtmpfs* 的文件都会消失。

​使用选项 *-h*【单位为 KiB（2\^10）、MiB（2\^20）或GiB（2\^30）】或选项 *-H*【单位为 KB(10\^3)、MB(10\^6)或GB(10\^9)。

```bash
[root@localhost ~]\# df -h
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        1.8G     0  1.8G   0% /dev
tmpfs           1.9G     0  1.9G   0% /dev/shm
tmpfs           1.9G  9.6M  1.9G   1% /run
tmpfs           1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/nvme0n1p3   18G  4.7G   14G  27% /
/dev/nvme0n1p1  295M  174M  121M  59% /boot
tmpfs           371M  1.2M  370M   1% /run/user/42
tmpfs           371M     0  371M   0% /run/user/0
-i,	--inodes	# 输出 inodes 数而不是大小
```

​使用 `du` 命令查看某一特定目录树使用的空间的详细信息，同 `df`，其也有 *-h* 和 *-H* 两个选项。

```bash
[root@localhost ~]\# du /home/student/.config
64	/home/student/.config/pulse
0	/home/student/.config/gnome-session/saved-session
0	/home/student/.config/gnome-session
......
[root@localhost ~]\# du -h /home/student/.config
64K	/home/student/.config/pulse
0	/home/student/.config/gnome-session/saved-session
0	/home/student/.config/gnome-session
......
```

## 挂载和卸载文件系统

### 手动挂载文件系统

​要在系统中挂载某个存储设备才能访问其内容，使用 `mount` 命令来手动挂载某个存储设备。

```bash
mount -a [options]	# 挂载 /etc/fstab 配置文件中涉及的所有配置
mount [options] <source> <directory>	# 将设备文件系统挂载到目录
```

​其中，指定设备时既可以使用 */dev* 目录中的设备名称也可以使用格式化文件系统时产生的 UUID。

### 查看系统连接的块设备

​使用 `lsblk` 命令来列出指定块设备或所有可用块设备的详细信息：

```bash
[root@localhost ~]\# lsblk	# 不会列出临时内存文件系统 tmpfs
NAME        MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0          11:0    1 1024M  0 rom  
nvme0n1     259:0    0   20G  0 disk 
├─nvme0n1p1 259:1    0  300M  0 part /boot
├─nvme0n1p2 259:2    0    2G  0 part [SWAP]
└─nvme0n1p3 259:3    0 17.7G  0 part /
```

### 按块设备名称挂载

​以下命令将 */dev/sdb1* 设备挂载到 */mnt/data* 目录中。

```bash
[root@localhost ~]\# mount /dev/sdb1 /mnt/data
```

​若要挂载成功，目标目录必须存在，默认情况下，*/mnt* 目录存在并用作临时挂载点，可以将 */mnt* 目录用作临时挂载点（创建 */mnt* 的一个子目录会更好）。

> 若系统调整过存储设备则操作系统检测磁盘的顺序可能会发生变化，将会导致存储设备的名称的改变，最好通过以下方式（按照文件系统 UUID 挂载）进行挂载。

### 按文件系统 UUID 挂载

​UUID 即通用唯一识别码，它唯一性的代表某个事物，可用作某个存储设备上的文件系统的唯一性标识。使用 `lsblk -fp` 命令可以列出设备的完整路径、其 UUID 和挂载点，以及分区中文件系统的类型。如果未挂载文件系统，则挂载点将为空。

```bash
[root@localhost ~]\# lsblk -fp
NAME             FSTYPE LABEL UUID                                 MOUNTPOINT
/dev/sr0                                                           
/dev/nvme0n1                                                       
├─/dev/nvme0n1p1 xfs          529dd49b-7b72-4394-8fbb-98eb7f24de1e /boot
├─/dev/nvme0n1p2 swap         bb97577f-bf23-4a75-8ff1-46b7dce21eb9 [SWAP]
└─/dev/nvme0n1p3 xfs          360e2f42-d6de-4209-8487-f36b7cc69cc2 /
```

​根据文件系统的 UUID 挂载文件系统。

```bash
[root@localhost ~]\# mount UUID="360e2f42-d6de-4209-8487-f36b7cc69cc2" /
```

### 自动挂载可移动存储设备

​登陆图形界面插入任何可移动存储时会自动挂载。可移动存储设备将挂载到 */run/media/USERNAME/LABEL*，其中 USERNAME 是登陆图形界面环境的用户名，而 LABEL 是一个标识符，通常是创建时给文件系统取的名称（如果存在）。在移除设备之前，应手动将它卸载。

### 卸载文件系统

​关机和重新启动过程会自动卸载所有文件系统，在此过程中，缓存在内存中的任何文件系统数据都将会刷新到存储设备中，从而确保文件系统不会遭受数据损坏。

> 文件系统数据通常缓存在内存中。因此，为了避免损坏磁盘上的数据，务必先卸载可移动驱动器，然后再拔下它们。卸载过程会在释放驱动器之前同步数据，以确保数据完整性。

​使用 `umount` 命令来卸载设备，该命令需要挂载点作为参数。

```bash
[root@localhost ~]\# umount /mnt/data
```

​正在使用的文件系统将无法成功卸载，若要成功执行上述命令，则所有进程都需要停止访问卸载点下的数据。

​使用 `lsof` 命令可以列出所给目录中所有打开的文件以及访问它们的进程。

```bash
[root@localhost ~]\# lsof /root
COMMAND    PID USER   FD   TYPE DEVICE SIZE/OFF     NODE NAME
bash      3082 root  cwd    DIR  259,3      219 33575041 /root
sftp-serv 3123 root  cwd    DIR  259,3      219 33575041 /root
sftp-serv 3123 root    8r   DIR  259,3      219 33575041 /root
lsof      3794 root  cwd    DIR  259,3      219 33575041 /root
lsof      3795 root  cwd    DIR  259,3      219 33575041 /root
```

> 常见的无法卸载的原因是当前命令行的工作目录处于挂载点或其子目录中，将工作目录切换到挂载点之外即可解决此问题。

## 查找系统中的文件

### 搜索文件

​使用 `locate` 和 `find` 命令进行文件系统层次结构中搜索文件的操作。

- `locate` 命令搜索预生成索引中的文件名或文件路径，并即使返回结果；
- `find` 命令通过爬取整个文件系统层次结构来实时搜索文件。

### 根据名称查找文件

​使用 `locate` 命令搜索文件会比较快速，因为它是从 *mlocate* 数据库中查找的信息，但是该数据库是每日自动更新的而不是实时更新的，*root* 用户可以通过命令 `updatedb` 来强制更新数据库。

```bash
[root@localhost ~]\# updatedb
```

​普通用户的搜索结果会根据用户的访问权限作出限制。

```bash
[root@localhost ~]\# locate Desktop # 子串匹配
/home/student/Desktop
/usr/lib/python3.6/site-packages/xdg/DesktopEntry.py
/usr/lib/python3.6/site-packages/xdg/__pycache__/DesktopEntry.cpython-36.opt-1.pyc
/usr/lib/python3.6/site-packages/xdg/__pycache__/DesktopEntry.cpython-36.pyc
/usr/lib64/girepository-1.0/GDesktopEnums-3.0.typelib
......
```

​使用 *-i* 选项来忽略大小写限制，*-n* 选项来限制命令返回的搜索结果数量：

```bash
[root@localhost ~]\# locate -i -n 5 messages 
/usr/lib/locale/C.utf8/LC_MESSAGES
/usr/lib/locale/C.utf8/LC_MESSAGES/SYS_LC_MESSAGES
/usr/lib/locale/en_AG/LC_MESSAGES
/usr/lib/locale/en_AG/LC_MESSAGES/SYS_LC_MESSAGES
/usr/lib/locale/en_AU/LC_MESSAGES
```

### 实时搜索文件

​使用 `find` 命令实时遍历整个文件系统来查找文件。它比 `locate` 更慢得到搜索结果但是会更准确，它不仅支持搜索文件名，还支持根据文件的权限、类型、大小或修改时间来搜索文件。

​`find` 命令查看文件及目录的权限与执行该命令的用户保持一致。

```bash
find [options] [path...] [expression]	# find 命令格式
# 默认 path 为当前目录，默认 expression 为 -print（即打印搜索结果）
```

#### 按照文件名搜索文件

```bash
[root@localhost ~]\# find / -name ssh_config
/etc/ssh/ssh_config
[root@localhost ~]\# find /home/student -name "*.txt" | head -n 5	# 使用通配符
/home/student/.mozilla/firefox/uf3n2m6z.default-default/pkcs11.txt
/home/student/.mozilla/firefox/uf3n2m6z.default-default/SiteSecurityServiceState.txt
/home/student/.mozilla/firefox/uf3n2m6z.default-default/SecurityPreloadState.txt
/home/student/.mozilla/firefox/uf3n2m6z.default-default/TRRBlacklist.txt
/home/student/.mozilla/firefox/uf3n2m6z.default-default/AlternateServices.txt
[root@localhost ~]\# find /usr/share -iname "*messages" | head -n 3	# iname 忽略用户名大小写
/usr/share/locale/af/LC_MESSAGES
/usr/share/locale/az/LC_MESSAGES
/usr/share/locale/bg/LC_MESSAGES
```

#### 根据所有权或权限搜索文件

```bash
[root@localhost ~]\# find /home/student/.pki  -user student	# 按用户搜索
/home/student/.pki
/home/student/.pki/nssdb
[root@localhost ~]\# find / -group cdrom	# 按用户组搜索
/dev/sr0
/dev/sg0
......
[root@localhost ~]\# find /etc/ssh/ssh_config -uid 0	# 按用户 id 搜索
/etc/ssh/ssh_config
[root@localhost ~]\# find / -user root -gid 12	# 用户和用户组（mail）搭配使用
/var/spool/mail
```

​使用 *-perm* 按照特定权限进行搜索，使用数字表示法，详细情况可以参考前面有关章节的内容。*-perm* 选项的参数的前面可以加上 */* 或 *\-* 符号。其中 */* 代表匹配 *u*,*g*,*o* 中的至少一个存在才匹配，*-* 代表 *u*,*g*,*o* 需要都存在才匹配。

```bash
[root@localhost ~]\# find -perm 764	# 匹配用户具有 rwx 权限，组成员具有 rw 权限以及其他人只有读权限的文件
[root@localhost ~]\# find -perm -324	# 匹配用户至少拥有 wx 权限，组成员至少拥有 w 权限，其他成员至少具有 r 权限的文件
[root@localhost ~]\# find -perm /442	# 匹配用户具有 r 权限或者组成员具有 r 或者其他人具有 w 权限的文件
[root@localhost ~]\# find -perm -004	# 匹配其他人至少拥有 r 权限的文件
```

#### 根据大小搜索文件

```bash
# 1. 搜索大小为 10 兆字节（向上取整）的文件
[root@localhost ~]\# find /boot -size 10M
/boot/vmlinuz-4.18.0-240.el8.x86_64	# 实际 9.1M
/boot/vmlinuz-0-rescue-6651ee1da4824a37975cd5245c8cb076	# 实际 9.1M
# 2. 搜索大小超过 10 兆字节的文件
[root@localhost ~]\# find /boot -size +10M
/boot/initramfs-4.18.0-240.el8.x86_64.img
/boot/initramfs-0-rescue-6651ee1da4824a37975cd5245c8cb076.img
/boot/initramfs-4.18.0-240.el8.x86_64kdump.img
# 3. 搜索大小不超过 10 兆字节的文件
[root@localhost ~]\# find /boot -size -10M
/boot
/boot/efi
/boot/efi/EFI
/boot/efi/EFI/redhat
......
```

#### 根据修改时间搜索文件

```bash
# 1. 搜索 120min 前修改的文件（向下舍入）【119min - 120min】
[root@localhost ~]\# find / -mmin 120
# 2. 搜索在 200min+ 前修改过的文件
[root@localhost ~]\# find / -mmin +120
# 3. 搜索过去 150min 内修改过的文件
[root@localhost ~]\# find / -mmin -150
```

#### 根据文件类型搜索文件

​不同文件类型的标志如下：

- *f*，普通文件
- *d*，目录
- *l*，软链接
- *b*，块设备

```bash
[root@localhost ~]\# find /dev -type b
/dev/sr0
/dev/nvme0n1p3
/dev/nvme0n1p2
/dev/nvme0n1p1
/dev/nvme0n1
```
