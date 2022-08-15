---
title: 管理本地用户和组（二）
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

## 管理本地用户账户

### 使用 `useradd` 命令创建新用户

​添加用户所使用的默认配置保存在 */etc/default/useradd* 中，同时还有一些默认配置从 */etc/login.defs* 文件中读取。这两个文件只会影响新创建的用户，更改这两个文件不会影响现有用户。没有设置密码的用户是无法登陆系统的。

```bash
useradd [OPTION]... LOGIN_NAME    # 新建登陆用户
useradd -D [-befgs]    # 查看或修改创建用户使用的默认设置，
# 常见选项
-b, --base-dir BASE_DIR # 指定创建用户家目录的基目录（一般为 /home ）
-u, --uid UID # 为新账户指定 UID
-c, --comment COMMENT # 指定新用户描述信息或真实姓名
-d, --home-dir HOME_DIR # 指定新用户的家目录，默认 $BASE_DIR/LOGIN_NAME
-D, --difaults # 打印或改变默认的 useradd 配置
-g, --gid GROUP # 指定新账户的主要组的 ID 或名字
-G, --groups GROUPS # 指定新账户的补充组
-m, --create-home # 创建用户的家目录（指定该字段或 CREATE_HOME 配置项为 yes 才会创建）
-M, --no-create-home # 不创建用户的家目录，即使 /etc/login.defs 中的 CREATE_HOME 字段的值为 yes
-U, --user-group # 为用户创建与用户同名的用户组
-N, --no-user-group # 不创建与用户同名的用户组
-p, --password PASSWORD # 为该新账户设置加密的密码（不推荐）
-r, --system # 创建系统账户
-s, --shell SHELL # 为新账户指定登陆 shell
```

```bash
[root@localhost ~]\# useradd -D    # 添加用户所使用的默认配置
GROUP=100
HOME=/home
INACTIVE=-1
EXPIRE=
SHELL=/bin/bash
SKEL=/etc/skel
CREATE_MAIL_SPOOL=yes
[root@localhost ~]\# getent group 100 # 查询 GID 100 对应的 group 信息
users:x:100:    # 添加的新用户会默认将其主要组设置为 users 组
```

​以上示例中使用了 `getent` 命令，在本章结尾对这个命令会有更详细的介绍。

### 使用 `usermod` 命令修改现有用户

```bash
usermod [OPTION]... LOGIN_NAME # 修改用户账户
# 以下 useradd 中的选项也适用于 usermod，只不过行为变成了更改而不是创建，将不再赘述：
-u, -d, -g, -G, -c, -s, -p
# 其余常见选项：
-a, --append # 与 -G 搭配，追加加入补充组模式（-G 默认是替换）
-l, --login NEW_LOGIN # 修改用户名
-L, --lock # 锁定账户
-U, --unlock # 解锁账户
-m, --move-home # 与 -d 搭配，立即移动到新的家目录
```

### 使用 `userdel` 命令删除现有用户

```bash
userdel [OPTION]... LOGIN_NAME # 删除用户账户（默认不删除此用户的文件，创建的新用户如果使用了之前被删的用户的 uid 则新用户会默认获得旧用户文件的所属权）
# 常见选项：
-f, --force # 强制删除普通删除会失败的账户（如：删除有人登陆的账户）
-r, --remove # 同时删除该用户家目录和邮箱目录（推荐）
```

### 使用 `passwd` 命令设置用户初始密码或更改现有的密码

```bash
passwd [-k] [-l] [-u] [-e] [-d] [-S] [--stdin] [username]
# 常见选项：
-k, --keep-tokens # 继续使用旧密码，重新计算过期时间
-l, --lock # 仅 root 可用，锁定指定账户的密码（但账户没有被锁，其他方式仍能登陆）
-u, --unlock # 仅 root 可用，解锁指定账户的密码
-d, --delete # 仅 root 可用，删除某个账户的密码
-e, --expire # 仅 root 可用，强制某个用户下次登陆时修改密码
-S, --status # 输出账户的账户密码状态信息
--stdin, # 从标准输入中读取密码（常与管道配合）
# 不使用任何选项时可省略 username, 默认修改当前登陆账户的用户密码
```

## 管理本地组账户

### 使用 `groupadd` 命令添加用户组

```bash
groupadd [OPTIONS] GROUP # 创建新用户组
# 常见选项：
-g, --gid GID # 指定 GID
-K, --key KEY=VALUE # 覆盖 /etc/login.defs 中的默认值，可以指定多个 -K 选项
-r, --system # 创建系统组
```

### 使用 `groupmod` 命令修改用户组

