---
title: 配置和保护 SSH
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

## 识别远程用户

​通过 *w* 命令查看当前登陆的用户

```bash
[root@localhost ~]\# w
 20:39:40 up 30 min,  2 users,  load average: 0.03, 0.02, 0.04
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
student  tty2     tty2             20:10   30:20  27.28s  0.29s /usr/li
root     pts/1    172.16.1.195     20:37    0.00s  0.09s  0.02s w
```

## 使用免密登陆（公私钥对）

​SSH 可通过公钥加密的方式保持通信安全。当某一 SSH 客户端连接到 SSH 服务器时，在该客户端登陆之前，服务器会向其发送公钥副本。这颗用于设置通信渠道的安全加密，并可验证客户端的服务器。

>🟢 设置位于 ~/.ssh/config 文件或 /etc/ssh/ssh_config 中的 *StrictHostKeyChecking* 参数为 yes 可使 ssh 命令在公钥不匹配时始终中断 SSH 连接。

### 配置步骤

1. 本地主机执行 *ssh-keygen -t rsa* 命令生成公私钥对；

```bash
[root@localhost ~]\# ssh-keygen -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 	# 保存 key 的文件绝对路径
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase): 				# 口令
Enter same passphrase again: 
Your identification has been saved in /root/.ssh/id_rsa.	# 私钥文件绝对路径
Your public key has been saved in /root/.ssh/id_rsa.pub.	# 公钥文件绝对路径
The key fingerprint is:
SHA256:+wQfHxm3T7+yJSobz6QXWeh2JU3WqQgfpYuxjWzlX90 root@localhost.localdomain
The key\'s randomart image is:
+---[RSA 3072]----+
|            ..  o|
|         . ..  +.|
|         .oo= *  |
|        . O+.B +o|
|        S*o+= +.E|
|        .+ B.o.o.|
|        ..+.+o .o|
|         +*...o .|
|         o=+ .o. |
+----[SHA256]-----+
[root@localhost ~]\# ll /root/.ssh
total 8
-rw-------. 1 root root 2610 Aug 19 22:42 id_rsa		# 密钥文件
-rw-r--r--. 1 root root  580 Aug 19 22:42 id_rsa.pub	# 公钥文件
```

2. 本地主机执行 *ssh-copy-id -i ~/.ssh/id_rsa.pub user@host_name_or_ip* 命令将公钥文件发送到需要免密登陆的服务器；

```bash
[root@localhost ~]\# ssh-copy-id -i ~/.ssh/id_rsa.pub skinyi@127.0.0.1
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host '127.0.0.1 (127.0.0.1)' can\'t be established.
ECDSA key fingerprint is SHA256:CYiNIK6WgrScHzOuI/rnxIdZN68Fm0LVvFKbCzx721w.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
skinyi@127.0.0.1\'s password: 

Number of key(s) added: 1

Now try logging into the machine, with: "ssh 'skinyi@127.0.0.1'" and check to make sure that only the key(s) you wanted were added.
```

3. （与第二步二选一）在远程机器上手动导入公钥：将 id_rsa.pub 文件中的内容复制追加到远程主机的 ~/.ssh/authorized_keys 文件中去；

```bash
[root@localhost .ssh]\# cat ~/.ssh/authorized_keys
ssh-rsa AAAAB3NzaC1yc2EAAAABIwAAAQEAzIU/3G5LbJEs763JRhvfTzdWAUdfmbRBTzVUo3r0IPnGDUIF135wR7FoI6KVc6jFesgJKfkGos06GdphPgF04z5tFG5VaJS+xQZepveGza2QH6eMssBNwzpZ517FQkRs1l3pLbHweI47gByhRx1xMKWjWX5dhkI0aiNaf2GAZlevb4YNRznx6MEj39LdNEqFKHdfU0iybFDm1p0ql+jPGV7T7N/Xz4GRvnuZQGUrBK39tzAdLHDlf8lU5Muhs8zJ0pMzv7tRdJCMNZ6gW+4dK6GTaOGAR6tc4HwGgIYTOPLGbYpdyJpdNd8hvBp5b17jAs5xLZHib4PVlZJJ6gw== rsa2048-082021
```

4. 验证。

>🔴 注意：远程机器的 .ssh 目录需要 700 权限，authorized_keys 文件需要 600 权限。否则将会被拒绝连接。

### 已知主机密钥管理

​若远程主机公钥被替换，则需要在本地主机上编辑已知主机文件以确保将旧公钥条目替换为新公钥条目。

​远程主机共享的公钥会在本地 */etc/ssh/ssh_known_hosts* 或 *~/.ssh/known_hosts* 文件中记录。例如：

```bash
[root@localhost .ssh]\# cat known_hosts
127.0.0.1 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBBnrDFHMD5vaRSn1P9CN5FTpHTGEnAR9n0PN15bMdmD/dNcUQUYGsrKva+StL4y1CXGK4FurJ9gDLliRvjere+U=
```

​其中，每个公钥各占一行。每条记录的第一个字段是共享该公钥的主机名和 IP 地址的列表；第二个字段是公钥的加密算法；最后一个字段是公钥本身。

​远程主机将公钥文件存放在 */etc/ssh/* 目录下，其拓展名为 .pub。

## SSH 配置优化

​OpenSSH 服务端由一个名为 sshd 的守护进程提供。它的主配置文件为 */etc/ssh/sshd_config*。

### 禁止远程使用 root 账户登陆

​完全禁止可以将 */etc/ssh/sshd_config* 配置文件中的 *PermitRootLogin* 配置设置为 no，仅允许通过密钥登陆可以将其值设置为 without-password。

### 仅允许使用公私钥对的方式进行登陆

​此场景下可以将 */etc/ssh/sshd_config* 配置文件中的 *PasswordAuthentication* 配置为 no。

> 🟢 修改 sshd 服务的配置文件后，请使用 `systemctl reload sshd` 命令确保更改立即生效。
