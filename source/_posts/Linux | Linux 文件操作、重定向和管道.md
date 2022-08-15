---
title: Linux 文件操作、重定向和管道
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

## 文件操作相关

### 文件名通配符

| 通配符 | 作用  | 举例  |
| --- | --- | --- |
| \*  | 代表任意多个字符 | 列出以".log"结尾的文件：ll \*.log |
| ?   | 代表任意单个字符 | 列出单个字符构成文件名的文件：ll ? |
| [abc...] | 括起的类（位于方括号间的）中的任何一个字符 | 列出由纯小写字母构成，第二位为i的三位字符构成文件名的文件：ll -d [a-z]i[a-z] |
| [!abc...] | 不是括起的类中的任何一个字符 | ll -d [\^a-b]i[\^x-z] |
| {匹配一,匹配二,匹配三...}或{匹配一..匹配三} | 将上下文与花括号内的内容结合匹配 | ll -d /etc/rc{1,4,5}.d |

​ [] 中的其它常见类名：[:alpha:] 任何字母字符，[:lower:] 任何小写字母，[:upper:] 任何大写字母，[:alnum:] 任何字母字符或数字，[:punct:] 除空格和字母数字以外的任何可打印字符，[:digit:] 从 0 到 9 任何单个数字，[:space:] 任何一个空白字符。可能包含制表符、换行符、回车符、换页符或空格。

### 使用 `pwd` 命令查看当前工作目录的绝对路径

```bash
[root@localhost ~]\# type pwd
pwd is a shell builtin
[root@localhost ~]\# pwd
/root
```

### 使用 `touch` 命令新建文件

​ `touch` 命令的作用是改变文件的时间戳，不过最常用来创建新的空白文件：

```bash
[root@localhost ~]\# touch file.txt && ls | grep file
file.txt
```

### 使用 `file` 命令查看文件类型

​Linux 不需要文件拓展名来区分文件类型，使用 `file` 命令可以扫描文件的文件头来显示文件或目录类型：

```bash
[root@localhost ~]\# file /usr/bin/file
/usr/bin/file: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=2f48178ab3e0f2368f38669dd1d58c49bf0dcf8f, stripped
# file 命令常见选项：
-b, --brief 不会在输出结果前面加上文件名和冒号（简洁模式）
-i, --mime 显示更加详细的 mime 以及字符集信息，效果相当于：--mime-type --mime-encoding
-L, --dereference 进一步显示符号链接到的文件的类型
```

### 使用 `cat` 命令查看文件内容（以字符文件形式打开）

> 🟢 准确的说 `cat` 命令的作用是将文件以字符的形式输出到标准输出设备中。

```bash
[root@localhost ~]\# cat /proc/cpuinfo # 无文件名参数或提供"-"时会读取标准输入设备,支持同时输出多个文件
processor    : 0
vendor_id    : GenuineIntel
cpu family    : 6
model        : 142
model name    : Intel(R) Core(TM) i5-7200U CPU @ 2.50GHz
# 常见选项
-b, --number-noblank 为非空白行显示行号并输出
-n, --number 为所有输出行进行编号
```

### 使用 `grep` 命令搜索文件内容

​`grep` - 将匹配模式的文件内容按行输出。如果没有提供 FILE 参数，递归搜索会检查工作目录，非递归搜索会读取标准输入。

