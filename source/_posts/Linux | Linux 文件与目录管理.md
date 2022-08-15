---
title: Linux 文件与目录管理
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

## 文件目录操作

### 使用 `mkdir` 创建目录

```bash
mkdir [OPTION]... DIRECTORY...    # 如果不存在则创建目录
# 常见可选选项
-m, --mode=MODE # 设置文件读写执行权限（跟 chmod 以数字设置文件权限的方法一样）
-v, --verbose # 打印创建的每一个目录的信息
-p, --parents # 递归创建每层目录，即如果父目录不存在则创建好父目录后再创建自己
```

### 使用 `rmdir` 删除空的目录

```bash
rmdir [OPTION]... DIRECTORY... # 要删除的目录必须为空
# 常见选项
-p, --parents # 递归删除
-v, --verbose # 打印程序执行过程
--ignore-fail-on-non-empty # 忽略目录非空错误
```

### 使用 `cd` 切换工作目录

```bash
cd [-L|[-P [-e]] [-@]] [DIR]    # 切换 shell 工作目录到 DIR（默认未当前用户家目录，可省略）
# 常见选项
-P，# 符号链接直接转到物理目录结构中，例：cd -L /bin/ && pwd 输出 /usr/bin
```

​常见且典型的例子：

```bash
[root@localhost cups]\# cd && pwd    # 不跟参数默认切换到家目录
/root
[root@localhost home]\# cd ~ && pwd    # cd ~ 切换到家目录
/root
[root@localhost home]\# cd -    # cd - 为返回旧的工作目录
/root
[root@localhost ~]\# ls /etc/    # 省略输出
[root@localhost ~]\# cd !$    # cd !$ 将上条命令的参数作为 cd 的参数
cd /etc/
```

> 🟢 想玩点花的😎：`touch 1.txt && rm -rf !$`，如果你不巧是在执行了以某个存在的文件或目录作为参数的命令之后执行的这条语句，那么你刚才操作的那个文件或目录将不幸被删除😢！人生忠告：在删除文件前一定要确保知道你在干什么。一个例子：`rm -rf $VAR_NOT_DEFINED/*`，这个例子中使用了一个环境变量 `VAR_NOT_DEFINED`，但该变量未在系统中定义，故原命令就相当于 `rm -rf /*`，根目录下的所有文件和目录将不保😂。一定切记：*删东西需谨慎，验证后再玩花的*。献上我差点中招的例子：

```bash
[root@localhost ~]\# cd /etc
[root@localhost etc]\# touch 1.txt && rm -f !$    # 一条语句 != 一条命令，幸好我没指定 -r
touch 1.txt && rm -f /etc
rm: cannot remove '/etc': Is a directory
```

### 列出目录里的内容

```bash
ls [OPTION]... [FILE]...
# 常见可选选项：
-a, --all # 列出所有文件（包括 ./、../以及隐藏文件）
-A, --almost-all # 列出所有文件（包括隐藏文件）
--color[=WHEN] # 用颜色渲染输出：always 总是（默认），auto 自动, never 永不
-r, # 将文件以相反次序显示（原来以英文字母次序排序）
-t, # 将文件依建立时间之先后次序列出
-R, # 递归显示子目录的内容
-l, # 一行展示一个文件（包括文件夹），更加详细
    
ll [OPTION]... [FILE]...    # ll 实际上是 ls -l --color=auto 的别名，有些发行版可能不会默认设置
# 例子
[root@localhost ~]\# ll /home
total 0
drwx------. 3 student student 99 Apr 11 00:22 student
# 可选选项
-h, # 人类友好方式显示文件大小
-d，# 列出目录或文件自己本身的属性
```

​ `ls -l` 命令执行结果的各个部分的意义如下表所示：

| 部分  | 意义  |
| --- | --- |
| drwx------. | 文件类型读写执行权限属性标识，其中第一个字母代表文件类型，`-` 代表普通文件，`d` 代表目录，`l` 符号链接，`b` 块设备文件，`c` 字符设备文件，`c` 套接字文件，接下来每三位分别代表文件所有者、所有者处于的组、其余用户的读取执行权限：`-` 代表无权限，`r` 代表有读取权限，`w` 代表有写入权限，`x` 代表有执行该文件的权限 |
| 3   | 链接数，表示有多少个文件链接到inode号码，普通文件为 1，目录的话是其包含的文件或目录的个数 |
| student | 该文件或目录的拥有者 |
| student | 该文件拥有者的分组 |
| 99  | 文件大小，默认用 byte 来表示 |
| Apr 11 00:22 | 文件最后一次修改的时间，格式：*月-日-时间* |
| student | 文件名 |

