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
| \s\S        | Basic RegExp    | 表示匹配所有字符。\s 包括空白符（含换行）、\S 不包括空白符   |
| \w          | Basic RegExp    | 相当于 [A-Za-z0-9]，匹配大写字母、小写字母或 0 到 9 中的一个 |
| \d          | Basic RegExp    | 匹配 0 到 9 中的一个数                                       |
| \t          | Basic RegExp    | 匹配一个制表符                                               |
| \v          | Basic RegExp    | 匹配一个垂直制表符                                           |
| \r          | Basic RegExp    | 匹配一个回车符                                               |
| \f          | Basic RegExp    | 匹配一个换页符                                               |

> 1. \* 和 + 限定符都是贪婪的，即会尽可能多的匹配字符（最长子串），在它们后面加上 ？限定符并指定使用 Perl 正则模式实现非贪婪或最小字串匹配。
> 2. 特殊符号要匹配本身，需进行转义。

## 使用 grep + 正则查看文件内容

> grep 默认支持 Basic RegExp 语法，可以通过传递选项 `-E` 以支持扩展正则表达式语法（相当于 `egrep` 命令）或 `-P` 以支持 Perl 语言的正则表达式语法。

测试文件内容的内容：
```bash
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

## sed

`sed -r` 支持扩展正则。