```bash
grep [OPTIONS] PATTERN [FILE...] # 搜索每个文件中的模式（默认使用基本正则表达式语法），用 - 代表读取标准输入文件
grep [OPTIONS] -e PATTERN ... [FILE...]
grep [OPTIONS] -f FILE ... [FILE...]
# 常见选项：
-E, --extended-regexp=PATTERN # 将 PATTERN 解释为拓展正则表达式进行搜索匹配
-F, --fixed-strings=PATTERN # 将 PATTERN 解释为固定字符串列表（而不是正则表达式），由 \ 分隔，其中任何一个字符串都要匹配
-e, --regexp=PATTERN # 可以指定多次，也可以和 -f 混用，搜索所有指定的模式，搜索指定的模式
-f, --file=FILE # 可以指定多次，也可以和 -e 混用，搜索所有指定的模式，从文件中读取模式，一行一个
-i, --ignore-case # 忽略大小写
-v, --invert-match # 输出不匹配模式的所有行（反选）
-w, --word-regexp # 完整匹配，即搜索结果必须是完整的字
-x, --line-regexp # 搜索一整行
-c  # 忽略输出匹配结果，只输出匹配结果数，-cv 输出不匹配的结果 
--color=WHEN # 结果颜色高亮，配色方案可在环境变量 $GREP_COLORS 中配置，值：never,always,auto，默认：grep=grep --color=auto
-L, --files-without-matches # 忽略输出匹配结果，只输出没有匹配结果的文件
-l, --files-with-matches # 忽略输出匹配结果，只输出含有匹配结果的文件
-m, --max-count # 达到指定数量行数的匹配结果后停止搜索
-o, --only-matching # 只输出匹配结果中匹配的部分
```

例子：

```bash
[root@localhost ~]\# grep -e rm -e mv .bashrc
alias rm='rm -i'
alias mv='mv -i'
[root@localhost ~]\# grep -c -e rm -e mv .bashrc
2
[root@localhost ~]\# grep -w alias .bashrc
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
[root@localhost ~]\# grep -w alia .bashrc
[root@localhost ~]\# grep -i ALIas .bashrc
# User specific aliases and functions
alias rm='rm -i'
alias cp='cp -i'
alias mv='mv -i'
```

![grep 命令执行截图](images/linux/grep-命令执行结果.png)

### 使用 `wc` 命令统计文件中行、字和字符的数量

​可以一次指定多个文件，且未指定文件名参数或指定参数 ”-“ 时默认读取标准输入文件。默认情况下该命令输出的结果按照 行数、 字数、字符数、字节数、最大行的长度 的顺序排序，也可以通过选项来指定要输出的内容：

```bash    
[root@localhost ~]\# wc --help | wc # 统计命令 wc --help 执行结果的行数、字数、字符数
     24     158    1201
# 常见选项
-l, --lines 打印行数
-w, --words 打印字数
-m, --chars 打印字符数
-c, --bytes 打印字节数
-L, --max-line-length 打印最长行的长度
```

## 重定向和管道

​因为上下文中都有涉及，先来了解重定向和管道的相关知识：

### 重定向和管道

​Linux 有三个特殊的文件分别是标准输入流文件（stdin)、标准输出流文件（stdout）、标准错误输出流文件（stderr），它们在你打开的程序开始运行时被默认打开。以终端程序为例，每次键入的内容被送到标准输入流作为程序的输入，程序处理后的文字输出通过标准输出流成为我们看到的命令执行结果，错误信息则会被送到标准错误输出流中处理，它们通常都会显示在对应的终端窗口中。在终端中分别可以用 0、1、2 代指它们，例子：

```bash
[root@localhost ~]\# cat file.txt 1>> res.txt && cat res.txt # 文件描述符 1 可以省略
echo helloword
echo helloword
# 解释：读取 file.txt 的内容并直接追加输出到 res.txt 并显示 res.txt 文件里的内容
```

​有时我们会有需求将某个文件作为命令输入或者命令执行结果的输出，这可以使用重定向来实现，输入输出重定向可简单理解为用其它设备代替键盘和显示器。

​注意，如果我们想要将某条命令的执行结果作为第二条命令的参数，需要通过管道来实现而不是重定向。

> 🔴 如果误使用重定向向第二条命令传参的话可能会破环第二条命令的可执行文件的内容！切记：*重定向是将本该在标准输入输出（键盘、屏幕）的内容改用其他文件来进行读写的*。

