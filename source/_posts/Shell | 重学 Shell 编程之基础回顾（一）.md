---
title: 重学 Shell 编程之基础回顾（一）
lang: zh-CN
tags: 
  - Shell
  - Bash
  - Linux
  - 脚本编程
categories: 
  - [学习, Shell]
  - [日常记录]
---

## Shebang

Shebang 是指出现在 Shell 脚本文件第一行中的前两个字符 `#!`，用以指定命令解释器，例如：

* `#!/bin/sh` 会使用 `/bin/sh`，即 `bash`(`sh` 是 `bash` 的一个软连接)解释器执行脚本内容；
* `#!/usr/bin/python` 会使用 python 解释器去执行脚本内容；
* `#!/usr/bin/env NAME`，是一种在不同平台上都能找到 NAME 解释器，并用 NAME 解释器执行脚本内容的写法。

> 🟢 一些细节：
>
> 1. 如果脚本没有指定 **shebang**，脚本执行的时候，默认用当前 shell 去解释脚本，即 **$SHELL** 环境变量；
> 2. 如果 **shebang** 指定了可执行的解释器，如 **/bin/bash**、**/usr/bin/python**，那么脚本在执行时，文件名会作为参数传递给解释器；
> 3. 如果 **shebang** 指定的解释器没有可执行权限，则会报错："bad interpreter: Permission denied"；
> 4. 如果 **shebang** 指定的解释器不是一个可执行文件，那么则会忽略此指定的解释器，继而转交给当前的 shell 去执行这个脚本；
> 5. 如果 **shebang** 指定的解释器不存在，那么会报错 "bad interpreter: No such file or directory"；
> 6. **shebang** 指定解释器需写其绝对路径（如：**#!/bin/bash**），找寻解释器的过程不会在 **$PATH** 环境变量中找；
> 7. 如果使用 `bash script.sh` 这样的命令来执行脚本，脚本中 **shebang** 所指定的解释器会被忽略而使用命令显式指定的 **bash**。

## 执行脚本

执行一个脚本一般有以下几种方法，以执行 `script.sh` 脚本为例：

* `bash script.sh` 或 `sh script.sh`，被执行的脚本文件可以没有可执行权限，或者脚本没有指定 `shebang` 时可使用；
* 使用 绝对/相对 路径执行脚本，被执行的脚本文件需要有可执行权限；
* `source script.sh` 或者 `. script.sh`，在当前 Shell 进程中执行脚本（不产生子 Shell 进程）；
* 少见用法：`sh < script.sh`。

一些常见的其他 shell 解释器：

* `/bin/sh`
* `/bin/bash`
* `/sbin/nologin`
* `/usr/bin/sh`
* `/usr/bin/bash`
* `/usr/sbin/nologin`
* `/bin/tcsh`
* `/bin/csh`
* `/bin/dash`

当前系统支持的解释器类型可以执行 `cat /etc/shells` 命令查看。不过最受欢迎的解释器还是 `bash` 解释器。`bash` 解释器支持以下特性：

* 文件路径 tab 键补全
* 命令补全
* 快捷键 `ctrl` + `a`、`e`、`u`、`k`、`l`
* 通配符
* 历史命令
* 命令别名
* 命令行展开

## Shell 变量

Shell 语言是弱类型语言，如其他编程语言，其也有变量的概念。

### 定义变量

```bash
\$ NAME="VALUE" # 默认所有变量类型都是字符串类型，不能指定类型
\$ echo ${NAME}
\$ VALUE
```

关于单引号、双引号、反引号用法的一个案例：

```bash
\$ date="2020-03-29 20:24:56 CST" 
\$ VAR='${date}';echo ${VAR}
\$ $date                                        # 单引号括起来的字符串不解析变量及命令结果
\$ VAR="${date}";echo ${VAR}
\$ 2020-03-29 20:24:56 CST                      # 双引号括起来的字符串会解析变量和命令结果
\$ VAR=`date`;echo ${VAR}
\$ 2022年 03月 30日 星期三 08:58:33 CST           # 反引号代表执行命令，并取得命令的结果
\$ VAR="`date`";echo ${VAR}
\$ 2022年 03月 30日 星期三 09:01:14 CST           # 双引号也会解析反引号执行的结果
```

### 变量替换及引用

```bash
\$ NAME="ANOTHER_VALUE"
\$ echo ${NAME}
\$ ANOTHER_VALUE
\$ echo $NAME # 省略花括号
\$ ANOTHER_VALUE
```

