---
title: 提高命令行生产效率
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

## 编写简单的 BASH 脚本

​Shell 脚本可以大幅提高工作效率，对处理重复、复杂的任务很有用。精通 Shell 脚本编写是 Linux 从业者的基本要求。

​Shell 脚本是一个可执行文件，其内容是一系列命令和逻辑的集合。

​Shell 脚本的第一行以 “#\!” 开头，后面跟为执行该脚本的内容时所需要的正确的命令解释器的绝对路径，如：

```bash
#!/bin/bash
```

​Shell 脚本必须被赋予执行权限，才能作为常规命令运行。使用 `chmod` 命令可为其添加执行权限，并且可能需要与 `chown` 命令组合来更改脚本的文件所有权，仅为脚本的目标用户授予执行权限。

​将脚本放在 shell 的 *PATH* 环境变量中所列出的某个目录下，则可以像其他命令那样单独用文件名来调用 shell 脚本。但应注意自己编写的脚本名需要区别于 linux 系统命令或已有的脚本文件。

### 编写脚本的两个须知点

#### 特殊字符的字面化处理

​一些字符或词语对 shell 来说有特殊含义，在使用这些字符和词语的时候如果只想用其字面值的话则需要以下三种方法来取消或转义对应特殊字符或词语。

​前缀转义字符来输出 # 号：

```bash
[root@localhost ~]\# echo #1 is here

[root@localhost ~]\# echo \#1 is here
\#1 is here    # md 格式冲突，请忽略输出结果中的转义符号，下同
```

使用单引号括起需要多次转义的语句而不用分别进行转义：

```bash
[root@localhost ~]\# echo \#1 is here, but where is \#2?
\#1 is here, but where is \#2?
[root@localhost ~]\# echo '#1 is here, but where is #2'
\#1 is here, but where is \#2
```

​使用双引号可以阻止通配和 shell 扩展，但依然允许命令和变量替换，单引号则进一步不允许命令和变量替换。问号也是一个需要防止扩展的元字符。

```bash
[root@localhost ~]\# var=$(hostname -s); echo $var    # 将命令的执行结果进行变量赋值： VAR=${COMMAND}
localhost
[root@localhost ~]\# echo "****** hostname is ${var} ******"
****** hostname is localhost ******
[root@localhost ~]\# echo Your username variable is \$USER.    # 对 $ 进行了转义、因而 $ 失去了原有作用
Your username variable is $USER.
[root@localhost ~]\# echo "Variable $var is evaluate to $(hostname -s)"
Variable localhost is evaluate to localhost
[root@localhost ~]\# echo 'Variable $var is evaluate to $(hostname -s)'
Variable $var is evaluate to $(hostname -s)
[root@localhost ~]\# echo "\"Hello,world and $var\""; echo '"Hello,world and $var"'
"Hello,world and localhost"
"Hello,world and $var"
[root@localhost ~]\# echo "'Hello,world and $var'"; echo \""Hello,world and $var"\"
'Hello,world and localhost'
"Hello,world and localhost"
```

#### 多使用 `echo` 命令来输出执行状态信息

​最好将普通提示信息输出至标准输出，而将错误信息输出至错误输出。

```bash
#!/bin/bash
......
echo "Something has done!";
......
echo "Error: Something went wrong and need to exit!" >&2;
......
```

### 脚本编写之循环

​Bash 的 *for* 循环结构使用以下语法：

```bash
for VARIABLE in LIST; do
COMMAND VARIABLE
done
```

​以下语句的输出结果是一样的：

```bash
# 语句一
[root@localhost ~]\# for HOST in host1 host2 host3; do echo $HOST; done
# 语句二
[root@localhost ~]\# for HOST in host{1,2,3}; do echo $HOST; done
# 语句三
[root@localhost ~]\# for HOST in host{1...3}; do echo $HOST; done
# 输出结果
host1
host2
host3
```

​查询安装的跟内核相关的包并输出其安装时间：

```bash
[root@localhost ~]\# for PKG in $(rpm -qa | grep "kernel"); \
do echo "$PKG was installed in $(date -d @$(rpm -q --qf "%{INSTALLTIME}\n" $PKG))"; \
done
kernel-tools-4.18.0-240.el8.x86_64 was installed in Thu Jun  3 07:08:19 CST 2021
kernel-core-4.18.0-240.el8.x86_64 was installed in Thu Jun  3 07:05:38 CST 2021
kernel-tools-libs-4.18.0-240.el8.x86_64 was installed in Thu Jun  3 07:04:20 CST 2021
abrt-addon-kerneloops-2.10.9-20.el8.x86_64 was installed in Thu Jun  3 07:08:21 CST 2021
kernel-modules-4.18.0-240.el8.x86_64 was installed in Thu Jun  3 07:06:03 CST 2021
kernel-devel-4.18.0-240.el8.x86_64 was installed in Thu Jun  3 07:10:07 CST 2021
kernel-4.18.0-240.el8.x86_64 was installed in Thu Jun  3 07:09:28 CST 2021
kernel-headers-4.18.0-240.el8.x86_64 was installed in Thu Jun  3 07:04:49 CST 2021
# date -d @时间戳    # 将某一时间戳（1970 年 01 月 01 日 08 时整起经过的秒数）转换成标准日期格式
```