| 命令符号格式 | 作用  |
| --- | --- |
| 命令 `<` 文件 | 将指定文件的内容作为命令的输入设备 |
| 命令 `<<` 分界符 | 表示从标准输入设备（键盘）中读入，直到遇到分界符才停止（读入的数据不包括分界符），这里的分界符其实是自定义的字符串 |
| 命令 `<` 文件1 `>` 文件2 | 将文件1作为命令的输入设备，该命令的执行结果输出到文件2中 |
| 命令 `>` 文件 | 将命令执行的标准输出结果重定向输出到指定的文件中（清空写入模式） |
| 命令 `2>` 文件 | 将命令执行的错误输出结果重定向输出到指定的文件中（清空写入模式） |
| 命令 `>>` 文件 | 将命令执行的标准输出结果重定向输出到指定的文件中（追加写入模式） |
| 命令 `2>>` 文件 | 将命令执行的错误输出结果重定向输出到指定的文件中（追加写入模式） |
| 命令 `>>` 文件 `2>&1` 或者 命令 `&>>` 文件 | 将命令执行的标准输出或者错误输出写入到指定文件中（追加写入模式） |
| 命令1 ` | ` 命令2 |

### 重定向的例子

```bash
## 例一：
[root@localhost ~]\# cat < /etc/hosts    # 跟 cat /etc/hosts 效果一致
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
# 虽然它们的执行结果相同，但本例是以 /etc/hosts 文件作为输入设备的，注释是以键盘作为输入设备的。

## 例二：
[root@localhost ~]\# cat << EOF    # 从标准输入设备不断读取内容知道读取到 EOF 才停止，并输出
> abcd
> cde
> eof EOF
> EOF
abcd
cde
eof EOF

## 例三：
# 做的事：新建 hosts.bak 文件 -> 从 /etc/hosts 读入内容并输出到 hosts.bak -> 显示 hosts.bak 的内容
[root@localhost ~]\# touch hosts.bak && cat < /etc/hosts > hosts.bak && cat hosts.bak
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
# 整条命令由三条语句构成，且后面的语句执行与否由前面语句的执行成功与否的结果决定

## 例四：
# 准备工作：新建两个空白文件 file1.txt 和 file2.txt，其中向 file1.txt 添加内容 ”123321“
# 查看两个文件的内容
[root@localhost ~]\# more file1.txt file2.txt
::::::::::::::
file1.txt        # -> 文件1的文件名
::::::::::::::
123321            # -> 文件1的内容
::::::::::::::
file2.txt        # -> 文件2的文件名 因为没有内容所以显示内容为空
::::::::::::::
# 读取 file2.txt 的内容并清空写入到 file1.txt 并显示 file1.txt 的内容
[root@localhost ~]\# cat file2.txt > file1.txt && cat file1.txt
[root@localhost ~]\# █    # file1.txt 的内容被清空写入了空内容
# 向文件 file1.txt 和 file2.txt 分别写入 123321
[root@localhost ~]\# echo '123321' > file1.txt && cat file1.txt > file2.txt
# 此时再查看这两个文件的内容
[root@localhost ~]\# more file1.txt file2.txt
::::::::::::::
file1.txt        # -> 文件1的文件名
::::::::::::::
123321            # -> 文件1的内容
::::::::::::::
file2.txt        # -> 文件2的文件名
::::::::::::::
123321            # -> 文件2的内容
# 将 file1.txt 的内容追加到 file2.txt 中并显示 file2.txt 的内容
[root@localhost ~]\# cat file1.txt >> file2.txt && cat file2.txt
123321
123321            # <-- 新追加的内容
# 将不存在的文件的内容清空写入到 file1.txt 并尝试查看 file1.txt 的内容
[root@localhost ~]\# cat file_not_exist.txt > file1.txt; cat file1.txt
cat: file_not_exist.txt: No such file or directory    # 前面语句执行出错,标准输出为空，错误输出显示在了屏幕上，file1.txt 的内容被清空写入了空内容
# 同上例，不过要求将错误信息输出到 file1.txt，标准输出正常输出到显示器
[root@localhost ~]\# cat file_not_exist.txt 2> file1.txt; cat file1.txt
cat: file_not_exist.txt: No such file or directory    # 注意这是 file1.txt 的文件内容

[root@localhost ~]\# cat file_not_exist.txt &>> file1.txt # 再次执行，将标准输出或错误输出都追加到 file1.txt 
[root@localhost ~]\# cat file1.txt
cat: file_not_exist.txt: No such file or directory
cat: file_not_exist.txt: No such file or directory

## 例五：（要使用上例中的两个文件）
# 查看 file1.txt 和不存在的文件的内容并将标准输出和错误输出都重定向追加写入到 file2.txt
[root@localhost ~]\# cat file1.txt file_not_exist.txt >> file2.txt 2>&1
[root@localhost ~]\# cat file2.txt
123321                                                # -> file2.txt 的原有内容
123321                                                # -> file2.txt 的原有内容
cat: file_not_exist.txt: No such file or directory    # -> file1.txt 定向过来的内容
cat: file_not_exist.txt: No such file or directory    # -> file1.txt 定向过来的内容
cat: file_not_exist.txt: No such file or directory     # -> 找不到文件的标准错误输出
# 上例也可以如下实现：
[root@localhost ~]\# cat file1.txt file_not_exist.txt &>> file2.txt
[root@localhost ~]\# cat file2.txt
123321                                                # -> file2.txt 的原有内容
123321                                                # -> file2.txt 的原有内容
cat: file_not_exist.txt: No such file or directory    # -> file2.txt 的原有内容
cat: file_not_exist.txt: No such file or directory    # -> file2.txt 的原有内容
cat: file_not_exist.txt: No such file or directory    # -> file2.txt 的原有内容
cat: file_not_exist.txt: No such file or directory    # -> file1.txt 定向过来的内容
cat: file_not_exist.txt: No such file or directory    # -> file1.txt 定向过来的内容
cat: file_not_exist.txt: No such file or directory    # -> 找不到文件的标准错误输出

## 例六
[root@localhost .config]\# ps aux | grep bash # 执行 ps aux 命令并在其输出中查找 bash 关键字
root        1045  0.0  0.0  26344  2468 ?        S    Apr11   0:13 /bin/bash /usr/sbin/ksmtuned
root       12458  0.0  0.1  27660  5612 pts/0    Ss   08:25   0:01 -bash
root       20373  0.0  0.0  12112  1092 pts/0    S+   21:33   0:00 grep --color=auto bash
```

