---
title: 使用 SELinux 确保系统安全性
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

## SELinux 简述

​SELinux 是 Security Enhanced Linux 的简称。

​SELinux 在 Linux 中具有重要的安全用途，它可以允许或拒绝访问文件及其他资源，且精准度相比用户权限要大幅提高。SELinux 相比普通的文件权限控制提供了更多安全细节，不仅可以限制谁能读取、写入、执行某文件，还能控制如何来使用该文件。

​SELinux 由应用开发人员定义的若干策略组成，这些策略准确声明了对于应用使用的每个二进制可执行文件、配置文件和数据文件，哪些操作和访问权限是恰当且被允许的。这被称为目标政策，因为会编写一个策略以涵盖单个应用的活动。策略声明了各个程序、文件和网络端口上放置的预定义标签。

​SELinux 实施了一组可以防止一个应用程序的弱点影响其他应用或基础系统的访问规则。SELinux 提供了一个额外的安全层，此外还增加了一层复杂结构，这使得 SELinux 的学习成本增加，但执行政策意味着系统某一部分的弱点不会扩散到其他部分。如果 SELinux 在特定的子系统上运行不佳，则可以关闭该特定服务的执行，直至找到潜在问题的解决方案。

​SELinux 有三种模式：

- 强制（*Enforcing*）：SELinux 强制执行访问控制规则。计算机通常在此模式下运行；
- 许可（*Permissive*）：SELinux 处于活动状态，但不强制执行访问控制规则，而是记录违反规则的警告。该模式主要用于测试和故障排除；
- 禁用（*Disable*）：SELinux 完全关闭 —— 不拒绝任何 SELinux 违规，甚至不予记录。不建议禁用 SELinux。

## SELinux 工作原理

​SELinux 是用于确定哪个进程可以访问哪些文件、目录和端口的一组安全规则。每个文件、进程、目录和端口都具有专门的安全标签，称为 SELinux 上下文。上下文是一个名称，SELinux 策略使用它来确定某个进程能否访问文件、目录或端口。除非显式规则授予访问权限，否则，在默认情况下，策略不允许任何交互。如果没有允许规则，则不允许访问。

​SELinux 标签具有多种上下文：*用户*、*角色*、*类型*和*敏感度*。目标策略会根据三个上下文（即类型上下文）来制定自己的规则。类型上下文名称通常以 *\_t* 结尾。

> ​    *unconfined_u*:*object_r*:*httpd_sys_content_t*:*s0* /var/www/html/file2
> 
> ​     SELinux 用户    角色            类型       级别         文件

​说明：

> Web 服务器的类型上下文是 *httpd_t*。通常位于 */var/www/html* 中的文件和目录的类型上下文是 *httpd_sys_content_t*。通常位于 */tmp* 和 */var/tmp* 中的文件和目录上下文是 *tmp_t*。Web 服务器端口的类型上下文是 *http_port_t*。

​Apache 具有类型上下文 *httpd_t*。有一个策略规则允许 Apache 访问具有 *httpd_sys_content_t* 类型上下文的文件和目录。默认情况下，在 *var/www/html* 和其他 Web 服务器目录中找到的文件具有 *httpd_sys_content_t* 类型上下文。策略中没有允许规则适用于通常位于 */tmp* 和 */var/tmp* 中的文件，因此不允许访问。在启用 SELinux 的情况下，破坏了Web服务器进程的恶意用户将无法访问 */tmp* 目录。

​MariaDB 服务器具有类型上下文 *mysqld_t*。默认情况下，在 */data/mysql* 中找到的文件具有 *mysqld_db_t* 类型上下文。该类型上下文允许 MariaDB 访问这些文件，但禁止其他服务（如Apache Web服务）访问。

> 许多处理文件的命令都使用 *-Z* 选项来显示或设置 SELinux 上下文。例如，`ps`、`ls`、`cp` 和 `mkdir` 都使用 *-Z* 选项显示或设置 SELinux 上下文。
> 
> ```bash
> [root@localhost ~]\# ps axZ | head -n 3
> LABEL                               PID TTY      STAT   TIME COMMAND
> system_u:system_r:init_t:s0           1 ?        Ss     0:05 /usr/lib/systemd/systemd --switched-root --system --deserialize 18
> system_u:system_r:kernel_t:s0         2 ?        S      0:00 [kthreadd]
> [root@localhost ~]\# ls -Z /home
> unconfined_u:object_r:user_home_dir_t:s0 student
> [root@localhost ~]\# ls -Z /var/log/cups
> system_u:object_r:cupsd_log_t:s0 access_log
> system_u:object_r:cupsd_log_t:s0 access_log-20210819
> system_u:object_r:cupsd_log_t:s0 error_log
> system_u:object_r:cupsd_log_t:s0 error_log-20210819
> system_u:object_r:cupsd_log_t:s0 page_log
> system_u:object_r:cupsd_log_t:s0 page_log-20210819
> ```