### 使用 `mv` 命令重命名或移动文件或目录

```bash
mv cp [OPTION]... [-T] SOURCE DEST    # 重命名|产生文件的不同名的副本
mv cp [OPTION]... SOURCE... DIRECTORY    # 将所有 SOURCE 参数代表的文件移动|复制到 DIRECTORY 目录中
mv cp [OPTION]... -t DIRECTORY SOURCE...    # 传参顺序改变，更容易理解
# 常见参数
--backup[=CONTROL]    # 备份已存在的目标文件，通过选项传参决定：none|off 不备份，numbered|t 产生有编号的备份文件 existing|nil 如果已存在有编号的备份则编号备份，否则不编号，simple|never 普通备份，也可以通过环境变量 $VERSION_CONTROL 决定备份类型
-b # 同 --backup，不过不接受参数
-S, --suffix=SUFFIX # 指定备份文件后缀名，默认 ~ 。
# -i, -n, -f 同时指定则仅最后一个生效
-f, --force # 不询问，直接覆盖移动
-i, --interactive # 在覆盖前询问
-n, --no-clobber # 不覆盖已存在的文件
--strip-trailing-slashes # 从源参数中移除尾随斜杠
-t, --target-directory=DIRECTORY # 将所有 SOURCE 参数代表的文件移动到 DIRECTORY 目录中
-T, --no-target-directory # 将 DESC 视作正常文件
-u, --update # 仅当源文件比目标文件新或目标文件丢失的时候移动
-v, --verbose # 输出操作的详细信息
```

### 使用 `cp` 命令复制文件或目录

​ `cp` 命令格式见 `mv`。

```bash
# 其它常见选项
--attributes-only # 不要复制文件数据，仅复制文件的属性
-R, -r, --recursive # 递归复制，即包括目录里的内容也复制
```

### 使用 `rm` 命令删除文件或目录

```bash
rm [OPTION]... [FILE]...
-f, --force # 忽略不存在的文件或参数报错，不提示
-i, # 删除前需确认操作
-I, # 删除三个以上目录或文件时，或者在递归删除时，仅提示一次
-r, -R, --recursive # 递归删除目录和其中的文件
-d, --dir # 删除空目录
-v, --verbose # 删除时输出操作信息
--, # 删除以 - 为开头的文件时使用
```

## 链接文件

### 了解 linux 文件底层机制

​Linux 下的文件除了文件数据内容（block 节点）本身之外，还包含对这些纯文件数据内容的管理信息，如：文件名、访问权限、文件的属主以及该文件的数据所对应的磁盘块等，这些管理信息称之为元数据（meta data)，元数据保存在文件的 inode 节点之中。

​可以通过 `stat` 命令查看一个文件的 `inode` 信息。

```bash
[root@localhost ~]\# stat /var/log/messages # linux 下 inode 是该文件系统中的某一文件的唯一标识
   File: /var/log/messages
   Size: 1378414       Blocks: 2696       IO Block: 4096   regular file
 Device: fd00h/64768d    Inode: 34218021    Links: 1
 Access: (0600/-rw-------)  Uid: (    0/    root)   Gid: (    0/    root)
Context: system_u:object_r:var_log_t:s0
 Access: 2021-04-14 18:41:03.906745879 +0800
 Modify: 2021-04-14 18:41:03.906745879 +0800
 Change: 2021-04-14 18:41:03.906745879 +0800
  Birth: -

## stat 显示文件或文件系统状态
stat [OPTIONS]... FILE...
# 常见选项：
-L, --dereference # 跟随链接
-f, --file-system # 显示文件系统状态而不是文件状态
-t, --terse # 以精小的方式打印信息（不常用）
```

​使用 `df -ih` 命令查看系统给每个文件系统的 inode 区域划分情况。

```bash
[root@localhost ~]\# df -ih
Filesystem            Inodes IUsed IFree IUse% Mounted on
devtmpfs                457K   426  456K    1% /dev
tmpfs                   464K     1  464K    1% /dev/shm
tmpfs                   464K   987  463K    1% /run
tmpfs                   464K    17  464K    1% /sys/fs/cgroup
/dev/mapper/rhel-root    18M  120K   18M    1% /
/dev/nvme0n1p1          512K   302  512K    1% /boot
tmpfs                   464K    20  464K    1% /run/user/42
tmpfs                   464K    32  464K    1% /run/user/0
```