```bash
groupmod [OPTIONS] GROUP # 修改系统中的用户组的定义
# 常见选项：
-g, --gid GID # 改变组 ID
-n, --new-name NEW_GROUP # 修改组名
```

### 使用 `groupdel` 命令删除用户组

```bash
groupdel [OPTIONS] GROUP # 删除用户组
```

## 管理用户密码

​用户密码存放文件 /etc/shadow 的格式：

​        `用户名`:`加密后的密码`[^1]:`上次更改密码的日期`:`从上次更改密码到下一次可以更改所必须经过的最短天数`:`在密码过期之前不进行密码更改可以经过的最长天数`[^2]:`警告期`[^3]:`非活动期`[^4]:`密码过期日`[^5]:`预留字段`

[^1]: 加密密码字符串的不同部分由美元符号 \$ 隔开，第一个 \$ 后的数字代表此密码所用的哈希算法，数字 6 代表使用的是 SHA-512 哈希算法（红帽 8 默认算法），1 代表 MD5 哈希算法，5 代表 SHA-256 哈希算法。第二个 \$ 后面的是加密密码使用的盐值。第三个 \$ 后面的是用户密码的加密哈希值，前面盐值和用户未加密密码进行组合加密生成的密码哈希串。
[^2]: 空字段表示它不会根据上次更改以来的时间失效。
[^3]: 当用户在截止日前之前登陆达到该天数时，会收到有关密码过期的警告。
[^4]: 一旦密码过期，这些天内仍可以接受登陆，过了这一时期之后，账户将被锁定。
[^5]: 其设置值为从 1970 年 1 月 1 日起的天数，并按 UTC 时区计算。空字段表示它不会在特定的日期失效。

​当用户尝试登陆时，系统会从 /etc/shadow 中查询用户的密码条目，并将用户的盐值和键入的未加密密码组合，再使用指定的哈希算法加密。如果结果与已加密哈希匹配，则用户输入了正确的密码，即会登陆成功，否则表示用户输入了错误的密码，登陆尝试也会失败。

​下图展示了相关密码期限参数及其周期作用。

![chage 命令密码周期](images/linux/chage-命令密码周期.png)

#### 使用 `chage` 命令修改用户密码过期设置

```bash
chage [OPTIONS] LOGIN_NAME # 修改用户密码过期信息
# 常见选项：
-d, --lastday LAST_DAY # 修改最后一次更改密码的时间，可以设置为从 1970 年 1 月 1 日起度过的天数，也可以以 YYYY-MM-DD 的形式进行设置，如果设置为 0 的话，则该用户在下次登陆时会被强制要求更改密码
-E, --expiredate EXPIRE_DATE # 修改密码强制过期时间，过期将无法登陆，只能由 root 解锁后才能继续使用，可以设置为 1970 年 1 月 1 日起度过的天数，也可以以 YYYY-MM-DD 的形式进行设置，设置为 -1 密码将永不过期
-I, --inactive INACTIVE # 设置宽限期（天数，即密码已过期到用户账户被锁这段时间）,设置为 -1 则没有宽限期
-l, --list # 显示账户密码的重要时间段
-m, --mindays MIN_DAYS # 设置两次更改密码的最小间隔（天数），设置为 0 代表没有限制
-M, --maxdays MAX_DAYS # 设置密码可用的最大天数，超过时间即会过期，设置为 -1 会移除密码过期限制
-W, --warndays WARN_DAYS # 设置密码到期前发出改密警告的天数 
```

​如果没有指定选项，则 `chage` 会进入交互式操作模式。

## GETENT 命令介绍

​使用 `getent` 命令来查看系统的数据库中的相关记录。

### 语法

```bash
getent database [key ...]    # Get entries from administrative database.
-s, --service=CONFIG    # 要使用的服务配置
-?, --help                # 给出该系统求助列表
    --usage                # 给出简要的用法信息
-V, --version            # 打印程序版本号
```

### 支持的系统数据库

​包含：ahosts、ahostsv4、ahostsv6、alias、ethers、group、gshadow、hosts、netgroup、networks、passwd、protocols、rpc、services、shadow。

### 示例

```bash
# 1. 查看前三项系统注册的服务及其占用的端口和协议
[root@vmrhelskinyi ~]\# getent services | head -3
tcpmux                1/tcp
tcpmux                1/udp
rje                   5/tcp
# 2. 根据当前登陆信息查找 UID
[root@vmrhelskinyi ~]\# getent passwd `whoami`
root:x:0:0:root:/root:/bin/bash
# 3. 进行 DNS 反查
[root@vmrhelskinyi ~]\# getent hosts kernel.org
198.145.29.83   kernel.org
```
