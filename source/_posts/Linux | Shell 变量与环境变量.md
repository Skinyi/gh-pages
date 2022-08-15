---
title: Shell 变量与环境变量
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

## Shell 变量

### 使用 Shell 变量

​Shell 变量可以帮助运行命令或修改 shell 的行为，shell 变量也可以被导出为环境变量，它会在程序启动时自动复制到从该 shell 运行的程序中。

​Shell 变量对于特定 shell 会话是唯一的，每个 shell 都有自己的一组 shell 变量值。

#### 使用 `echo` 命令输出一行文本内容

```bash
echo [SHORT_OPTION]... string...
# 常见参数：
-n, # 不要输出跟随的换行符
-e, # 解释转义字符
-E, # 不解释转义字符（默认）
```

​Shell 变量名称可以包含大写或小写字母、数字和下划线字符\_。用美元符号 \$ 可以读取设置的 shell 变量的值。如下例所示：

```bash
[root@localhost ~]\# count=250    # 设置变量，注意等号左右别加多余的空格字符
[root@localhost ~]\# echo count    # 不加 $ 符号只会将原文当文本输出
count
[root@localhost ~]\# echo $count    # 加上 $ 符号会将 count 视作变量，且此命令也会输出其值
250
```

#### 使用花括号来保护变量

```bash
[root@localhost ~]\# echo something $countx    # 找不到变量不会导致错误，而是不返回任何内容
something
[root@localhost ~]\# echo something ${count}x
something 250x
```

### 使用 `set` 命令列出当前设置的所有 Shell 变量

​`set` 命令由于打印当前终端设置的所有 shell 变量和 shell 函数。

```bash
[root@localhost ~]\# set | more
BASH=/bin/bash
BASHOPTS=checkwinsize:cmdhist:complete_fullquote:expand_aliases:extglob:e
xtquote:force_fignore:histappend:interactive_comments:login_shell:progcom
p:promptvars:sourcepath
# 省略大部分输出
```

### 使用 Shell 变量配置 Bash

​一些 shell 变量在 bash 启动时设置，但可以进行修改来调整 shell 的行为。如：\$PATH、\$PS1、\$HISSIZE 等。

## 环境变量

​Shell 提供了一个环境包括有关文件系统上当前工作目录的信息、传递给程序的命令行选项，以及环境变量的值。程序可以使用这些环境变量来更改其行为或其默认设置。 

​不是环境变量的 shell 变量只能由 shell 使用，环境变量可以由 shell 以及从该 shell 启动的程序使用。

### 设置环境变量

```bash
[root@localhost ~]\# RHEL_VERSION=8.3    # 创建 shell 变量
[root@localhost ~]\# export RHEL_VERSION    # 将其导出为环境变量
[root@localhost ~]\# export RHEL_VERSION=    # 在一行中完成上面两步
[root@localhost ~]\# echo $RHEL_VERSION    # 输出环境变量的内容

[root@localhost ~]\# 
```

### 使用 `env` 命令列出 shell 的所有环境变量

​env 修改环境变量并运行程序。使用 `export -p` 也可以打印所有的 shell 变量，不过每条输出前面都会加上："declare -x "。

```bash
env [OPTION]... [-] [NAME=VALUE]... [COMMAND [ARG]...]    # 修改环境变量并运行程序
# 常见选项：
-i, --ignore-environment # 使用空的环境运行程序
-0, --null # 使用空字符结束输出，而不是换行符
-u, --unset=NAME # 从环境中移除环境变量 NAME
-c, --chdir=DIR # 修改工作目录到 DIR
-S, --split-string=S # 将 S 拆分为多个参数，用来传递多个参数
# -S 在脚本中的使用举例，如想执行 1.pl 这个 perl 脚本,其第一行的写法为：
#!/user/bin/env -S perl -w -T 
# 将会执行 perl -w -T 1.pl
# 没有 -S 的话则会提示：
/usr/bin/env: 'perl -w -T'：No such file or directory
```

```bash
[root@localhost ~]\# env    # env 不加参数则会列出所有设置的环境变量
# 省略部分输出
RHEL_VERSION=
SSH_TTY=/dev/pts/1
MAIL=/var/spool/mail/root
TERM=xterm
SHELL=/bin/bash
# 省略部分输出
```

### 保存环境变量以长期使用

​如果要修改整个系统的环境变量，则建议在 /etc/profile.d 目录下添加自己的 .sh 结尾的文件（需要 root 权限），如果只是希望修改自己用户的环境变量，则修改自己的 ~/.bashrc 文件。

### 取消设置变量与取消导出环境变量

​使用 `export -n` 命令取消导出的环境变量（但其还是 shell 变量）：

```bash
[root@localhost ~]\# export RHEL_VERSION=8.3
[root@localhost ~]\# env | grep RHEL
RHEL_VERSION=8.3
[root@localhost ~]\# export -n RHEL_VERSION
[root@localhost ~]\# env | grep RHEL
[root@localhost ~]\# echo $RHEL_VERSION
8.3
```

​使用 `unset` 命令取消设置变量以及环境变量：

```bash
[root@localhost ~]\# export RHEL_VERSION
[root@localhost ~]\# env | grep RHEL
RHEL_VERSION=8.3
[root@localhost ~]\# unset RHEL_VERSION
[root@localhost ~]\# env | grep RHEL
[root@localhost ~]\# echo $RHEL_VERSION

[root@localhost ~]\# 
```
