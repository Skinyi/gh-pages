---
title: 重学 Shell 编程之 Linux 三剑客
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

## 正则表达式 RegExp

一个在线图形化解释正则表达式的[工具网站](https://jex.im/regulex/)。支持正则表达式的 linux 命令主要有：`grep`、`sed`、`awk`、`find`、`expr`等。正则表达式一般以一系列格式控制字符来匹配具有特定规律和意义的字符串，如电话号码、电子邮件地址等。

正则语法速查表：

| 符号        | 支持的语法      | 描述                                                         |
| ----------- | --------------- | ------------------------------------------------------------ |
| *归类*      |                 | **开始、结束匹配控制**                                       |
| ^           | Basic RegExp    | 表示以它后面的表达式开头才匹配                               |
| $           | Basic RegExp    | 表示以它前面的表达式结尾才匹配                               |
| *归类*      |                 | **一些特殊字符的意义**                                       |
| .           | Basic RegExp    | 表示匹配任意一个字符（不匹配 \n、\r）                        |
| ?           | Extended RegExp | 表示起一个表达式匹配零次或一次                               |
| *           | Basic RegExp    | 表示前一个表达式匹配零次或多次                               |
| +           | Extended RegExp | 表示前一个表达式匹配一次或多次                               |
| \|          | Extended RegExp | 表示在其分隔的两个表达式之间进行选择                         |
| \           |                 | 表示将下个字符进行转义                                       |
| ()          | Extended RegExp | 标记一个子表达式的开始和结束                                 |
| []          | Basic RegExp    | 标记一个中括号表达式的开始和结束                             |
| {}          | Extended RegExp | 标记限定符表达式的开始和结束                                 |
| *归类*      | Basic RegExp    | **除 * + ? 的限定符，n 取非负整数**                          |
| {n}         | Extended RegExp | 表示精确匹配 n 次                                            |
| {n,}        | Extended RegExp | 表示最少匹配 n 次                                            |
| {n,m}       | Extended RegExp | n\<m，表示匹配 n 到 m 次                                     |
| *归类*      |                 | **[] 限定的普通字符**                                        |
| [anychars]  | Basic RegExp    | 表示匹配其中列举的任一字符                                   |
| [^anychars] | Basic RegExp    | 表示匹配除了其中列举的任意字符                               |
| [A-Z]       | Basic RegExp    | 表示匹配所有大写字母中的一个                                 |
| *归类*      |                 | **常见转义字符**                                             |
| \s\S        | Basic RegExp    | \s 匹配所有空白字符、\S 匹配所有非空白字符                   |
| \w          | Basic RegExp    | 相当于 [A-Za-z0-9]，匹配大写字母、小写字母或 0 到 9 中的一个 |
| \d          | Basic RegExp    | 匹配 0 到 9 中的一个数                                       |
| \t          | Basic RegExp    | 匹配一个制表符                                               |
| \v          | Basic RegExp    | 匹配一个垂直制表符                                           |
| \r          | Basic RegExp    | 匹配一个回车符                                               |
| \f          | Basic RegExp    | 匹配一个换页符                                               |

> 1. \* 和 + 限定符都是贪婪的，即会尽可能多的匹配字符（最长子串），在它们后面加上 ？限定符并指定使用 Perl 正则模式实现非贪婪或最小字串匹配。
> 2. 特殊符号要匹配本身，需进行转义。

## 使用 grep + 正则查找文件内容

```bash
grep, egrep, fgrep - 打印匹配给定模式的行
# 用法格式
grep [options] PATTERN [FILE...]
grep [options] [-e PATTERN | -f FILE] [FILE...]
# 常见选项
-A NUM, --after-context=NUM   # 打印出匹配的行之后的下文 NUM 行 
-B NUM, --before-context=NUM  # 打印出匹配的行之前的上文 NUM 行
-C NUM, --context=NUM         # 打印除匹配行之前和之后的上下文 NUM 行
-c NUM, --count # 打印出匹配的数量而不是匹配的内容，配合 -v 为不匹配的数量
-E, --extended-regexp # 使用拓展正则进行解释匹配
-P, --perl-regexp # 使用 perl 正则进行解释匹配
-o, --only-matching # 只显示匹配中的行中与模式相匹配的部分
-v, --invert-match # 只选择不匹配的行（反选）
-w, --word-regexp 
# 只选择含有能组成完整的词的匹配的行。判断方法是匹配的子字符串必须是一行的开始，
# 或者是在一个不可能是词的组成的字符之后。与此相似，它必须是一行的结束，或者是
# 在一个不可能是词的组成的字符之前。词的组成字符是字母，数字，还有下划线。
```

> 🟢grep 默认支持 Basic RegExp 语法，可以通过传递选项 `-E` 以支持扩展正则表达式语法（相当于 `egrep` 命令）或 `-P` 以支持 Perl 语言的正则表达式语法。

```bash
# 测试文件内容
[skinyi@fedora ~]$ cat -n lyrics.txt 
 1  Fly me to the moon
 2
 3  Fly me to the mo0n
 4  And let me play @mong the stars
 5  (I want you)Let me see what Spring is like 0n Jupiter and Mars
 6  In other words,hold my hand
 7  _In other words,darling,kiss me
 8  Fill m9 heart with song
 9  And 1et me sing forever more
10  (Because)You are all I long for All I worship and adOre
11  in other words,please be true
12  in other words,I love you
13
```

```bash
# 过滤空行 -v 反选 -n 输出行号
[skinyi@fedora ~]$ grep -nv '^$' lyrics.txt 
1:Fly me to the moon
3:Fly me to the mo0n
4:And let me play @mong the stars
5:(I want you)Let me see what Spring is like 0n Jupiter and Mars
6:In other words,hold my hand
7:_In other words,darling,kiss me
8:Fill m9 heart with song
9:And 1et me sing forever more
10:(Because)You are all I long for All I worship and adOre
11:in other words,please be true
12:in other words,I love you
```

```bash
# 去掉以某个字符开头的行（常用来去注释）
[skinyi@fedora ~]$ grep -nv '^(' lyrics.txt 
1:Fly me to the moon
2:
3:Fly me to the mo0n
4:And let me play @mong the stars
6:In other words,hold my hand
7:_In other words,darling,kiss me
8:Fill m9 heart with song
9:And 1et me sing forever more
11:in other words,please be true
12:in other words,I love you
13:
```

```bash
# 匹配以 i 开头，e 结尾的最短子串，-P 指定使用 perl 语言的正则语法，支持非贪婪写法
[skinyi@fedora ~]$ egrep -no '^i.+?e' lyrics.txt     # 不能达到目的 
11:in other words,please be true
12:in other words,I love
[skinyi@fedora ~]$ grep -no -P '^i.+?e' lyrics.txt   # 可达到目的 
11:in othe
12:in othe
```

## 使用 `sed` 命令流式的处理文本文件

`sed -r` 支持扩展正则。Sed是一个流式编辑器。流式编辑器是用来在输入流（一个文件或者管道输入）中完成基本文本转换的工具。

`sed` 命令会根据脚本命令来处理文本文件中的数据，这些命令要么从命令行中输入，要么存储在一个文本文件中，此命令的执行过程如下：

1. 每次读取一行内容；
2. 根据提供的规则命令匹配并修改数据。*注意，sed 默认不会直接修改源文件数据，而是会将数据复制到缓冲区中，修改也仅限于缓冲区中的数据；*
3. 将执行结果输出；
4. 当一行数据匹配完成后，它会继续读取下一行数据，并重复 1-3 的过程，直到将文件中所有行的数据处理完毕。

```bash
sed - 文本筛选和格式转换的流式编辑器
# 用法格式
sed [选项]... {脚本（若没有其他脚本）} [输入文件]...
# 常见选项
-e 脚本, --expression=脚本      # 添加 脚本 到程序的处理工作流列表
-f 脚本文件, --file=脚本文件      # 添加 脚本文件 到程序的处理工作流列表
-i [扩展名], --in-place[=扩展名]  # 直接修改文件（如果指定拓展名则备份文件）
-E, -r, --regexp-extended       # 在脚本模式中使用拓展正则
-n, --quiet, --silent          # 取消自动打印模式空间
```

```bash
# 测试文本
[skinyi@fedora ~]$ cat -n lyrics.txt 
     1  My whole world changed from the moment I met you
     2  And it would never be the same
     3  Felt like I knew that I always loved you
     4  From the moment I heard your name
     5
     6  Everything was perfect, I knew this love was worth it
     7  Our own miracle in the making
     8  And till this world stops turning
     9  I’ll still be here waiting and waiting to make that vow that I’ll...
    10
    11  I’ll be by your side
    12  Till the day I die
    13  I’ll be waiting till I hear you say I do
    14  Something old, something new
    15  Something borrowed, something blue
    16  I’ll be waiting till I hear you say I do
    17
```

### 打印内容

`sed` 命令实现文件内容打印采用的基本格式为：`[address][/{front-}pattern/,/{back-pattern/}]p`。其中 address 表示指定要操作的具体行，2 条斜线内实现指定模式内容的打印，3 条斜线分隔且包围的可指定一定范围的内容的查找（若后模式无法匹配则会一直打印到文件末尾）。

```bash
# 查找 met 和 name 之间的内容
[skinyi@fedora ~]$ sed -nr '/\smet\s/,/\sname\s?/p' lyrics.txt 
My whole world changed from the moment I met you
And it would never be the same
Felt like I knew that I always loved you
From the moment I heard your name
```

```bash
# 不显示空行和以 I 开头的行，常用来进行配置文件的过滤（I 替为 #）
[skinyi@fedora ~]$ sed -nr '/^$|I/!p' lyrics.txt 
And it would never be the same
Our own miracle in the making
And till this world stops turning
Something old, something new
Something borrowed, something blue
```

> 🔴注意：! 代表执行反操作，比如 !p 代表匹配的内容不打印（不匹配的打印），!d 代表匹配的内容不删除（不匹配的删除）。

### 查找并替换

`sed` 命令实现文件内容查找并替换采用的基本格式为：`[address]s/pattern/replacement/flags`。其中：

* address 表示指定要操作的具体行；
* pattern 指的是需要替换的内容；
* replacement 指的是要替换的新内容；
* flags 用来实现一些特定功能。

| flags 标记 | 功能                                                                                                                                   |
| ---------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| n          | n 为 1~512 之间的数字，表示指定要替换的字符串出现第几次时才进行替换，例如，一行中有 3 个 A，但用户只想替换第二个 A，这是就用到这个标记 |
| g          | 对数据中所有匹配到的内容进行替换，如果没有 g，则只会在第一次匹配成功时做替换操作。例如，一行数据中有 3 个 A，则只会替换第一个 A        |
| p          | 会打印与替换命令中指定的模式匹配的行。此标记通常与 -n 选项一起使用                                                                     |
| w file     | 将缓冲区中的内容写到指定的 file 文件中                                                                                                 |
| &          | 用正则表达式匹配的内容进行替换                                                                                                         |
| \n         | 匹配第 n 个子串，该子串之前在 pattern 中用 \(\) 指定                                                                                   |
| \\         | 转义（转义替换部分包含：&、\\ 等）                                                                                                     |

```bash
# 将所有出现的 do 替换为 will（不修改源文件）并比较
[skinyi@fedora ~]$ sed 's/do/will/g' lyrics.txt | colordiff lyrics.txt -
13c13
< I’ll be waiting till I hear you say I do
---
> I’ll be waiting till I hear you say I will
16c16
< I’ll be waiting till I hear you say I do
---
> I’ll be waiting till I hear you say I will
```

```bash
# 将第 4 行的 heard 改为 knew 并仅输出修改的行（不修改源文件）
[skinyi@fedora ~]$ sed -n '4s/heard/knew/p' lyrics.txt
From the moment I knew your name
```

```bash
# 将第 13 行的第三个 I 改为 we 并仅输出修改的行（不修改源文件）
[skinyi@fedora ~]$ sed -n '13s/I/we/3p' lyrics.txt
I’ll be waiting till I hear you say we do
```

> 🔵注意：匹配模式字符串包含 / 时需要进行转义。

#### 案例：仅打印网卡 ens160 的 ip

```bash
[skinyi@fedora gh-pages]$ ip addr show ens160 | sed -nr '4s#(^.*t )(.*)(/.*$)#\2#gp' # \2 表示第二组
192.168.30.128
```

### 删除文件中的特定行（单行或连续行）

`sed` 命令实现删除文件中的特定行的基本格式为：`[address]d`，其中 address 表示指定要操作的具体行，当不指定地址时代表删除所有行。

以下例子还是以上述 lyrics.txt 文件为范例。

```bash
# 删除所有行（仅在缓冲区）
[skinyi@fedora ~]$ sed 'd' lyrics.txt
[skinyi@fedora ~]$
```

```bash
# 删除第 12 行（仅在缓冲区）
[skinyi@fedora ~]$ sed '12d' lyrics.txt | cdiff lyrics.txt - # alias cdiff='colordiff'，下同
12d11
< Till the day I die
```

```bash
# 删除第 11 到 13 行（仅在缓冲区）
[skinyi@fedora ~]$ sed '11,13d' lyrics.txt | cdiff lyrics.txt -
11,13d10
< I’ll be by your side
< Till the day I die
< I’ll be waiting till I hear you say I do
```

```bash
# 删除第 10 行到最后一行（仅在缓冲区）
[skinyi@fedora ~]$ sed '10,$d' lyrics.txt | cdiff lyrics.txt - # $代表最后一行的行号
10,17d9
< 
< I’ll be by your side
< Till the day I die
< I’ll be waiting till I hear you say I do
< Something old, something new
< Something borrowed, something blue
< I’ll be waiting till I hear you say I do
< 
```

### 增加内容（以行为单位）

`sed` 命令实现文件中增加特定内容行的基本格式为：`sed [address]{a|n}\new_content`，其中 a 代表向指定位置后面插入内容、i 代表向指定位置前面插入内容、address 表示指定要操作的具体行，不指定则在每行前面或后面添加内容。

```bash
# 给每行后面添加 ----------------------------------------------------（仅在缓冲区），并查看前 10 行
[skinyi@fedora ~]$ sed 'a\-------------------------------------------------' lyrics.txt | head -10
My whole world changed from the moment I met you
-------------------------------------------------
And it would never be the same
-------------------------------------------------
Felt like I knew that I always loved you
-------------------------------------------------
From the moment I heard your name
-------------------------------------------------

-------------------------------------------------
```

```bash
# 给第 1 行前增加 911 - I Do（仅在缓冲区），并查看前 5 行
[skinyi@fedora ~]$ sed '1i\911 - I Do' lyrics.txt | head -5
911 - I Do
My whole world changed from the moment I met you
And it would never be the same
Felt like I knew that I always loved you
From the moment I heard your name
```

```bash
# 添加多行可以使用 \n 转义字符，效果如下：
[skinyi@fedora ~]$ sed '1i\I Do\n911' lyrics.txt | head -5
I Do
911
My whole world changed from the moment I met you
And it would never be the same
Felt like I knew that I always loved you
```

### 替换整行

`sed` 命令实现文件中替换某行的内容的基本格式为：`sed [address]c\new_content`，其中 address 表示指定要操作的具体行，不指定则会将每行的内容都替换成新行。

```bash
# 更改首行内容为 Your partial heart got broken from the moment you left me（仅在缓冲区）并查看前三行
[skinyi@fedora ~]$ sed '1c\Your partial heart got broken from the moment you left me' lyrics.txt | head -3
Your partial heart got broken from the moment you left me
And it would never be the same
Felt like I knew that I always loved you
```

## 使用 `awk` 实现文本扫描处理

AWK 是一门实现文本扫描与处理的编程语言，常用来实现文本过滤、统计、计算等。awk 将文本文件按照指定（或默认）的分隔符抽象成一个不太严格的表格（每列不一定等长），继而实现对文本的精确处理，awk 抽象和表格的对应关系如下：

| 表格概念  | awk 中的概念 | awk 默认分隔符 | awk 内置变量及作用 |
| --------- | ------------ | -------------- | ------------------ |
| 行 Row    | 记录 Record  | 换行 \n        | NR 指定记录号      |
| 列 Column | 字段 Feild   | 空白字符 \s    | NF 获取列数        |

```bash
# 用法格式
awk - pattern scanning and processing language
awk [ options ] -f program-file [ -- ] file ...
awk [ options ] [ -- ] program-text file ...
## awk 程序中的 BEGIN{}、END{} 分别控制读取文件之前和读取文件结束的过程。
# 常见选项
-v var=new_var              # 修改 awk 内置变量的取值，如：-v OFS=:（打印字段时以冒号分隔）
-F fs, --field-separator fs # 指定 fs 作为每列的分隔符，相当于 -v FS=fs。
```