### 既想在终端上输出命令执行结果又想将其保存到文件中

​ `tee` 命令：从标准输入中获取内容并写入到标准输出和文件中去。

```bash
tee [OPTIONS]... [FILE]... # 复制标准输入获得的内容到文件和标准输出
# 常见选项：
-a, --append # 向指定的文件中追加写入，而不是覆盖
-i, --ignore-interrupts # 忽略中断信号
```

例子：

```bash
[root@localhost ~]\# ll | tee file.txt | grep Videos && cat file.txt
drwxr-xr-x. 2 root root    6 Apr  9 04:49 Videos
total 8
-rw-r--r--. 1 root root    0 Apr 10 21:51 $
-rw-------. 1 root root 1327 Apr  9 04:40 anaconda-ks.cfg
drwxr-xr-x. 2 root root    6 Apr  9 04:49 Desktop
drwxr-xr-x. 2 root root    6 Apr  9 04:49 Documents
drwxr-xr-x. 2 root root    6 Apr  9 04:49 Downloads
-rw-r--r--. 1 root root    0 Apr 15 19:46 file.txt
-rw-r--r--. 1 root root 1680 Apr  9 04:43 initial-setup-ks.cfg
drwxr-xr-x. 2 root root    6 Apr  9 04:49 Music
drwxr-xr-x. 2 root root    6 Apr  9 04:49 Pictures
drwxr-xr-x. 2 root root    6 Apr  9 04:49 Public
drwxr-xr-x. 2 root root    6 Apr  9 04:49 Templates
drwxr-xr-x. 2 root root    6 Apr  9 04:49 Videos
```

## 其他查看文件内容的方法

