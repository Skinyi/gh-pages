---
title: 归档和传输文件
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

## 进行文件的归档管理

​使用 `tar` 命令来将文件和目录归档到压缩文件中，还能提取现有的 *tar* 存档内容。*tar* 存档是一个结构化的文件数据序列，其中包含有关每个文件和索引的元数据，以便可以提取单个文件。该存档可以使用 gzip、bzip2 或 xz 压缩方式进行压缩。*tar*命令也可仅列出归档文件的内容而不用进行提取。

```bash
tar [OPTION...] [FILE...]	# 使用 tar 进行文件归档和存档
# 常见选项
## 一般操作选项（必选）：
-c, --create	# 创建一个新存档
-A, --catenate, --concatenate	# 追加文件到 tar 文件
-x, --extract	# 从现有存档文件进行提取
-t, --list		# 列出存档的目录
## 一般通用选项：
-v, --verbose	# 详细信息。显示存档或提取的文件有哪些
-f, --file=		# 文件名。此选项必须后接要使用或创建的存档的文件名
-p, --preserve-permissions	# 在提取存档时保留文件和目录的权限，而不去除 umask
## 压缩选项：
-z, --gzip		# 使用 gzip 压缩方式（.tar.gz）
-j, --bzip2		# 使用 bzip2 压缩方式（.tar.bz2），bzip2 的压缩率通常比 gzip 高
-J, --xz		# 使用 xz 压缩方式（.tar.xz），xz 的压缩率通常比 bzip2 更高
```

> 一般使用时的常用用法：tar \[c\|x\|t\]\[z\|j\|J\]vf 目标文件 [要归档的文件...]

### 归档文件和目录

​以下命令创建名为 *archive.tar* 的存档，其内容为用户主目录中的 file1、file2 和 file3：

```bash
[root@vmrhelskinyi ~]\# tar -cf archive.tar file1 file2 file3 
[root@vmrhelskinyi ~]\# tar -tf archive.tar
file1
file2
file3
```

### 列出存档的内容

```bash
[root@vmrhelskinyi ~]\# tar -tvf archive.tar	# 查看归档文件的详细信息 
-rw-r--r-- root/root         0 2021-09-01 09:18 file1
-rw-r--r-- root/root         0 2021-09-01 09:18 file2
-rw-r--r-- root/root         0 2021-09-01 09:18 file3
```

### 从存档中提取文件

​ *tar* 存档通常应当提取到空目录中，以确保它不会覆盖任何现有文件。使用 root 用户提取存档时，`tar` 命令会保留文件的原始用户和组所有权。如果普通用户使用 `tar` 提取文件，文件所有权将属于从存档中提取文件的用户。

​以下命令将把 *archive.tar* 归档中的文件提取到 /root/filebackup 目录中：

```bash
[root@vmrhelskinyi ~]\# mkdir /root/filebackup
[root@vmrhelskinyi ~]\# cd /root/filebackup/
[root@vmrhelskinyi filebackup]\# tar -xvf /root/archive.tar 
file1
file2
file3
```

​默认情况下，从文档中提取文件时，将从存档内容的权限中去除 *umask*。要保留存档文件的权限，可在提取存档时使用 *-p* 选项。

### 创建压缩了的归档文件

​最好是使用单个顶级目录，其中可包含其他的目录和文件，以通过有序的方式来简化文件的提取。

​以下命令创建了 */var/log* 目录下的文件的归档，并使用 xz 的方式进行压缩：

```bash
[root@vmrhelskinyi ~]\# tar cJvf log_backup.tar.xz /var/log	# tar 命令选项中的短线可省略
tar: Removing leading `/' from member names
/var/log/
/var/log/lastlog
/var/log/private/
......
```

​查看压缩了的归档包的内容（不强制使用压缩选项）：

```bash
[root@vmrhelskinyi ~]\# tar tvf /root/log_backup.tar.xz
drwxr-xr-x root/root         0 2021-09-01 09:13 var/log/
-rw-rw-r-- root/utmp    292292 2021-09-01 09:18 var/log/lastlog
drwx------ root/root         0 2021-06-03 07:05 var/log/private/
......
```

### 解压压缩了的归档文件

​以下命令将创建 */root/logbackup* 目录并将 *log_back.tar.xz* 中的内容解压到该目录下：

```bash
[root@vmrhelskinyi ~]\# mkdir logbackup && cd logbackup
[root@vmrhelskinyi logbackup]\# tar xvf ../log_backup.tar.xz 
var/log/
var/log/lastlog
var/log/private/
......
var/log/vmware-network.3.log
[root@vmrhelskinyi logbackup]\# tree -L 3
.
└── var
    └── log
        ├── anaconda
        ├── audit
        ├── boot.log
        ......
```

> gzip、bzip2 和 xz 也可单独用于压缩单个文件，它们对应的解压缩命令分别为 gunzip、bunzip2、unxz。

## 在系统之间安全的传输文件

### 使用 SECURE COPY 传输文件

​`scp` 命令是 OpenSSH 套件的一部分，它可将文件从远程系统复制到本地系统或从本地系统复制到远程系统。此过程需要进行身份验证，并在数据传输之前对其进行加密。

​远程主机的目标文件位置的一般格式为：*\[user@\]host:/path*。其中 user 为可选项，若不指定则默认使用当前的本地用户名。

​以下命令将 host 上的本地文件 /etc/yum.conf 和 /etc/hosts 文件复制到了远程主机 remotehost 上 remoteuser 用户的主目录：

```bash
[user@host ~]\$ scp /etc/yum.conf /etc/hosts remoteuser@remotehost:/home/remoteuser
remoteuser@remotehost\`s password: 
yum.conf								100%	813		0.8KB/s		00:00
hosts									100%	227		0.2KB/s		00:00
```