### 变量名命名规范

* 见名知意，避免使用保留关键字
* 变量名组成可以是字母、数字以及下划线
* 不能以数字开头
* 不能使用标点符号
* 变量名严格区分大小写

### 变量的作用域

* 局部变量：只能在函数内部使用的变量
* 全局变量：只能在当前 Shell 进程中使用的变量
* 环境变量：可以在子 Shell 进程中使用的变量

> 🔵 注意点：
>
> 1. 不同于其他编程语言，Shell 函数内部定义的变量的作用域是全局的，即执行这个脚本的进程内可见（不等同于脚本内部可见，因为使用 **source** 可以在一个 Shell 进程内执行多个脚本），要实现仅在函数内部可见需使用 **local** 关键字，如：**local var="something"**
> 2. 全局变量只在当前 Shell 进程可见，对其他进程、子进程都不可见，当使用 **export** 命令将变量进行导出后，父 Shell 进程的被导出的变量将会对子进程可见，该变量即被称作环境变量
> 3. 父 Shell 进程退出后，此环境变量会消失，如要定义一些常用的环境变量并在启动每个 Shell 进程时自动定义它们，此时就需将它们写在特定的配置文件里

### 一些特殊变量

| 变量      | 描述                                                         |
| --------- | ------------------------------------------------------------ |
| \$0       | 当前脚本的文件名                                             |
| \$n(n>=1) | 传递给脚本或函数的参数。（n 代表是第几个传递的参数，数字）   |
| \$\#      | 传递给脚本或函数的参数个数。                                 |
| \$\*      | 传递给脚本或函数的所有参数，被引号括起来时所有参数视作一个整体。 |
| \$@       | 传递给脚本或函数的所有参数，被引号括起来时每个参数视作一个整体，可遍历每个参数。 |
| \$?       | 上个命令的退出状态，或函数的返回值。                         |
| \$\$      | 当前 Shell 进程 ID。对于 Shell 脚本，就是脚本所在的进程 ID。 |
| \$\!      | 上一次后台进程的 PID。                                       |
| \$\_      | 获取上次执行命令的最后一个参数。                             |

```bash
[skinyi@fedora ~]$ cat test.sh
#!/bin/sh
echo '使用"$*"打印参数：'
for var in "$*"
do
        echo "$var"
done
echo '使用"$@"打印参数：'
for var in "$@"
do
        echo "$var"
done

[skinyi@fedora ~]$ bash test.sh a b c d
使用"$*"打印参数：
a b c d
使用"$@"打印参数：
a
b
c
d
```

> 🟢 一般来说，程序、命令的返回值为 0 时代表命令成功执行完成，返回其他值即代表发生了错误，可以根据返回值去查询相应的文档进行排错调试。对于 Shell 来说，函数的返回值跟 C、JAVA 之类的函数返回值的意义和作用是不同的。

### 查看环境变量

* set
* declare
* env
* export

案例：判断传入参数个数。

```bash
[skinyi@fedora ~]$ cat test.sh
#!/bin/sh
[ "$#" -ne "2" ] && {
        echo "Expect 2 args!"
        exit 112
}

[skinyi@fedora ~]$ bash test.sh 1
Expect 2 args!
```

## Bash 内建命令

可以使用 `compgen -b` 查看所有 shell 内建命令。shell 内建命令附加在 bash 命令中，因此它们是随 bash 进程常驻内存的，内建命令执行一般不需要创建子进程。

| 命令      | 作用                                              |
| --------- | ------------------------------------------------- |
| echo      | 将指定字符串输出到 STDOUT                         |
| eval      | 将指定的参数拼接成一个命令，然后执行该命令        |
| exec      | 用指定命令替换 shell 进程                         |
| export    | 设置子 shell 进程可用的变量                       |
| read      | 从 STDIN 读取一行数据并将其赋给一个变量           |
| shift     | 将位置参数依次向下降一个位置                      |
| alias     | 为指定命令定义一个别名                            |
| getopts   | 分析指定的位置参数                                |
| let       | 计算一个数学表达式中的每个参数                    |
| local     | 在函数中创建一个局部变量                          |
| readonly  | 从 STDIN 读取一行数据并将其赋给一个不可修改的变量 |
| readarray | 从 STDIN 读取数据行并将其放入索引数组             |
| test      | 基于指定条件返回退出状态码 0 或 1                 |
| trap      | 如果收到了指定的系统信号，执行指定的命令          |
| wait      | 等待指定的进程完成，并返回退出状态码              |