使用 `cat` 命令查看大篇幅的文件内容时，显示这些文件时所占用的空间可能要超过终端提供的大小。cat 命令不会对文件的内容进行分页。

### 使用 `more` 分页查看文件内容

​more 和 cat 类似，但是 more 支持分页查看。

```bash
[root@localhost ~]\# more /proc/meminfo
MemTotal:        3798432 kB
MemFree:         1387992 kB
...... # 省略内容
KReclaimable:     213284 kB
--More--(0%)█    # space 或 ctrl + f 往下翻页，b 或 ctrl + b 往回翻页，enter 向下滚动一行，q 退出
# 常见选项           h 显示帮助信息
-d, 显示操作提示信息
-f, 以实际文件行数计算行数（非自动换行后的行数）
```

### 使用 `less` 分页查看文件内容

​less 算是 more 的加强版，less 不会读取整个文件，因此其加载时间会很快。

```bash
# 使用 less 读取文件
[root@localhost ~]\# less /proc/cpuinfo
processor       : 0
vendor_id       : GenuineIntel
......    # 省略内容
a cmov pat pse36 clflush mmx fxsr sse sse2 ss ht syscall nx pdpe1gb rdtscp lm constant_tsc arch_perfmon nopl xtopology tsc_reliable nonstop_tsc cp......
/proc/cpuinfoc█
# 使用 less 读取命令输出的内容（需结合管道）
[root@localhost ~]\# ps aux | less
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
...... # 省略内容
root          13  0.0  0.0      0     0 ?        S    11:00   0:00 [watchdog/0]
:█
# 常见选项
-N, 显示行号
-X, 退出 less 后在屏幕上保留内容
+F, 表现与 tail -f 命令几乎相同
```

可使用的键盘操作：

| 按键  | 作用  | 备注  |
| --- | --- | --- |
| `space` or `f` | 往下翻 N 行 | N 默认为一个窗口的行数大小，可以在冒号后面输入数字指定 N 的值再按快捷键 |
| `b` 或 `ctrl` + `b` 或 `esc` + `v` | 往上翻 N 行 | N 默认为一个窗口的行数大小，可以在冒号后面输入数字指定 N 的值再按快捷键 |
| `d` 或 `ctrl` + `d` | 往下翻 N 行 | N 默认为半个屏幕的行数大小，可以在冒号后面输入数字指定 N 的值再按快捷键 |
| `u` 或 `ctrl` + `u` | 往上翻 N 行 | N 默认为半个屏幕的行数大小，可以在冒号后面输入数字指定 N 的值再按快捷键 |
| ``↓`` 或 `e` 或 `j` 或 `enter` | 往下翻一行 |     |
| `↑` 或 `y` 或 `k` | 往上翻一行 |     |
| `g` 或 `p` | 分别对应转到第一行和最后一行 | `g` 前面指定 N 转到第 N 行，`p` 前面指定 N 转到第 N% 的内容 |
| `/` 或 `?` 跟模式 | 分别对应向前或向后搜索模式 | `n` 重复上一次搜索，`N` 反向重复上一次搜索 |
| `h` 或 `q` | `h` 显示帮助信息 | `q` 退出 less。 |

### `head` 或 `tail` 显示部分文件内容

​head 和 tail 分别显示文件的开头和结尾部分，默认情况下，这两个命令显示文件的 10 行，但它们都有一个 *-n* 选项，允许指定不同的行数。未指定文件名参数或指定参数 ”-“ 时默认读取标准输入文件。

```bash
[root@localhost ~]\# head -n 3 /etc/passwd
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
[root@localhost ~]\# tail -n 2 -fs 2 /var/log/messages
Apr 11 22:21:47 localhost systemd[1]: Started Network Manager Script Dispatcher Service.
Apr 11 22:21:57 localhost systemd[1]: NetworkManager-dispatcher.service: Succeeded.
█    # -f 监视文件，文件内容发生变化时也跟着输出（追加）， -s 指定扫描文件变化间隔的秒数（与 -f 搭配，默认 1s）
# 按 ctrl + c 退出监控模式
```
