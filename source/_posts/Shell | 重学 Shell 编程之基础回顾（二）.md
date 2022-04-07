---
title: 重学 Shell 编程之基础回顾（二）
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

## 引用命令的结果

* \`command\`
* $(command)

例：

```bash
[skinyi@fedora ~]$ echo "当前日期是：`date`"
当前日期是：2022年 03月 31日 星期四 11:13:50 CST
[skinyi@fedora ~]$ echo "当前用户的 id 是：$(id -u)"
当前用户的 id 是：1000
```

## Shell 运算符

Shell 和其它编程语言不同，Shell 不能直接进行算数运算，必须使用数学计算命令。以下是 Shell 中使用的算数运算符：

| Shell 算数运算符         | 意义（带 **\*** 代表常用）                        |
| :--------------------- | :-------------------------------------------- |
| +、-                   | 加法（或正号）、减法（或负号） **\***              |
| \*、/、%               | 乘法、除法、取余（取模） **\***                    |
| \*\*                   | 幂运算 **\***                                      |
| ++、--                 | 自增、自减 **\***                                  |
| !、&&、\|\|            | 逻辑非（取反）、逻辑与（and）、逻辑或（or） **\*** |
| <、<=、>、>=           | 小于、小于等于、大于、大于等于                |
| ==、!=、=              | 相等、不等、对于字符串 = 也可表示相当于 **\***     |
| <<、>>                 | 左移位、右移位                                |
| ~、\|、&、^            | 按位取反、按位异或、按位与、按位或            |
| =、+=、-=、\*=、/=、%= | 赋值运算符，a+=1 相当于 a=a+1，后面类推 **\***     |

要想让数学计算发挥作用，必须使用数学计算命令，Shell 中常用的数学计算命令如下表所示。

| Shell 运算操作符与运算命令 | 意义                                                  |
| -------------------------- | ----------------------------------------------------- |
| (())                       | 用于整数运算的常用运算符，效率很高                    |
| let                        | 用于整数运算，类似于”(())“                            |
| expr                       | 可用于整数运算，但还有很多其他的额外功能              |
| bc                         | Linux 下的一个计算器程序（适合整数及小数运算）        |
| \$[]                       | 用于整数运算                                          |
| awk                        | awk 既可以用于整数运算，也可以用于小数运算            |
| declare                    | 定义变量值和属性，-i 参数可以用于定义整型变量，做运算 |

### (()) 运算符

用法示例：

```bash
# 定义变量：
[skinyi@fedora ~]$ i=1
# 错误示例：
[skinyi@fedora ~]$ i=((i+1));echo $i
bash: 未预期的符号“(”附近有语法错误
[skinyi@fedora ~]$ echo ((i+1))
bash: 未预期的符号“(”附近有语法错误
# 正确示例：
[skinyi@fedora ~]$ i=$((i+1));echo $i
2
[skinyi@fedora ~]$ ((i=i+1));echo $i
3
[skinyi@fedora ~]$ i=$((i>1));echo $i
1
[skinyi@fedora ~]$ ((i=i>1));echo $i
0
[skinyi@fedora ~]$ echo $((i<1))
1
```

> 总结：
>
> 内部赋值（不直接引用）：((var=表达式))；
> 外部赋值（直接引用）：var=$((表达式))、echo $((表达式))。

### let 指令

用法示例：

```bash
# 正确示例：
[skinyi@fedora ~]$ let num=1+1; echo $num
2
[skinyi@fedora ~]$ let num=num+2; echo $num
4
[skinyi@fedora ~]$ let num+=2; echo $num
6
# 错误示例：
[skinyi@fedora ~]$ let num=2>1; echo $num
2               # 无法根据比较结果赋值
[skinyi@fedora ~]$ let num**2; echo $num
2               # 只能是以赋值的形式进行计算
[skinyi@fedora ~]$ let num=$((2>1)); echo $num
1               # 仅用双括号就能实现效果，用 let 冗余
```

#### 案例：监测网站服务的存活状态

以下示例展示了一个简单的网站服务存活检测的脚本，其原理是对网站定时发送 head 请求以确认网站服务可用，当请求失败次数达到一定次数后给当前系统用户发送邮件告知网站服务出了问题并退出脚本的执行。要想再严格些可以考虑认为一定周期内达到最大失败次数才认为网站挂了，还有在脚本传参时可以进行 url 格式的正则检查，当然这里只是个小案例，主要还是将之前学到的没学到的东西都用了一遍，加深记忆。

{% codeblock lang:bash check_website_status.sh %}
#!/bin/sh
## Check web service status

req_url=''            # 请求的网址 url
req_timeout='5'       # 请求超时时间
req_fails='0'         # 最大尝试请求次数
req_alert_fails='10'  # 触发报警的最大尝试次数
req_wait_time='1'     # 每次请求间隔时间

check_parameters(){
  if [ $# -ne 1 ]; then 
    printf "错误：未指定或指定了多个参数！\n用法：$0 <url>\n" 2>&1
    exit 1
  else
    req_url=$1
  fi
}

send_mail_alert(){
  if [ $req_fails -ge $req_alert_fails ]; then
    local mail_message="自脚本启动以来共发生了${req_fails}次失败的请求，已达最大失败上限，请及时进行人工检查处理。\n"
    echo -e $mail_message | mail -s "警告：你的网站服务可能已经不可用" $USER
    exit 0 # 发送邮件后就退出，防止邮件轰炸
  fi
}

check_status(){
  while true; do
    local date=`date --rfc-3339=seconds`
    # 使用 head 请求来进行网站是否存活检测
    curl --head --connect-timeout ${req_timeout} -s -f ${req_url} -o /dev/null
    # 若命令返回非 0 则表明请求失败
    if [ $? -ne 0 ]; then 
      let req_fails+=1  # 请求失败次数自增
      echo "${date%+*} 时进行了一次失败的请求"
      send_mail_alert   # 调用报警判断是否报警
    else
      echo "${date%+*} 时进行了一次成功的请求"
    fi
    # 每次请求间隔一段时间
    sleep $req_wait_time
  done
}

check_parameters $@     # 函数不会继承 shell 脚本的参数，调用时需进行传参
check_status
{% endcodeblock %}

### expr 命令

使用 `expr --help` 命令查看 `expr` 命令帮助信息。

```bash
[skinyi@fedora ~]$ expr --help
用法：expr 表达式
　或：expr 选项

      --help            显示此帮助信息并退出
      --version         显示版本信息并退出

将表达式的值列印到标准输出，分隔符下面的空行可提升算式优先级。
可用的表达式有：

  ARG1 | ARG2       若ARG1 的值不为 0 或者为空，则返回 ARG1，否则返回 ARG2

  ARG1 & ARG2       若两边的值都不为 0 或为空，则返回 ARG1，否则返回 0

  ARG1 < ARG2       ARG1 小于 ARG2
  ARG1 <= ARG2      ARG1 小于或等于 ARG2
  ARG1 = ARG2       ARG1 等于 ARG2
  ARG1 != ARG2      ARG1 不等于 ARG2
  ARG1 >= ARG2      ARG1 大于或等于 ARG2
  ARG1 > ARG2       ARG1 大于 ARG2

  ARG1 + ARG2       计算 ARG1 与 ARG2 相加之和
  ARG1 - ARG2       计算 ARG1 与 ARG2 相减之差

  ARG1 * ARG2       计算 ARG1 与 ARG2 相乘之积
  ARG1 / ARG2       计算 ARG1 与 ARG2 相除之商
  ARG1 % ARG2       计算 ARG1 与 ARG2 相除之余数

  字符串 : 表达式               定位字符串中匹配表达式的模式

  match 字符串 表达式           等于"字符串 : 表达式"
  substr 字符串 偏移量 长度      替换字符串的子串，偏移的数值从 1 起计
  index 字符串 字符             在字符串中发现字符的地方建立下标，或者标0
  length 字符串                 字符串的长度
  + 记号                     将给定记号解析为字符串，即使它是一个类似
                               “match”或运算符"/"那样的关键字

  ( 表达式 )                 给定<表达式>的值

请注意有许多运算操作符都可能需要由 shell 先实施转义。
如果参与运算的 ARG 自变量都是数字，比较符就会被视作数学符号，否则就是多义的。
模式匹配会返回"\"和"\"之间被匹配的子字符串或空(null)；如果未使用"\"和"\"，
则会返回匹配字符数量或是 0。

若表达式的值既不是空也不是 0，退出状态值为 0；若表达式的值为空或为 0，
退出状态值为 1。如果表达式的句法无效，则会在出错时返回退出状态值 3。
```

用法示例：

```bash
# 使用时最好都进行转义防止与 shell 特殊含义符号冲突
[skinyi@fedora ~]$ expr 1 + 3
4
[skinyi@fedora ~]$ expr 2 \* 3
6
[skinyi@fedora ~]$ expr 2 * 3
expr: 语法错误：未预期的参数
[skinyi@fedora ~]$ expr 2 \+ 5
7
# expr match 模式匹配，输出匹配字符个数
[skinyi@fedora ~]$ expr .bashrc.d \: ".*"
9
[skinyi@fedora ~]$ expr match .bashrc.d ".*\.d"
9
[skinyi@fedora ~]$ expr .bashrc.d \: ".*b"
2
[skinyi@fedora ~]$ expr .bashrc.d \: ".*p"
0
[skinyi@fedora ~]$ expr match zhiyaoniqingqingyixiao wodexinjiumizui
0
[skinyi@fedora ~]$ expr match zhiyaoniqingqingyixiao zhiyaoni
8
[skinyi@fedora ~]$ expr match zhiyaoniqingqingyixiao zhini
0
# expr length 获取字符串长度
[skinyi@fedora ~]$ expr length 123456789
9
```

#### 案例：参数传入一个文件的名字，判断其是否是 mp3 文件

{% codeblock lang:bash check_mp3.sh %}
#!/bin/sh
# && 分隔的两条语句前面的执行成功（退出码为 0）后才执行后面的语句，||反之
expr "$1" ":" ".*\.mp3\$" 2>>/dev/null && echo "这是个 mp3 文件" || echo "这不是 mp3 文件"
{% endcodeblock %}

#### 案例：参数传入一句话（拼音或单词），判断其长度大于 3 的拼音、单词个数

{% codeblock lang:bash count_more_than_3.sh %}
#!/bin/sh
str_count=0
for word in $@; do
  if [ `expr length $word` -gt 3 ]; then
    let str_count+=1
  fi
done
echo "大于 3 的拼音/单词个数为：$str_count"
{% endcodeblock %}

### bc 命令

bc 命令默认是以交互模式运行的，即执行 `bc` 命令会直接进入交互模式，用户可以输入表达式进行计算并得到结果，可以使用管道来传递单个表达式进行计算。bc 同时支持整数和小数计算。

用法示例：

```bash
[skinyi@fedora ~]$ echo 4.0/2.0 | bc
2
[skinyi@fedora ~]$ echo 9/3 | bc
3
```

#### 案例：计算 1 到 100 的和

```bash
#!/bin/sh
# echo {1..100} 会打印以空格作为分隔符的 1 到 100 的数字
# tr 命令：用于转换或删除文件中的字符，`tr ' ' '+'` 代表将输入中的空格转换为 + 再输出
echo "1 到 100 的和是 $(echo {1..100} | tr ' ' '+' | bc)。"
# 或者
# seq 用于生成序列，-s 选项指定分隔符为 +
echo "1 到 100 的和是 seq -s '+' 100 | bc。"
# 使用双括号
echo "1 到 100 的和是 $((`seq -s '+' 100`))。"
# 使用 expr，注意这次的分隔符
echo "1 到 100 的和是 $(seq -s ' + ' 100 | xargs expr)。"
```

### awk 命令

用法示例：

```bash
[skinyi@fedora ~]$ echo "4.2 2.4" | awk '{print $1+$2}'
6.6
[skinyi@fedora ~]$ echo "4.2 2.4" | awk '{print $1*$2}'
10.08
[skinyi@fedora ~]$ echo "4.2 2.4" | awk '{print $1+4*$2}'
13.8
```

### $[表达式]

`$[表达式]` 仅能用于整数运算，用法示例：

```bash
[skinyi@fedora ~]$ echo $[2**2]
4
[skinyi@fedora ~]$ echo $[2+2+2]
6
[skinyi@fedora ~]$ num=5
[skinyi@fedora ~]$ echo $[num+2+2]
9
```

## read 指令

read 指令用于从标准输入设备中读取输入。

```bash
[skinyi@fedora ~]$ read --help
read: read [-ers] [-a 数组] [-d 分隔符] [-i 缓冲区文字] [-n 读取字符数] [-N 读取字符数] [-p 提示符] [-t 超时] [-u 文件描述符] [名称 ...]
    从标准输入读取一行并将其分为不同的域。
    
    从标准输入读取单独的一行，或者如果使用了 -u 选项，从文件描述符 FD 中读取。
    该行被分割成域，如同词语分割一样，并且第一个词被赋值给第一个 NAME 变量，第二
    个词被赋值给第二个 NAME 变量，如此继续，直到剩下所有的词被赋值给最后一个 NAME
    变量。只有 $IFS 变量中的字符被认作是词语分隔符。
    
    如果没有提供 NAME 变量，则读取的行被存放在 REPLY 变量中。
    
    选项：
      -a array  将词语赋值给 ARRAY 数组变量的序列下标成员，从零开始
      -d delim  持续读取直到读入 DELIM 变量中的第一个字符，而不是换行符
      -e        使用 Readline 获取行
      -i text   使用 TEXT 文本作为 Readline 的初始文字
      -n nchars 读取 nchars 个字符之后返回，而不是等到读取换行符。
                但是分隔符仍然有效，如果遇到分隔符之前读取了不足 nchars 个字符。
      -N nchars 在准确读取了 nchars 个字符之后返回，除非遇到文件结束符或者读超时，
                任何的分隔符都被忽略
      -p prompt 在尝试读取之前输出 PROMPT 提示符并且不带
                换行符
      -r        不允许反斜杠转义任何字符
      -s        不回显终端的任何输入
      -t timeout        如果在 TIMEOUT 秒内没有读取一个完整的行则超时并且返回失败。
                TMOUT 变量的值是默认的超时时间。TIMEOUT 可以是小数。
                如果 TIMEOUT 是 0，那么仅当在指定的文件描述符上输入有效的时候，
                read 才返回成功；否则它将立刻返回而不尝试读取任何数据。
                如果超过了超时时间，则返回状态码大于 128
      -u fd     从文件描述符 FD 中读取，而不是标准输入
    
    退出状态：
    返回码为零，除非遇到了文件结束符、读超时（且返回码不大于128）、
    出现了变量赋值错误或者无效的文件描述符作为参数传递给了 -u 选项。
```

用法示例：

```bash
[skinyi@fedora ~]$ read -p "给你五秒时间输入你的名字：" -t 5 name
给你五秒时间输入你的名字：[skinyi@fedora ~]$
[skinyi@fedora ~]$ read -p "给你五秒时间输入你的名字：" -t 5 name
给你五秒时间输入你的名字：skinyi
[skinyi@fedora ~]$ echo $name
skinyi
[skinyi@fedora ~]$ read -p "输入你的名字和年龄："  name age ;echo "$name $age"
输入你的名字和年龄：skinyi # 未输入的变量默认为空
skinyi 
[skinyi@fedora ~]$ read -p "输入你的名字和年龄："  name age ;echo "$name $age"
输入你的名字和年龄：skinyi 18
skinyi 18
```
