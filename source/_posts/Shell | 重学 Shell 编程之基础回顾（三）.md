---
title: 重学 Shell 编程之基础回顾（三）
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

## Shell 条件测试

条件测试只会得到真与假两种结果，常与 if 判断语句结合进行具有一定复杂逻辑的工作。以下是 shell 条件测试中一些比较常用的语法：

> 条件测试返回 0 时表示真，返回其他非 0 值时表示假，跟其他编程语言不一致！

| 语法                 | 描述                                                         |
| -------------------- | ------------------------------------------------------------ |
| test \<测试表达式\>  | 利用 test 命令进行条件测试，注意 test 命令和参数之间存在空格 |
| [ \<测试表达式\> ]   | 推荐。等价于上条语法，注意左右中括号和表达式之间存在空格     |
| [[ \<测试表达式\> ]] | 扩展语法，较新，部分解释器可能不支持，注意左右双中括号和表达式之间存在空格 |
| ((\<测试表达式\>))   | 一般用于 if 语句，左右双小括号和表达式之间可以不存在空格     |

### test \<测试表达式\> 和 [ <测试表达式>  ] 语法

这两种语法需要记忆一些涉及文件、逻辑判断使用的特殊选项符号，将其整理如下：

#### 文件测试语法

| 文件测试语法      | 描述                                                                |
| --------------- | --------------------------------------------------------------------|
| *归类*           | **文件存在、类型判断**                                                |
| -e pathname     | 当由 pathname 指定的文件或目录存在时返回真                               |
| -s filename     | 当 filename 存在并且文件大小大于 0 时返回真                              |
| -f filename     | 当 filename 存在并且是常规文件时返回真                                   |
| -d pathname     | 当 pathname 存在并且是一个目录时返回真                                  |
| -b filename     | 当 filename 存在并且是块文件时返回真                                     |
| -c filename     | 当 filename 存在并且是字符文件时返回真                                   |
| -S filename     | 当 filename 存在并且是 socket 时返回真                                  |
| -p filename     | 当 filename 存在并且是命名管道时返回真                                    |
| -h filename     | 当 filename 存在并且是符号链接文件时返回真 (或 -L filename)               |
| -t fd           | 当 fd 是与终端设备相关联的文件描述符时返回真                                |
| *归类*           | **文件权限位判断**                                                     |
| -r pathname     | 当由 pathname 指定的文件或目录存在并且可读时返回真                         |
| -w pathname     | 当由 pathname 指定的文件或目录存在并且可写时返回真                         |
| -x pathname     | 当由 pathname 指定的文件或目录存在并且可执行时返回真                        |
| -u pathname     | 当由 pathname 指定的文件或目录存在并且设置了 SUID 位时返回真                |
| -g pathname     | 当由 pathname 指定的文件或目录存在并且设置了 SGID 位时返回真                |
| -k pathname     | 当由 pathname 指定的文件或目录存在并且设置了"粘滞"位时返回真                 |
| -O pathname     | 当由 pathname 存在并且被当前进程的有效用户 id 的用户拥有时返回真(字母 O 大写) |
| -G pathname     | 当由 pathname 存在并且属于当前进程的有效用户 id 的用户的用户组时返回真        |
| *归类*           | **两个文件的比较**                                                      |
| file1 -nt file2 | file1 比 file2 新时返回真                                               |
| file1 -ot file2 | file1 比 file2 旧时返回真                                               |
| file1 -ef file2 | file1 和 file2 是同一个文件的硬链接时返回真                               |

#### 逻辑测试语法

| 逻辑测试语法 | 描述                                        |
| ---------- | ----------------------------------------- |
| -a         | 逻辑与，操作符两边均为真，结果为真，否则为假。   |
| -o         | 逻辑非，操作符两边一边为真，结果为真，否则为假。 |
| !          | 取非，条件为假，结果为真。                    |

#### 常见字符串测试语法

| 常见字符串测试语法 | 描述                                         |
| --------------- | -------------------------------------------- |
| -z string       | 字符串 string 为空串(长度为0)时返回真         |
| -n string       | 字符串 string 为非空串时返回真                |
| str1 = str2     | 字符串 str1 和字符串 str2 相等时返回真         |
| str1 == str2    | 同 =                                         |
| str1 != str2    | 字符串 str1 和字符串 str2 不相等时返回真       |
| str1 < str2     | 按字典顺序排序，字符串 str1 在字符串 str2 之前 |
| str1 > str2     | 按字典顺序排序，字符串 str1 在字符串 str2 之后 |

#### 常见数值测试语法

| 常见数值测试语法   | 描述                            |
| ---------------- | ------------------------------- |
| int1 -eq int2    | 如果 int1 等于 int2，则返回真     |
| int1 -ne int2    | 如果 int1 不等于 int2，则返回真   |
| int1 -lt int2    | 如果 int1 小于 int2，则返回真     |
| int1 -le int2    | 如果 int1 小于等于 int2，则返回真 |
| int1 -gt int2    | 如果 int1 大于 int2，则返回真     |
| int1 -ge int2    | 如果 int1 大于等于 int2，则返回真 |

> 单中括号语法引用变量除了要使用 $ 符号还需要用引号将变量括起来。

