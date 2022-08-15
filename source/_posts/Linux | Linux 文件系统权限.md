---
title: Linux 文件系统权限
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

## Linux 文件权限类别

​对于一个 linux 系统上的一般文件，其访问权限的具体内容如下：

<table>
    <tr>
        <th>例子</th>
        <th>用户类别</th>
        <th>具体权限</th>
    </tr>
    <tr>
        <td rowspan="9">sample.sh</td>
        <td rowspan="3">u: 该文件的属主，通常是创建者</td>
        <td>r: 读取权限，可以读取文件内容</td>
    </tr>
    <tr>
        <td>w: 修改权限，可以更改文件内容</td>
    </tr>
    <tr>
        <td>x: 执行权限，可以做为可执行文件执行</td>
    </tr>
    <tr>
        <td rowspan="3">g: 该文件属主所处的主要组</td>
        <td>r: 读取权限，可以读取文件内容</td>
    </tr>
    <tr>
        <td>w: 修改权限，可以更改文件内容</td>
    </tr>
    <tr>
        <td>x: 执行权限，可以做为可执行文件执行</td>
    </tr>
    <tr>
        <td rowspan="3">o: 其他用户以及他们的用户组</td>
        <td>r: 读取权限，可以读取文件内容</td>
    </tr>
    <tr>
        <td>w: 修改权限，可以更改文件内容</td>
    </tr>
    <tr>
        <td>x: 执行权限，可以做为可执行文件执行</td>
    </tr>
</table>

​对于目录文件，其权限标识的具体内容如下：

| 权限      | 作用                                       |
| ------- | ---------------------------------------- |
| r: 读取权限 | 可以列出目录的内容                                |
| w: 写入权限 | 可以创建或删除目录中的任一文件                          |
| x: 执行权限 | 该目录可以成为当前工作目录（可以 ch 到），但要读取权限才能查看该目录下的文件 |

## 修改 Linux 文件权限

### 使用 `chmod` 命令修改文件权限

```bash
chmod [OPTION]... MODE[,MODE]... FILE... # 以符号形式修改权限
chmod [OPTION]... OCTAL-MODE FILE... # 以数字形式修改权限
chmod [OPTION]... --reference=RFILE FILE... # 将文件的权限修改的和 RFILE 一样
# 常见选项：
-R, --recursive # 递归修改目录和文件
-v, --verbose # 为每个处理的文件输出诊断信息
-c, --changes # 类似 -v 选项，但是仅在文件更改成功时报告
-f, --silent, --quiet # 忽略大多数错误信息
```

#### 符号法

​可以使用 u、g、o、a 字母分别代表文件属主、属主所在组、其他用户及组、所有用户及组，对应的动作 +、-、=

分别代表添加、删除、精确设置，r、w、x 代表所要设置的权限，设置多个用户类别时用逗号隔开，即如下面的例子所示：

```bash
chmod u+wx,o-rx executable.sh # 属主增加写入和执行权限，其他用户删除读和执行权限
```

#### 数字法

​让 r、w、x 权限分别对应数字 4、2、1，由这三个数字任意组合相加得到一个 0-7 的结果，将这个结果从左到右三位分别代表 u、g、o 对应的权限再组合为一个三位数（最高位可以为零），此方法适用于精确设置文件的权限，即如下面的例子所示：

```bash
chmod 777 executable.sh # 将此文件设置为所有用户可读写、执行。
```

​第三种形式较易理解，不再赘述。

### 使用 `chown` 修改文件所属用户或组

​只有 root 用户可以更改文件的所属用户，文件所有者和 root 可以更改文件的所属用户（前者只能改到自己在的组里，后者任意）。

```bash
chown [OPTION]... [OWNER][:[GROUP] FILE... # 修改文件所属用户和组
chown [OPTION]... --reference=RFILE FILE... # 参照 RFILE 修改文件所属用户和组
# 常见选项：
-R, --recursive # 递归修改目录和文件
-v, --verbose # 为每个处理的文件输出诊断信息
-c, --changes # 类似 -v 选项，但是仅在文件更改成功时报告
-f, --silent, --quiet # 忽略大多数错误信息
```

```bash
chown -R student:student /opt/dir/ # 将 /opt/dir 目录的所有者和组改为 student 和 student 组
chown :wheel /opt/file # 作用等同于下条命令
chgrp wheel /opt/file # 将 /opt/file 文件的所有组改为 wheel 组
```

### 特殊权限

| 特殊权限       | 对文件的影响                     | 对目录的影响                                           |
| ---------- | -------------------------- | ------------------------------------------------ |
| u+s(suid)  | 以拥有文件的用户身份，而不是以运行文件的身份执行文件 | 无影响                                              |
| g+s(sgid)  | 以拥有该文件的组身份执行文件             | 在目录中最新创建的文件将其组所有者设置为与目录的组所有者相匹配                  |
| o+t(stiky) | 无影响                        | 对目录具有写入访问权限的用户仅可以删除其所拥有的文件，而无法删除或强制保存到其他用户所拥有的文件 |

​示例：

```bash
[root@localhost ~]\# ll /usr/bin/passwd
-rwsr-xr-x. 1 root root 33544 Dec 14  2019 /usr/bin/passwd # u.x->s
[root@localhost ~]\# ll -d /run/log/journal
drwxr-sr-x. 3 root systemd-journal 60 Apr 15 16:47 /run/log/journal # g.x->s
[root@localhost ~]\# ll /usr/bin/locate
-rwx--s--x. 1 root slocate 47128 Aug 12  2018 /usr/bin/locate # g.x->s
[root@localhost ~]\# ll -d /tmp 
drwxrwxrwt. 22 root root 4096 Apr 23 20:10 /tmp # o.x->t
```

#### 设置特殊权限的方法

- 符号法：u+s、g+s、o+t；
- 数值法：在原来三位的基础上在左边增加一位：setuid=4、setgid=2、sticky=1；

### 创建的新文件的默认权限

​两个因素影响创建的新文件（含目录）的初始权限：一是文件的类型（常规文件还是目录），二是当前的 umask。对于因素一，若是创建的新目录，操作系统默认首先为其分配八进制权限 0777（drwxrwxrwx）；如果是常规文件，操作系统默认首先为其分配八进制权限 0666（-rw-rw-rw-），因此可执行文件创建后需手动赋予执行权限才能够被执行。对于因素二，shell 内置了 umask 机制来进一步限制创建的新文件的权限，使用不带参数的 `umask` 可以查看当前 shell 设置的 umask 值，其内容为一个八进制位掩码，每位的数字代表从当前用户的默认设置中删掉某种权限：

```bash
[root@localhost ~]\# umask
0022 # 代表 go-w，即在默认的基础上去掉同组用户和其它用户的写入权限
```

```bash
umask [-p] [-S] [MODE] # 显示或设置文件模式掩码
# 常见选项：
-p, # 输出的具体掩码前面加上 umask，即可以作为命令输入复用
-S, # 使用符号模式
```

​使用 `umask` 可以临时更改当前 shell 创建文件时的掩码，想永久更改需在 bashrc 和 profile 文件中覆盖。