### echo

```bash
# 常用选项
-n 不换行输出
-e 解析字符串中的格式化字符，等同于 `printf` 命令的效果
# 格式化字符
\n 换行
\r 回车
\t 制表符
\b 退格
```

### eval

```bash
# 例子
[skinyi@fedora ~]$ set 11 22 33 44 # 通过 set 命令设置 $1, $2, $3, $4
[skinyi@fedora ~]$ echo $4         # 输出 $4 的值，即 44
[skinyi@fedora ~]$ echo '$'$#      # 此时会输出 $4 而非 $4 的值
[skinyi@fedora ~]$ eval echo '$'$# # 此时会输出 $4 的值 44
```

原理：eval 会把参数进行扫描拼接再执行，即上述例子最后一条命令会先扫描得到 `echo $4`，然后再去执行。

### exec

exec 会在当前进程中直接执行后面的命令而不会创建子进程，因此当执行的命令退出后该进程也会被销毁，即效果是原本的 bash 进程“不见了”。

```bash
[skinyi@fedora ~]$ sudo su -
[root@fedora ~]\# exec whoami
root
[skinyi@fedora ~]$ 
```

## Shell 子串

| 字串                    | 描述                                                     |
| ----------------------- | -------------------------------------------------------- |
| \$\{var\}                 | 返回变量 var 的值                                        |
| \$\{@var\}                | 返回变量 var 的值的字符串长度，**注意@替换为#**              |
| \$\{var\:start\}           | 返回变量 var 的 start 个字符之后的所有字符               |
| \$\{var\:start\:length\}    | 返回变量 var 的 start 个字符之后长度为 length 的子字符串 |
| \$\{var\#word\}            | 返回变量 var ”掐头“（最短匹配）后的子字符串              |
| \$\{var\#\#word\}           | 返回变量 var ”掐头“（最长匹配）后的子字符串              |
| \$\{var%word\}            | 返回变量 var ”去尾“（最短匹配）后的子字符串              |
| \$\{var%%word\}           | 返回变量 var ”去尾“（最长匹配）后的子字符串              |
| \$\{var/pattern/string\}  | 返回变量 var 经单次匹配 pattern 并使用 string 替换后的值 |
| \$\{var//pattern/string\} | 返回变量 var 经多次匹配 pattern 并使用 string 替换后的值 |
| \$\{var:-word\}           | 未定义变量 var 或值为空时返回 word，否则直接返回 var 的值  |
| \$\{var:=word\}           | 未定义变量 var 或值为空时返回 word，同时将 word 赋值给 var |
| \$\{var:+word\}           | 当变量 var 已赋值时才返回 word 进行替换，否则不替换        |
| \$\{var:?MASSAGE}         | 当变量 var 已赋值时才返回 var 的值，否则输出 MASSAGE 到 STDERR |

子串处理示例：

```bash
[skinyi@fedora ~]$ var="abcdefghi"
[skinyi@fedora ~]$ echo ${var}
abcdefghi
[skinyi@fedora ~]$ echo ${#var}
9
[skinyi@fedora ~]$ echo ${var:2}
cdefghi
[skinyi@fedora ~]$ echo ${var:0-5}     # 从右数第五个开始输出
efghi
[skinyi@fedora ~]$ echo ${var:2:4}
cdef
[skinyi@fedora ~]$ echo ${var:0-4:2}   # 从右数第四个开始输出两个字符
fg
[skinyi@fedora ~]$ echo ${var#a*d}
efghi
[skinyi@fedora ~]$ echo ${var%e*i}
abcd
[skinyi@fedora ~]$ echo ${var/cde/edc}
abedcfghi
```

### 案例——批量文件名修改

```bash
# 准备工作：产生实验数据
[skinyi@fedora case]$ touch file_{1..5}_draft.{jpg,png} && ls
file_1_draft.jpg  file_2_draft.jpg  file_3_draft.jpg  file_4_draft.jpg  file_5_draft.jpg
file_1_draft.png  file_2_draft.png  file_3_draft.png  file_4_draft.png  file_5_draft.png
[skinyi@fedora case]$ mv file_2_draft.jpg file_2_finished.jpg
[skinyi@fedora case]$ mv file_2_draft.png file_2_delete.png
[skinyi@fedora case]$ ls
file_1_draft.jpg  file_2_delete.png    file_3_draft.jpg  file_4_draft.jpg  file_5_draft.jpg
file_1_draft.png  file_2_finished.jpg  file_3_draft.png  file_4_draft.png  file_5_draft.png
[skinyi@fedora case]$
# 任务一：将所有结尾为 draft 的 jpg 文件重新命名，去掉结尾的 _draft 后缀
[skinyi@fedora case]$ for filename in `ls *draft.jpg`; do mv $filename ${filename/_draft/}; \
done; ls
file_1_draft.png  file_2_delete.png    file_3_draft.png  file_4_draft.png  file_5_draft.png
file_1.jpg        file_2_finished.jpg  file_3.jpg        file_4.jpg        file_5.jpg
[skinyi@fedora case]$
# 任务二：将所有开头为 file 且结尾为 draft 的 png 文件重新命名，修改为 pic_{n}_undo.png 的形式
[skinyi@fedora case]$ for filename in `ls *draft.png`; do tmp=${filename/draft/undo}; \
mv $filename ${tmp/file/pic}; done; ls
file_1.jpg         file_2_finished.jpg  file_4.jpg  pic_1_undo.png  pic_4_undo.png
file_2_delete.png  file_3.jpg           file_5.jpg  pic_3_undo.png  pic_5_undo.png
[skinyi@fedora case]$
```

Shell 扩展变量示例：

```bash
# 未定义 undefined 变量，定义 defined 变量的值为 defined
[skinyi@fedora case]$ echo $undefined

[skinyi@fedora case]$ defined="defined"
[skinyi@fedora case]$
# var2=${var1:-word} 用法示例
[skinyi@fedora case]$ res=${undefined:-"not defined"};printf "$undefined\n$res\n"

not defined
[skinyi@fedora case]$ res=${defined:-"not defined"};printf "$defined\n$res\n"
defined
defined
[skinyi@fedora case]$
# var2=${var1:=word} 用法示例
[skinyi@fedora case]$ res=${undefined:="will defined"};printf "$undefined\n$res\n"
will defined
will defined
[skinyi@fedora case]$ res=${defined:="will redefined?"};printf "$defined\n$res\n"
defined
defined
[skinyi@fedora case]$ unset undefined           # 销毁 undefined 变量
[skinyi@fedora case]$ echo $undefined

[skinyi@fedora case]$
# var2=${var1:+word} 用法示例
[skinyi@fedora case]$ res=${undefined:+"something"};printf "$undefined\n$res\n"


[skinyi@fedora case]$ res=${defined:+"something"};printf "$defined\n$res\n"
defined
something
# var2=${var1:?MASSAGE} 用法示例
[skinyi@fedora case]$ res=${undefined:?"undefined is not defined!"};printf "$undefined\n$res\n"
bash: undefined: undefined is not defined!      # 程序立即返回，没有执行后面的语句
[skinyi@fedora case]$ res=${undefined:?}        # 也可不指定报错信息，会有默认的 stderr 信息
bash: undefined：参数为空或未设置
[skinyi@fedora case]$ res=${defined:?"undefined is not defined!"};printf "$defined\n$res\n"
defined
defined
[skinyi@fedora case]$
```

## 命令列表、进程列表、进程分组

* 命令列表，示例：*pwd; ls; ...*   直接执行，不能放到后台进程
* 进程列表，示例：*(pwd; ls; ...)* 在子 shell 中执行然后返回父 shell
* 进程分组，示例：*{pwd; ls; ...}* 不会启动子 shell，与命令列表一样，只是给命令进行了分组

> 🟢 可以通过环境变量 BASH_SUBSHELL 来判断自己是否是子 Shell，若该变量的值为 0，则代表自己不是 subshell，若为其他值则是。
> 🟢 创建子 shell 时新建子进程但子进程由 bash 维护，只能通过 $BASHPID 获取PID，与父进程共用同一个 POSIX 语义下的 PID 与 PPID。本质上实现了多进程。

```bash
[skinyi@fedora ~]$ pwd; ls; echo $BASH_SUBSHELL
/home/skinyi
case  gh-pages  package-lock.json  test.sh
0
[skinyi@fedora ~]$ (pwd; ls; echo $BASH_SUBSHELL)
/home/skinyi
case  gh-pages  package-lock.json  test.sh
1
[skinyi@fedora ~]$ echo $BASH_SUBSHELL;(echo $BASH_SUBSHELL;(echo $BASH_SUBSHELL;(echo $BASH_SUBSHELL;(echo $BASH_SUBSHELL;(echo $BASH_SUBSHELL;)))))
0
1
2
3
4
5
[skinyi@fedora ~]$
```
