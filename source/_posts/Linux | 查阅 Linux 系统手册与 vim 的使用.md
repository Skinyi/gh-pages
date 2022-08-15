---
title: 查阅 Linux 系统手册与 Vim 的使用
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

## Linux 操作手册

​Linux 本地系统提供的一个 linux 操作文档是系统手册，或称为 man page，可以通过 `man` 命令从命令行进行访问查询。man page 源自过去的 Linux 程序员手册，该手册篇幅很长，足以打印成多个章节，每个章节中都包含了有关特定主题的信息。以下列举了一些常见的章节。

| 章节  | 内容类型 |
| --- | --- |
| 1   | 用户命令（可执行命令和 shell 程序） |
| 2   | 系统调用（从用户空间调用的内核例程） |
| 3   | 库函数（由程序库提供） |
| 4   | 特殊文件（如设备文件） |
| 5   | 文件格式（用于许多配置文件和结构） |
| 6   | 游戏（过去的有趣程序章节） |
| 7   | 惯例、标准和其他（协议、文件系统） |
| 8   | 系统管理和特权命令（维护任务） |
| 9   | Linux 内核 API（内核调用） |

​为区分不同章节中的相同主题名称，man page 参考中在主题后附上了章节编号（用括号括起）：如 passwd(1) 介绍使用 passwd 命令更改用户密码，而 passwd(5) 说明用于存储本地用户账户的 /etc/passwd 配置文件格式。

```bash
man [THEME_NUM] TOPIC # 使用 man 命令查阅 TOPIC 的相关文档，不指定 THEME_NUM 则默认为 1
# 例子：
man passwd # 查看 passwd 命令的帮助信息
man 5 passwd # 查看 passwd 配置文件的介绍信息
```

## 阅读文档相关

### 导航与搜索

​man page 底层使用 `less` 作为其查看器，具体的导航与查找的操作信息可参见 *04. Linux 文件操作、重定向和管道* 中与 `less` 命令有关的相关章节。

### 小标题

​man page 每个主题都划分为几个部分，每个部分都有固定的小标题，大多数主题都共享相同的小标题，且这些小标题出现的循序都是约定俗成的。每个主题下不可能包含所有的小标题，因为有些小标题不适用于当前主题。以下列举了一些常见的小标题：

| 标题  | 描述  |
| --- | --- |
| NAME | 主题名称，通常是命令或文件名，后跟非常简短的描述 |
| SYNOPSIS | 命令语法的概要 |
| DESCRIPTION | 提供对主题的基本理解的深度描述 |
| OPTIONS | 对命令执行选项的说明 |
| EXAMPLES | 有关如何使用命令、功能或文件的示例 |
| FILES | 与 man page 相关的文件和目录的列表 |
| SEE ALSO | 相关的信息，通常是其他 man page 的主题 |
| BUGS | 软件中的已知错误 |
| AUTHOR | 有关参与编写该主题的人员的信息 |

### 根据关键字搜索 man page

```bash
man -k keyword # 在 manpage 中搜索跟 keyword 有关的 man page 主题和章节号
# 需更新 whatis 库后才能使用，否则提示：[KEYWORD]: nothting appropriate
# 通过 root 使用 makewhatis 指令来手动建立资料库 
apropos 与 man -k 相同， whatis 与 man -f 相同。
# 好吧，我系统上也没装 makewhatis ，知道就行算了😒
```

## pinfo 操作手册

​通常而言，man page 文档更加正式，其采用独立文本文件的结构，pinfo 文档更加全面和灵活，采用超文本文档的结构。

```bash
pinfo [TOPIC] # 打开关于 TOPIC 的 info 文档，不指定 TOPIC 则打开顶级目录
```

如果系统中没有针对该主题的 info 条目，则 pinfo 就会去查找 man page 并显示相应的内容。

​pinfo 和 man 命令的键盘导航操作的区别如下：

| 导航操作 | pinfo | man |
| --- | --- | --- |
| 向前（下）滚动一个屏幕 | PgDn 或 SPACE | PgDn 或 SPACE |
| 向后（上）滚动一个屏幕 | PgUp 或 b | PgUp 或 b |
| 显示主题目录 | d   | 无   |
| 向前（下）滚动半个屏幕 | 无   | d   |
| 显示主题的父节点 | u   | 无   |
| 显示主题的顶部（上部） | h   | g   |
| 向后（上）滚动半个屏幕 | -   | u   |
| 向前（下）滚动到下一个超链接 | ↓   | 无   |
| 打开光标处的主题 | ENTER | 无   |
| 向前（下）滚动一行 | 无   | ↓ 或 ENTER |
| 向后（上）滚动到上一超链接 | ↑   | 无   |
| 向后（上）滚动一行 | 无   | ↑   |
| 搜索某种模式 | /string | /sting 和 ?string |
| 显示主题中的下一节点（章节） | n   | 无   |
| 重复之前的向前（下）搜索 | / + ENTER | n   |
| 显示主题中的上一节点（章节） | p   | 无   |
| 重复之前的向后（上）搜索 | 无   | SHIFT + n |
| 退出程序 | q   | q   |

## Vim Cheat Sheet

​中文版 Cheat Sheet 可以参阅[这里](https://vim.rtorr.com/lang/zh_cn)。