> 某个文件系统的 inode 区域被使用完了就无法在该文件系统下创建文件了，即使该文件系统还有磁盘块可以存放文件数据。

​Linux 下的目录其实是一类特殊的文件，”目录“文件下保存着该目录下所有文件的文件名到 inode 号的对应关系，这里的每个对应关系被称为一个目录项（directory entry，即 dentry）。

### 硬链接（hard link）

​硬链接的实质是现有文件在文件目录树中的另一个入口，即分居于不同或相同目录下的指向同一个 inode 的 dentry，其文件内容也位于相同的磁盘数据库，具有相同的访问权限、属性等。硬链接可以理解为给已存在的文件取一个别名，只删除一个硬连接并不影响索引节点本身和其它的链接，只有当最后一个链接被删除后，文件的数据块及目录的链接才会被释放。也就是说，文件真正删除的条件是与之相关的所有硬连接文件均被删除。

​硬链接的优点是几乎不占磁盘空间，因为仅仅是在一个目录文件中增加了一个目录项的对应关系，但这一有点相较软连接并不明显，软链接占用的磁盘空间也很小。硬链接的缺点是：1. 不能跨文件系统创建硬链接；2. 不能对目录创建硬链接原因可以参照网络上的这篇[文章](https://www.jb51.net/LINUXjishu/513216_all.html)。

### 符号链接（symlink,symbolic link, 也称软链接）

​符号链接其本身也是一个文件，不过这个文件的内容是另一个文件的指针。当要查看某个软链接指向文件的内容时，系统会循着指针找出含有实际数据的目标文件，并读取其内容。

​软连接可以跨越文件系统指向另一个分区的文件，甚至可以跨越主机指向远程主机的一个文件，也可以指向目录文件。软连接文件的权限是 777，即所有权限都是开放的，且无法更改，但实际文件的权限保护机制仍然起作用。符号链接可以指向一个不存在的文件，即与目标文件处于断裂状态，硬链接不能。

### 使用链接的好处

#### 保持软件的兼容性

​在 RHEL8 操作系统中，`sh` 命令的二进制文件实际上是 `bash` 文件的一个软连接：

```bash
[root@localhost ~]\# ll /bin/sh
lrwxrwxrwx. 1 root root 4 Jun 23  2020 /bin/sh -> bash
```

​这样做的好处是方便系统向后兼容，因为有些较为早期的脚本文件其默认使用的解释器还默认是 `sh` ,而 `bash` 是后来 sh 的最成功的改进版本，为了使这些较为早期的脚本也能在 bash 上正常的执行，故将 sh 作为 bash 的一个链接，实现了不用更改脚本内容就可以使用 bash 来解释执行它们。

#### 方便使用

​如当我们在 /opt 目录下安装了大型软件后，可在 /usr/bin 目录下为其启动脚本或可执行文件创建链接，这样就可以实现不用修改环境变量，在命令行直接输入软链接的文件名执行就可以打开安装好的软件。

#### 维持旧的操作习惯

​比如在 SuSE 中，启动脚本的位置是放在 /etc/init.d 目录下，而在 RedHat 的发行版中，是放在 /etc/rc.d/init.d 目录下。为了避免因为从 SuSE 转换到 RedHat 系统而导致管理员找不到位置的情况，我们可以创建一个符号链接 /etc/init.d 使其指向 /etc/rc.d/init.d 即可。

```bash
[root@localhost ~]\# ll -d /etc/init.d
lrwxrwxrwx. 1 root root 11 Apr 17  2020 /etc/init.d -> rc.d/init.d
```

> RHEL8 版本中，原先的 service 机制（包括 init.d 控制启动项的机制）已经 systemd 机制取代了。但为了向后兼容，现版本还存在这些文件目录。

### 使用 `ln` 命令创建链接

```bash
ln [OPTION]... [-T] TARGET LINK_NAME # 为 TARGET 创建一个名为 LINK_NAME 的链接
ln [OPTION]... TARGET # 在当前工作目录中为 TARGET 创建一个链接
ls [OPTION]... TARGET... DIRECTORY # 在 DIRECTORY 中为每一个 TARGET 创建链接（默认硬链接，默认链接文件的文件名不允许在该目录中已存在）
ls [OPTION]... -t DIRECTORY TARGET... # 硬链接的目标文件必须存在
# 常见选项：
--backup=[CONTROL] # 为每个已存在的目标文件创建备份，详见 mv 的 --backup 选项
-b, # 类似 --backup 但不接受参数
-f, --force # 如果目标文件已经存在则删除它
-i, --interactive # 交互询问是否删除已存在的目标文件
-L, --logical # 断开符号链接文件和目标文件之间的链接
-n, --no-dereference # 将 LINK_NAME 视作正常文件如果它是目录的一个软连接
-P, --physical # 使硬链接直接指向符号链接
-s, --symbolic # 创建软连接而不是硬链接
-t, --target-directory=DIRECTORY # 指定在哪个目录下创建链接
-T, --no-target-directory # 将链接名总是视作正常文件
-v, --verbose # 打印每个创建好的链接文件名
```

相关例子如下：

```bash
# 创建练习文件 file.txt 并为其创建软链接和硬链接
[student@localhost ~]\$ touch file.txt && echo "this is an example file of links" > file.txt && ll
total 4
-rw-rw-r--. 1 student student 33 Apr 14 22:13 file.txt
[student@localhost ~]\$ ln -s file.txt symlink_file
[student@localhost ~]\$ ln file.txt hlink_file
[student@localhost ~]\$ ll
total 8
-rw-rw-r--. 2 student student 33 Apr 14 22:13 file.txt
-rw-rw-r--. 2 student student 33 Apr 14 22:13 hlink_file
lrwxrwxrwx. 1 student student  8 Apr 14 22:15 symlink_file -> file.txt
# 使用 stat 命令检查这三个文件
[student@localhost ~]\$ stat *
  File: file.txt
  Size: 33            Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 3252736     Links: 2
Access: (0664/-rw-rw-r--)  Uid: ( 1000/ student)   Gid: ( 1000/ student)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2021-04-14 22:13:47.856472747 +0800
Modify: 2021-04-14 22:13:47.857472747 +0800
Change: 2021-04-14 22:16:20.095481417 +0800
 Birth: -
  File: hlink_file
  Size: 33            Blocks: 8          IO Block: 4096   regular file
Device: fd00h/64768d    Inode: 3252736     Links: 2
Access: (0664/-rw-rw-r--)  Uid: ( 1000/ student)   Gid: ( 1000/ student)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2021-04-14 22:13:47.856472747 +0800
Modify: 2021-04-14 22:13:47.857472747 +0800
Change: 2021-04-14 22:16:20.095481417 +0800
 Birth: -
  File: symlink_file -> file.txt
  Size: 8             Blocks: 0          IO Block: 4096   symbolic link
Device: fd00h/64768d    Inode: 3252737     Links: 1
Access: (0777/lrwxrwxrwx)  Uid: ( 1000/ student)   Gid: ( 1000/ student)
Context: unconfined_u:object_r:user_home_t:s0
Access: 2021-04-14 22:16:23.155481591 +0800
Modify: 2021-04-14 22:15:43.072479308 +0800
Change: 2021-04-14 22:15:43.072479308 +0800
 Birth: -
```

​经过对比这三个文件可以发现：源文件和其硬链接文件共同指向了同一个 inode (Inode:3252736) ,硬链接数（Links）字段同为 2，表示有两个目录项指向该 inode，有多少个硬链接其值就为多少（普通文件必存在一个硬链接），它们的文件元数据是一样的；而软链接与源文件相比，其访问权限为 0777，且 inode 号也与源文件不同，其所占的文件数据块大小为0。

```bash
# 删除源文件后看看
[student@localhost ~]\$ rm file.txt && ll -i && cat hlink_file
total 4
3252736 -rw-rw-r--. 1 student student 33 Apr 14 22:13 hlink_file
3252737 lrwxrwxrwx. 1 student student  8 Apr 14 22:15 symlink_file -> file.txt
this is an example file of links
[student@localhost ~]\$ cat symlink_file 
cat: symlink_file: No such file or directory
```

​删除了源文件后还可以通过硬链接访问文件内容，但该文件的元数据的 Links 字段重新变回了 1（第三列），其 inode（第一列） 没有改变，软链接的记录会不断闪烁表明该软链接已经断裂失效，通过 cat 命令查看其内容也会提示没有该文件。

### 追随链接

​自从有了软链接，当要备份、复制或者移动目录或文件的时候，就会出现是否要”追随链接“的问题。如果是，则会操作链接所指的对象，如果不是，则仅仅操作链接本身。一些命令工具会出现是否追随的选项，如 `cp`：

```bash
# cp 命令关于链接的选项
-L, --dereference # 追随链接（复制链接所指向的目标）
-P, --no-dereference # 不追随链接（复制链接本身）
```