​输出 2 - 10 之间的所有偶数： 

```bash
[root@localhost ~]\# for EVEN in $(seq 2 2 10); do echo "$EVEN"; done
2
4
6
8
10
# seq 命令：生成数字序列。seq 2 2 10，从 2 开始，每次增加 2，直至 10。
```

​一个疑问：

```bash
# 这是为啥？
[root@localhost ~]\# for VAR in var{$(seq -s , 0 1 5)}; do echo $VAR; done
var{0,1,2,3,4,5}
[root@localhost ~]\# for NUM in $(seq 0 1 5); do echo var$NUM; done 
var0
var1
var2
var3
var4
var5
```

​Shell 脚本在处理完自己的所有内容后，脚本会退出到调用它的进程。若需在脚本执行完成之前退出执行，可以通过在脚本中使用 `exit` 命令来实现。可以为 exit 命令指定一个参数来表明整个操作的执行状态，其中 *0* 代表成功执行，其他非零值均代表存在错误。可以使用不同的非零值来代表不同的错误类型，退出后，该命令、脚本进程的执行结果将传回父进程，可以访问变量 *？* 来获取子进程的执行结果。如：

```bash
[root@localhost ~]\# rmdir DirThatNotExisted
rmdir: failed to remove 'DirThatNotExisted': No such file or directory
[root@localhost ~]\# echo $?    # ？变量的值是随时根据子进程的执行结果改变的
1
```

> 如果不使用任何参数调用 `exit` 命令，那么脚本将退出并且将最后执行的命令的退出状态传递给父进程。

> 为确保脚本不会由于意外情况轻易中断，一种良好的做法是不要进行与输入有关的假设，如命令行参数、用户输入、命令替换、变量扩展及文件名扩展。可以使用 Bash 的 test 命令来执行完整性检查。

​与所有命令一样，`test` 命令会在完成后生成一个退出代码，该退出代码存储为值 *\$?*。要查看测试的结论，请在执行 `test` 命令后立即显示 *\$?* 的值。

### 常用运算符

​Bash 测试命令语法：*\[\<TESTEXPRESSION\>\]*，较新的扩展测试命令：*\[\<TESTEXPRESSION\>\]*（Bash 2.02 及更高版本可用，可提供诸如通配模式匹配和正则表达式模式匹配等功能）。

#### 算数比较运算符

| 比较运算符            | 描述   | 示例          |
| ---------------- | ---- | ----------- |
| num1 *\-eq* num2 | 等于   | [ 1 -eq 1 ] |
| num1 *\-ne* num2 | 不等于  | [ 1 -ne 1 ] |
| num1 *\-lt* num2 | 小于   | [ 1 -lt 2 ] |
| num1 *\-le* num2 | 小于等于 | [ 2 -le 2 ] |
| num1 *\-gt* num2 | 大于   | [ 8 -gt 2 ] |
| num1 *-ge* num2  | 大于等于 | [ 2 -gt 2 ] |

```bash
[root@localhost ~]\# [ 1 -eq 1 ]; echo $?    # -eq：等于，1 等于 1 ？
0                                            # true
[root@localhost ~]\# [ 1 -ne 1 ]; echo $?    # -ne：不等于，1 不等于 1 ？
1                                            # false
[root@localhost ~]\# [ 8 -gt 2 ]; echo $?    # -gt：大于，8 大于 2 ？
0                                            # true
[root@localhost ~]\# [ 2 -ge 2 ]; echo $?    # -ge：大于等于，2 大于等于 2 ？
0                                            # true
[root@localhost ~]\# [ 2 -lt 2 ]; echo $?    # -lt：小于，2 小于 2 ？
1                                            # false
[root@localhost ~]\# [ 1 -lt 2 ]; echo $?    # -lt：小于，1 小于 2 ？
0                                            # true
[root@localhost ~]\# [ 1 -le 2 ]; echo $?    # -le：小于等于，1 小于等于 2 ？
0                                            # true
```

#### 字符串比较运算符

> 注意引号的使用，这是防止空格扰乱代码的好方法。*[ <EXPRESSION> ]* 语句中两侧的空格以及运算符两侧的空格是必须的。

| 比较运算符                | 描述                         | 示例                                 |
| -------------------- | -------------------------- | ---------------------------------- |
| *-z* string          | 若 string 长度为 0，则为真         | [ -z "\$MyVariable" ]              |
| *-n* string          | 若 string 长度为 0，则为真         | [ -n "\$MyVariable" ]              |
| string1 *=* string2  | 若 string1 与 string2 相同，则为真 | [ "\$MyVariable" = "Some words" ]  |
| string1 *!=* string2 | 若 string1 与 string2 不同，则为真 | [ "\$MyVariable" != "Some words" ] |
| string1 *==* string2 | 若 string1 与 string2 相同，则为真 | [ "\$MyVariable" == "Some words" ] |