## SELinux 工作模式

​SELinux 子系统提供了显示和更改模式的工具。要确定当前的 SELinux 模式，可以运行 `getenforce` 命令：

```bash
[root@localhost ~]\# getenforce
Enforcing
```

​要临时更改 SELinux 工作模式，可以使用 `setenforce` 命令：

```bash
[root@localhost ~]\# setenforce
usage:  setenforce [ Enforcing | Permissive | 1 | 0 ]
```

> 可以在系统启动时通过将向内核传递参数来设置 SELinux 模式：内核参数 *enforcing=0* 将以许可模式启动系统；值 *enforcing=1* 则设置强制模式。此外，您还可以通过传递内核参数 *selinux=0* 来彻底禁用 SELinux。值 *selinux=1* 将启用 SELinux。

​可以在 */etc/selinux/config* 文件来持久配置 SELinux。内核参数会覆盖此设置。

```ini
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected. 
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

## SELinux 上下文

​在运行 SELinux 的系统上，所有进程和文件都会有响应的标签。标签代表了与安全有关的信息，称为 SELinux 上下文。

​新文件通常从父目录继承其 SELinux 上下文，从而确保它们具有适当的上下文。但是，有两种不同的方式可能会破坏该继承过程。首先，如果您在与最终目标位置不同的位置创建文件，然后移动文件，则该文件将具有创建它时所在目录的 SELinux 上下文，而不是目标目录的 SELinux 上下文。其次，如果是复制一个保留 SELinux 上下文的文件（正如使用 `cp -a` 命令），则 SELinux 上下文将反映原始文件的位置。

​以下示例演示了继承关系及存在的缺陷。以 */tmp* 中创建的两个文件为例，一个移动到 */var/www/html*，另一个复制到同一目录中。移动到 */var/www/html* 目录的文件保留了 */tmp* 目录的文件上下文。复制到 */var/www/html* 目录中的文件则继承了 */var/www/html* 目录的 SELinux 上下文。

​使用 `ls -Z` 命令显示文件的 SELinux 上下文。

```bash
[root@localhost ~]\# ls -Z /var/lib/nfs/etab
system_u:object_r:var_lib_nfs_t:s0 /var/lib/nfs/etab
```

​使用 `ls -Zd` 命令显示目录的 SELinux 上下文。

```bash
[root@localhost ~]\# ls -dZ /var/lib/nfs
system_u:object_r:var_lib_nfs_t:s0 /var/lib/nfs
```

### 更改文件的 SELinux 上下文

​用于更改文件 SELinux 上下文的命令包括：`semanage fcontext`、`restorecon` 和 `chcon`。

​为文件设置 SELinux 上下文的首选方法是使用 `semanage fcontext` 命令来声明文件的默认标签，然后使用 `restorecon` 命令将该上下文应用于文件。这样可确保标签符合预期，即便在对文件系统完全重新标记之后也是如此。

​`chcon` 命令更改 SELinux 上下文。`chcon` 设置存储在文件系统中的文件安全上下文。它对于测试和实验很有用。但是，<b>它不会将上下文更改保存到 SELinux 上下文数据库中。</b>当 `restorecon` 命令运行时，`chcon` 命令所做的更改也同样无法保留。此外，如果对整个文件系统进行重新标记，则使用 `chcon` 更改过的文件的 SELinux 上下文将恢复。

```bash
# 1. 创建测试目录并查看其 SELinux 上下文
[root@localhost ~]\# mkdir testdir
[root@localhost ~]\# ls -dZ testdir
unconfined_u:object_r:admin_home_t:s0 testdir
# 2. 使用 chcon 命令修改其 SELinux 上下文
[root@localhost ~]\# chcon -t var_lib_nfs_t testdir
[root@localhost ~]\# ls -dZ testdir
unconfined_u:object_r:var_lib_nfs_t:s0 testdir
# 3. 使用 restorecon 命令重置其 SELinux 上下文
[root@localhost ~]\# restorecon -v testdir
Relabeled /root/testdir from unconfined_u:object_r:var_lib_nfs_t:s0 to unconfined_u:object_r:admin_home_t:s0
[root@localhost ~]\# ls -dZ testdir
unconfined_u:object_r:admin_home_t:s0 testdir
```

### 定义 SELinux 默认文件上下文规则

​`semanager fcontext` 命令可显示和修改 `restorecon` 用来设置默认文件上下文的规则。它使用扩展正则表达式来指定路径和文件名。`fcontext` 规则中最常用的扩展正则表达式是 *\(/\.\*\)?*，表示 “可选择匹配后跟任何数量字符的/” 。它将会匹配在表达式前面列出的目录并递归地匹配该目录中的所有内容。

​`semanage fcontext` 命令的选项：

| 选项           | 描述          |
| ------------ | ----------- |
| -a, --add    | 添加指定对象类型的记录 |
| -d, --delete | 删除指定对象类型的记录 |
| -l, --list   | 列出指定对象类型的记录 |

> `restorecon` 命令依赖于 *policycoreutil* 软件包，`semanage` 命令依赖于 *policycoreutil-python* 软件包。

```bash
[root@localhost ~]\# semanage fcontext -l | grep var/lib/nfs
/var/lib/nfs(/.*)?                                 all files          system_u:object_r:var_lib_nfs_t:s0 
/var/lib/nfs/rpc_pipefs(/.*)?                      all files          <<None>>
```

​接 1 中的例子（文件已创建），此次永久设置其 SELinux 上下文：

```bash
# 4. 将 /root/testdir 的类型上下文永久修改为 var_lib_nfs_t（继承）
[root@localhost ~]\# semanage fcontext -a -t var_lib_nfs_t '/root/testdir(/.*)?'
[root@localhost ~]\# ls -dZ testdir/
unconfined_u:object_r:admin_home_t:s0 testdir/
[root@localhost ~]\# restorecon -RFvv testdir/
Relabeled /root/testdir from unconfined_u:object_r:admin_home_t:s0 to system_u:object_r:var_lib_nfs_t:s0
# 5. 尝试在此目录下创建文件再验证：
[root@localhost ~]\# touch testdir/test.txt
[root@localhost ~]\# ls -Z testdir/test.txt 
unconfined_u:object_r:var_lib_nfs_t:s0 testdir/test.txt
# 6. 删除刚才的配置
[root@localhost ~]\# semanage fcontext -d -t var_lib_nfs_t '/root/testdir(/.*)?'
[root@localhost ~]\# restorecon -RFvv testdir/
Relabeled /root/testdir from system_u:object_r:var_lib_nfs_t:s0 to system_u:object_r:admin_home_t:s0
Relabeled /root/testdir/test.txt from unconfined_u:object_r:var_lib_nfs_t:s0 to system_u:object_r:admin_home_t:s0
```

## 使用布尔值调整 SELinux 策略

​SELinux 布尔值是可更改 SELinux 策略行为的参数。SELinux 布尔值是可以启用或禁用的规则。安全管理员可以使用 SELinux 布尔值来有选择地调整策略。

> *selinux-policy-doc* 软件包附带的 SELinux man page 中介绍了可用布尔值的用途。使用 `man -k '_selinux'` 命令可以列出这些 man page。

​使用 `getsebool` 命令列出机器状态的布尔值。

```bash
# 选项：
-a  # 列出所有配置
# 例子：
[root@localhost ~]\# getsebool -a | head -n 10
abrt_anon_write --> off
abrt_handle_event --> off
abrt_upload_watch_anon_write --> on
antivirus_can_scan_system --> off
antivirus_use_jit --> off
auditadm_exec_content --> on
authlogin_nsswitch_use_ldap --> off
authlogin_radius --> off
authlogin_yubikey --> off
awstats_purge_apache_log_files --> off
[root@localhost ~]\# getsebool httpd_enable_homedirs
httpd_enable_homedirs --> off
```

​使用 `setsebool` 命令修改布尔值。

```bash
# 选项：
-P  # 将设置持久化到政策文件中
# 例子：
[root@localhost ~]\# setsebool httpd_enable_homedirs on
[root@localhost ~]\# getsebool -P httpd_enable_homedirs off
```

​使用 `semanage boolean -l` 将报告布尔值是否为持久值，并提供该布尔值的简短描述。

```bash
[root@localhost ~]\# semanage boolean -l | grep httpd_enable_homedirs 
httpd_enable_homedirs          (off  ,  off)  Allow httpd to enable homedirs
```

​使用 `semanage boolean -l -C` 列出当前状态与默认配置不同的布尔值.

```bash
[root@localhost ~]\# semanage boolean -l -C
```

## 排查和解决 SELinux 问题

​解决因 SELinux 阻止而使得某些应用无法访问某些文件时应注意以下原则：

- 在考虑任何调整之前，应了解到 SELinux 禁止意图访问的这一做法也许很正确，如 Web 服务器对 /home 的访问；
- 最常见的 SELinux 问题是使用不正确的文件上下文，这种情况下可以运行 `restorecon` 来更正因为移动文件产生的文件访问问题；
- 对于严苛限制性访问的另一个补救措施可以是调整布尔值，如 *ftpd_anon_write* 布尔值控制匿名 FTP 用户能否上传文件；
- SELinux 策略可能存在阻止合法访问的漏洞，此种情况极少。 

### 监控 SELinux 冲突

> 依赖软件包：*setroubleshoot-server*。安装后 SELinux 消息将会发送至 /var/log/messages。

​*setroubleshoot-server* 侦听 */var/log/audit/audit.log* 中的审核消息，并发送简短摘要到 /var/log/messages。该摘要包括 SELinux 冲突的唯一标识符（UUID），可用于收集更多信息。`sealert -l UUID` 命令用于生成特定事件的报告。使用 `sealert -a /var/log/audit/audit.log` 可以在该文件中生成所有事件的报告。

> 使用 `ausearch` 命令可以检索 */var/log/audit.log* 中的日志内容，*-m* 选项指定消息类型，*-ts* 指定时间，如：
> 
> ```bash
> [root@localhost ~]\# ausearch -m AVC -ts recent
> <no matches>
> ```

### 以标准 Apache Web 服务器为例

​在 root 用户家目录创建 html 文件并将其移动到 Apache 静态页存放目录，启动 Apache 服务后访问：

```bash
[root@localhost ~]\# touch /root/file && mv /root/file /var/www/html
[root@localhost ~]\# systemctl start httpd
[root@localhost ~]\# curl http://localhost/file
<!DOCTYPE HTML PUBLIC "-//IFTF//DTD HTML 2.0//EN">
<html><head><title>403 Forbidden</title></head>
<body><h1>Forbidden</h1>
<p>You don\`t have permission to access /file on this server.</p>
</body></html>
```