​以下命令将 remotehost 上的远程目录 /var/log 复制到了本地主机 host 上的 user 用户的主目录下的 tmp 文件夹中：

```bash
[user@host ~]\$ scp -r root@remotehost:/var/log /home/user/tmp	# -r 选项代表递归复制目录
root@remotehost\`s password: 
......
```

### 使用安全文件传输程序传输文件

​要以交互式方式从 SSH 服务器上上传或下载文件，可以使用安全文件传输程序 *sftp*。`sftp` 命令的会话使用安全身份验证机制，并将数据加密后再与 SSH 服务器来回传输。

​以下命令以 remoteuser 的身份连接到远程主机：

```bash
[user@host ~]\$ sftp remoteuser@remotehost
remoteuser@remotehost\`s password: 
connected to remotehost.
sftp> 
```

​ *sftp* 交互会话接受通用的与 Shell 命令相同的命令如：*ls*、*cd*、*mkdir*、*pwd*。使用 `put` 命令将文件上传到远程主机，使用 `get` 命令从远程主机上下载文件。使用 `exit` 命令可退出 `sftp` 会话。

​ *sftp* 会话连接成功后会自动进入远程用户的主目录，且默认假设使用 `put` 命令后跟的是本地文件系统上的文件。

​以下命令将本地系统的 /etc/hosts 文件上传到 remotehost 新建的目录 /home/remoteuser/hostbackup中：

```bash
sftp> mkdir hostbackup
sftp> cd hostbackup
sftp> put /etc/hosts
Uploading /etc/hosts to /home/remoteuser/hostbackup/hosts
/etc/hosts								100%	227		0.2KB/s		00:00
sftp> 
```

​以下命令将 remotehost 上的 /etc/yum.conf 下载到本地系统上的当前目录并退出：

```bash
sftp > get /etc/yum.conf
Fetching /etc/yum.conf to yum.conf
/etc/yum.conf							100%	813		0.8KB/s		00:00
sftp > exit
[user@host ~]\$
```

### 使用 LRZSZ 进行文件传输

​这个传输工具需要支持 XMODEM、YMODEM 或 ZMODEM 协议的远程连接客户端如 XShell 才能使用，更详细的介绍可见其官网：https://ohse.de/uwe/software/lrzsz.html。

​远程连接到远程主机后，使用 `rz` 命令接收从本地主机发送的文件，使用 `sz FILE` 将文件发送到本地主机。

## 在系统间安全地同步文件

​ *rsync* 是一个实现本地和远程文件或目录同步更新的程序，其通过比较两个文件或目录的差异进行最小化更改、传输。它也可以支持两个本地文件或目录的同步。

```bash
# rsync 命令的常见形式：
rsync [OPTION...] SRC DEST				# 用于本地同步
rsync [OPTION...] SRC [USER@]HOST:DEST	# 用于将本地数据备份到远程主机（ssh 协议认证）
rsync [OPTION...] SRC [USER@]HOST::DST	# 用于将本地数据备份到远程主机（rsync 协议认证）
rsync [OPTION...] [USER@]HOST:SRC DEST	# 用于将远程机器上的数据备份到本地主机（ssh 协议认证）
rsync [OPTION...] [USER@]HOST::SRC DEST # 用于将远程机器上的数据备份到本地主机（rsync协议认证）
# 常见选项：
-v, --verbose		# 显示详细信息
-r, --recursive		# 递归复制目录，同步目录时必须使用
-n, --dry-run		# 空运行，显示正常运行时的更改但不实际运行
-a, --archive		# 使用存档模式，相当于 -rlptgoD（保留软链接接但不保留硬链接）
-l, --links			# 同步软连接
-p, --perms			# 保留权限
-t, --times			# 保留时间戳
-g, --group			# 保留组所有权
-o, --owner			# 保留文件所有者
-D, --devices		# 同步设备文件
-H, --hard-links	# 保留硬链接
-z, --compress		# 传输过程中进行压缩
    --delete		# 删除目标路径中源路径没有的文件
    --exclude=PATTERN	# 排除匹配 PATTERN 的文件
```

### 本地数据的同步

​以下命令保持了 /var/log 目录的内容与 /tmp 目录的同步：

```bash
[root@host ~]\# rsync -av /var/log /tmp
recieving incremental file list
log/
log/README
......
log/tuned/tuned.log

sent 11,592,423 bytes  received 779 bytes  23,186,404.00 bytes/sec
total size is 11,586,755  speedup is 1.00
[root@host ~]\# ls /tmp
log ssh-RLjDdarkKiw1
```

​源目录的路径后面加上 */* 即可实现同步源目录的内容而不在目标路径下创建同名子目录。

### 与远程数据的同步

​要保留文件的所有权，需使用目标系统上的 root 用户。

​以下命令将本地的 /var/log 目录同步到 remotehost 系统上的 /tmp 目录中：

```bash
[root@host ~]\# rsync -av /var/log remotehost:/tmp	# 省略用户默认为 root
root@remotehost\`s password: 
receiving incremental file list
log/
log/README
......
log/tuned/tuned.log

sent 9783 bytes  received 290,576 bytes  85,816.86 bytes/sec
total size is 11,585,690  speedup is 38.57
```

​同样，remotehost 上的 /var/log 远程目录也可同步到 host 上的 /tmp 本地目录：

```bash
[root@host ~]\# rsync -av remotehost:/var/log /tmp
root@remotehost\`s password: 
receiving incremental file list
log/boot.log
log/dnf.librepo.log
......
log/dnf.log

sent 9783 bytes  received 290,576 bytes  85816.86 bytes/sec
total size is 11,585,690  speedup is 38.57
```