```bash
[root@localhost ~]\# [ abc = abc ]; echo $?
0
[root@localhost ~]\# [ abc == def ]; echo $?
1
[root@localhost ~]\# [ abc != def ]; echo $?
0
[root@localhost ~]\# [ -z "" ]; echo $?
0
[root@localhost ~]\# [ -n "" ]; echo $?
1
```

#### 文件比较运算符

| 比较运算符                     | 描述                            | 示例                                              |
| ------------------------- | ----------------------------- | ----------------------------------------------- |
| *-e* filename             | 若 filename 存在，则为真             | [ -e /var/log/syslog ]                          |
| *-d* filename             | 若 filename 存在，则为真             | [ -d /tmp/mydir ]                               |
| *-f* filename             | 若 filenamee 为常规文件，则为真         | [ -f /usr/bin/grep ]                            |
| *-L* filename             | 若 filename 为符号链接，则为真          | [ -L /usr/bin/grep ]                            |
| *-r* filename             | 若 filename 为可读，则为真            | [ -r /var/log/syslog ]                          |
| *-w* filename             | 若 filename 为可写，则为真            | [ -w /root/log.txt ]                            |
| *-x* filename             | 若 filename 为可执行，则为真           | [ -x /usr/bin/grep ]                            |
| filename1 *-nt* filename2 | 若 filename1 比 filename2 新，则为真 | [ /tmp/install/etc/services -nt /etc/services ] |
| filename1 *-ot* filename2 | 若 filename1 比 filename2 旧，则为真 | [ /boot/bzImage -ot arch/i386/boot/bzImage ]    |

```bash
[root@localhost ~]\# [ -e /var/log/syslog ]; echo $?
1
[root@localhost ~]\# [ -d /tmp/mydir ]; echo $?
1
[root@localhost ~]\# [ -f /usr/bin/grep ]; echo $?
0
[root@localhost ~]\# [ -L /usr/bin/grep ]; echo $?
1
[root@localhost ~]\# [ -r /var/log/syslog ]; echo $?
1
[root@localhost ~]\# [ -w /root/log.txt ]; echo $?
1
[root@localhost ~]\# [ -x /usr/bin/grep ]; echo $?
0
[root@localhost ~]\# [ /tmp/install/etc/services -nt /etc/services ]; echo $?
1
[root@localhost ~]\# [ /boot/bzImage -ot arch/i386/boot/bzImage ]; echo $?
1
```

### 脚本编写之条件

​Bash 中最简单的 *if* 条件判断语句的格式如下：

```bash
if <CONDITION>; then
    <STATEMENT>
    ...
fi
```

​以下代码段演示使用 if 语句判断若 psacct 服务未处于活动状态则启动该服务：

```bash
[root@localhost ~]\# systemctl is-active psacct > /dev/null 2>&1
[root@localhost ~]\# if [ $? -ne 0 ]; then sudo systemctl start psacct; fi
[root@localhost ~]\# 
```

​两条线的 *if* 条件判断语句：

```bash
if <CONDITION>; then
    <STATEMENT>
    ...
else
    <STATEMENT>
    ...
fi
```

​以下代码段演示了使用 *if/then/else* 语句来启动 *psacct* 服务（若其未处于活动状态）和停止该服务（如果其处于活动状态）。

```bash
[root@localhost ~]\# systemctl is-active psacct > /dev/null 2>&1
[root@localhost ~]\# if [ $? -ne 0 ]; then \
> sudo systemctl start psacct \
> else \
> sudo systemctl stop psacct \
> fi
```

​多线的 *if* 条件判断语句：

```bash
if <CONDITION>; then
    <STATEMENT>
    ...
elif <CONDITION>; then
    <STATEMENT>
    ...
else 
    <STATEMENT>
    ...
fi
```

​以下代码段演示了使用 *if/then/elif/then/else* 语句来运行 mysql 客户端（若 mariadb 服务处于活动状态），运行 psql 客户端（若 postgresql 服务处于活动状态）或运行 sqlite3 客户端（若 mariadb 和 postgresql 服务均未处于活动状态）。

```bash
[root@localhost ~]\# systemctl is-active mariadb > /dev/null 2>&1
[root@localhost ~]\# MARIADB_ACTIVE=$?
[root@localhost ~]\# sudo systemctl is-active postgresql > /dev/null 2>&1
[root@localhost ~]\# POSTGRESQL_ACTIVE=$?
[root@localhost ~]\# if [ "$MARIADB_ACTIVE" -eq 0 ]; then    \
> mysql    \
> elif [ "$POSTGRESQL_ACTIVE" -eq 0 ]; then \
> psql \
> else \
> sqlite3 \
> fi
```

## 使用正则表达式

​用法可以参考此网站：https://regexr.com/。

​查看配置文件时忽略注释和空白行：

```bash
[root@localhost ~]\# cat /etc/ssh/sshd_config | grep -v "#" | grep -v ^$
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
SyslogFacility AUTHPRIV
PermitRootLogin yes
AuthorizedKeysFile    .ssh/authorized_keys
......
```