### [[ <测试表达式> ]] 语法

大部分情形下，双中括号语法与 test 和单中括号语法通用，但是仍存在以下区别：

#### 逻辑测试

[[ <测试表达式> ]] 语法中使用 &&、|| 表示逻辑与和逻辑非，但在 [ <测试表达式> ] 语法中使用 -a 和 -o 运算符。

#### 通配符和正则

[[ <测试表达式> ]] 语法中支持使用通配符和正则表达式。

#### 字符串

[[ <测试表达式> ]] 语法中匹配字符串或通配符不需要引号括起来。

### ((<测试表达式>)) 语法

((<测试表达式>)) 只支持算术运算比较。

### 案例：判断磁盘是否挂载且挂载点对当前用户是否可读可写

{% codeblock lang:bash check_disk.sh %}
#!/bin/sh
dev_name="sdb1"
dev_path="/dev/$dev_name"
mnt_path="/mnt/data"
if [ -b "$dev_path" -a -r "$mnt_path" -a -w "$mnt_path" ]; then
  echo "磁盘 $dev_path 已挂载且当前用户可读写！"
else
  echo "磁盘未挂载或对当前用户不可读或不可写！"
fi
{% endcodeblock %}

## Shell 分支语句结构

### if 判断语句

if 语句支持单分支和多分支选择结构，单/双分支语句的一般格式是：

```bash
if 条件; then
  条件为真执行的语句
else
  条件为假执行的语句  # 单分支时不用 else 的部分
fi
```

对于多分支：

```bash
if 条件1; then
  条件1成立时执行的语句
elif 条件2; then
  条件2成立时执行的语句
elif 条件n; then
  条件n成立时执行的语句
else
  以上条件都不成立时执行的语句
fi
```

与其他编程语言类似，条件判断语句中可以使用 break 关键字进行跳出。

#### 案例：编写脚本监测可用剩余内存并发邮件报警

要求：

1. 当内存可用大小小于 100m 时发邮件进行报警；

2. 将脚本加入 crontab 任务，定时每 30 分钟执行一次。

{% codeblock lang:bash check_mem.sh %}
#!/bin/sh
mem_free=`free -m|awk 'NR==2 {print $NF}'`
if test $mem_free -lt '100'; then
  local msg="当前可用内存为 ${mem_free}m，已不足 100m！"
  echo msg | mail -s '服务器内存告警' user@mailsrv.com
fi
{% endcodeblock %}

执行 `crontab -e`，写入定时任务：

```crontab
* /30 * * * * /bin/bash /path/to/scripts/check_mem.sh
```

### case in 判断语句

Case in 语句一般用于多分支且判断条件较简单的分支语句结构。Case in 语句的一般格式是：

```bash
case 表达式 in
  匹配模式1)
    匹配模式1成立时执行的语句
  ;;
  匹配模式2)
    匹配模式2成立时执行的语句
  ;;
  匹配模式n)
    匹配模式n成立时执行的语句
  ;;
  *)
    其他匹配模式都不成立时执行的语句
esac
```

其中，表达式既可以是一个变量、一个数字、一个字符串，还可以是一个数学计算表达式，或者是命令的执行结果，只要能够得到表达式的值就可以；匹配模式可以是一个数字、一个字符串，甚至是一个简单的正则表达式；`;;` 的作用相当于其他编程语言的 case 语句中的 break；`*)` 的作用相当于其他编程语言的 case 语句中的 default。

#### 案例：模拟实现服务启停、重启、查询的管理脚本

{% codeblock lang:bash service.sh %}
#!/bin/sh
case $1 in
  start)
    echo "服务启动成功！"
  ;;
  stop)
    echo "服务停止成功！"
  ;;
  restart)
    echo "重启服务成功！"
  ;;
  status)
    echo "呃。。。。。。"
  ;;
  *)
    echo "Usage: $0 {start|stop|restart|status}"
    exit 1
esac
exit 0
{% endcodeblock %}

## 函数

基本格式：

```bash
# 定义函数，function、return 关键字可省略 
## 标准写法
function func_name(){
  do_something...
  return ret_val
}
## 省略花括号
function func_name(
  do_something...
  return ret_val
)
## 一般写法
func_name(){
  do_something...
  retuen ret_val
}
# 调用函数，其中 arg1 arg2 ... argN 都是参数
func_name arg1 arg2 ... argN
# 必须先定义再调用
```

在日常工作中，一把将 shell 的函数定义和函数调用分布在两个脚本文件中，此行为有点类似 C 语言编程中设置头文件的行为，其初衷是模块化编程。

{% codeblock lang:bash def_func.sh %}
function my_func(){
  echo "单独定义的函数。"
  return 0
}
{% endcodeblock %}

调用独立定义的函数：

{% codeblock lang:bash exec_func.sh %}
#!/bin/sh
func_path="/path/to/def_func.sh"
# 判断函数文件是否存在
[ -f func_path ] && . func_path || {echo "找不到文件：$func_path" 2>&1; exit 1}
# 调用函数
my_func
{% endcodeblock %}

函数内部也有一些特殊变量，可进行传递的参数等的处理，可参考调用 shell 脚本时内部的特殊变量，脚本的特殊变量不会被函数内部自动继承，需要层层传递。