​按理说 file 文件在 Apache 的静态页目录里，一般不会出现访问被拒绝的情况，但是它却返回了 *permission denied* 的错误提示页面，可以通过查看 *1.监控 SELinux 冲突* 里的两个文件来分析问题：

```bash
[root@localhost ~]\# tail /var/log/audit/audit.log
type=AVC msg=audit(1392944135.482:429): avc: denied { getattr } for
  pid=1609 comm="httpd" path="/var/www/html/file" dev="sda1" ino=8980981
  scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0
  tclass=file
[root@localhost ~]\# tail /var/log/messages
Oct 8 10:18:20 localhost setroubleshoot: SELinux is preventing /usr/sbin/httpd from getattr access on the file. For complete SELinux messages, run `sealert -l 12345678-1234-5678-1234567890ab`.
```

​根据提示可以使用 `sealert -l 12345678-1234-5678-1234567890ab` 来查看更多信息，包括可能的解决办法：

```bash
[root@localhost ~]\# sealert -l 12345678-1234-5678-1234567890ab
SELinux is preventing /usr/sbin/httpd from getattr access on the file.

***** Plugin catchall (100. confidence) suggests *****************************************

If you believe that httpd should be allowed getattr access on the file by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access. 
Do
allow this access for now by executing:
# grep httpd /var/log/audit/audit.log | audit2allow -M mypol
# semodule -i mypol.pp

Additional Information:
Source Context                system_u:system_r:httpd_t:s0
Target Context              unconfined_u:object_r:admin_home_t:s0
Target Objects                 [ file ]
Source                         httpd
Source Path                    /usr/sbin/httpd
Port                         <Unknown>
Host                        localhost
Source RPM Packages            httpd-2.4.6-14.el7.x86_64
Target RPM Packages
Policy RPM                     selinux-policy-3.12.1-124.el7.noarch
Selinux Enabled             True
Policy Type                    targeted
Enforcing Mode                 Enforcing
......

Raw Audit Messages
type=AVC msg=audit(1392944135.482:429): avc: denied { getattr } for
  pid=1609 comm="httpd" path="/var/www/html/file" dev="sda1" ino=8980981
  scontext=system_u:system_r:httpd_t:s0 tcontext=unconfined_u:object_r:admin_home_t:s0
  tclass=file

type=SYSCALL msg=audit(1392944135.482:429): arch=x86_64 syscall=lstat success=no
  exit=EACCES a0=7f9fed0edea88 a1=7fff7bffc770 a2=7fff7bffc770 a3=0 items=0 ppid=1608 
  pid=1609 auid=4294967295 uid=48 gid=48 euid=48 suid=48 fsuid=48 egid=48 sgid=48 
  fsgid=48 tty=(none) ses=4294967295 comm=httpd exe=/usr/sbin/httpd 
  subj=system_u:system_r:httpd_t:s0 key=(null)

Hash: httpd,httpd_t,admin_home_t,file,getattr
```

​从该详细信息的 "Raw Audit Messages" 部分可以看出来目标文件 /var/www/html/file 正是问题所在：目标上下文 *tcontext* 并不属于 Web 服务器。可以使用 `restorecon /var/www/html/file` 命令修复此文件上下文。
