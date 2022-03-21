---
title: Docker 虚拟化技术的本质基底层技术
lang: zh-CN
tags: 
  - Docker
  - Linux
  - 容器
  - 虚拟化
categories: 
  - [学习, Docker]
  - [日常记录]
---

## 容器的本质

容器本身是一种进程隔离技术。容器为进程提供了一个隔离的环境，容器内的进程无法访问容器外的进程。以运行一个 `ubuntu` 容器为例：

```bash
[root@localhost ~]\# docker run -it ubuntu
root@d304e6f37918:\# top                        # 执行 top 命令
```

在主机上查看系统进程：

```bash
[root@localhost ~]\# ps -ef
# ......
root        3630    3629  0 13:48 pts/0    00:00:00 docker run -it ubuntu
root        3648       1  0 13:48 ?        00:00:00 /usr/bin/containerd-shim-runc-v2 -namespace moby -id d304e6f3791808ca34c8f8361da22
root        3674    3648  0 13:48 pts/0    00:00:00 bash
root        3775    3674  0 13:49 pts/0    00:00:00 top
# ......
```

其中第一条记录是启动该容器时的命令，其父进程是执行该命令的 `bash` 进程；第二条记录是该容器的进程，其父进程是 `systemd` 进程，由 `systemd` 进程直接执行 `containerd-shim-runc-v2` 命令启动容器的进程；第三条记录表明该容器的进程自动启动一个 `bash` 进程用以接受用户的指令；第四条记录表明在该 `bash` 进程中执行了 `top` 命令。

### 进程隔离

以上示例印证了容器的本质是一个进程，容器中的进程是容器进程的子进程或子子进程。但如何表明该容器进程是与其他进程隔离的呢？在上述 Ubuntu 容器中执行 `ps -ef` 命令：

```bash
root@baf49ec287bb:\# ps -ef
UID          PID    PPID  C STIME TTY          TIME CMD
root           1       0  0 06:15 pts/0    00:00:00 bash
root           9       1  0 06:15 pts/0    00:00:00 ps -ef
```

可以看到该容器的 `bash` 进程的 PID 竟然是 1 而不是和主机一样是 `systemd` 进程，而在主机上其进程号是 3674，由此可以视作该容器进程里的子进程认为自己运行在一个“与世隔绝”的操作系统世界里。

### 文件系统隔离

在该容器内部执行：

```bash
root@baf49ec287bb:\# ls -l /dev
total 0
crw--w----. 1 root tty  136, 0 Mar 21 06:42 console
lrwxrwxrwx. 1 root root     11 Mar 21 06:15 core -> /proc/kcore
lrwxrwxrwx. 1 root root     13 Mar 21 06:15 fd -> /proc/self/fd
crw-rw-rw-. 1 root root   1, 7 Mar 21 06:15 full
drwxrwxrwt. 2 root root     40 Mar 21 06:15 mqueue
crw-rw-rw-. 1 root root   1, 3 Mar 21 06:15 null
lrwxrwxrwx. 1 root root      8 Mar 21 06:15 ptmx -> pts/ptmx
drwxr-xr-x. 2 root root      0 Mar 21 06:15 pts
crw-rw-rw-. 1 root root   1, 8 Mar 21 06:15 random
drwxrwxrwt. 2 root root     40 Mar 21 06:15 shm
lrwxrwxrwx. 1 root root     15 Mar 21 06:15 stderr -> /proc/self/fd/2
lrwxrwxrwx. 1 root root     15 Mar 21 06:15 stdin -> /proc/self/fd/0
lrwxrwxrwx. 1 root root     15 Mar 21 06:15 stdout -> /proc/self/fd/1
crw-rw-rw-. 1 root root   5, 0 Mar 21 06:15 tty
crw-rw-rw-. 1 root root   1, 9 Mar 21 06:15 urandom
crw-rw-rw-. 1 root root   1, 5 Mar 21 06:15 zero
```

可以发现该容器的设备文件中没有主机中的硬盘 block 文件，这是由于该容器的文件系统也是和主机不一样的独立文件系统。那么容器的文件系统在哪儿呢？使用 `docker inspect 容器名|容器ID` 命令并查看输出的 *GraphDriver* 字段来找到容器的文件系统文件在主机中的位置。

```bash
[skinyi@localhost ~]\$ sudo docker inspect -f '{{json .GraphDriver}}' ubuntu | jq
{
  "Data": {
    "MergedDir": "/var/lib/docker/overlay2/7472d22959e652e97acb49e6de5dd704bd2fb46de0f35d12a7b0cfbba1655cb5/merged",
    "UpperDir": "/var/lib/docker/overlay2/7472d22959e652e97acb49e6de5dd704bd2fb46de0f35d12a7b0cfbba1655cb5/diff",
    "WorkDir": "/var/lib/docker/overlay2/7472d22959e652e97acb49e6de5dd704bd2fb46de0f35d12a7b0cfbba1655cb5/work"
  },
  "Name": "overlay2"
}
```

该容器的文件系统位置就在主机上 *UpperDir* 属性所指向的位置。主机另起终端在该目录下找到容器的 dev 目录，并 `ls -l` 会发现该目录是空的，其实这是正常的，因为容器中该目录是内存数据中的一部分而未持久化到主机硬盘中。

文件隔离的本质技术是使用 `chroot` 系统调用，该系统调用用于将一个进程及其子进程的根目录改变到文件系统中的一个新位置，并让这些进程只能访问到该目录，此功能的初衷是为每个进程提供独立的磁盘空间。

我们可以模拟创建容器所进行的操作：

```bash
[root@localhost ~]\# cd /var/lib/docker/overlay2/7472d22959e652e97acb49e6de5dd704bd2fb46de0f35d12a7b0cfbba1655cb5/
[root@localhost 7472d22959e652e97acb49e6de5dd704bd2fb46de0f35d12a7b0cfbba1655cb5]\# chroot diff
root@localhost:\# ls    
bin  boot  dev  etc  home  lib  lib32  lib64  libx32  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
root@localhost:\# pwd
/
```

此时无法执行 `ps -ef` 命令，因为其内存文件系统还没有挂载。可以到此为止了。

## Linux 命名空间技术

后面的容器技术的实现基本上都离不开 Linux 的 Namespace 技术，该技术是 Linux 提供的内核级环境隔离的方法，它提供了以下的系统隔离能力：

* Mount Namespace       提供磁盘挂载点和文件系统的隔离能力
* IPC Namespace         提供进程间通信隔离的能力
* Network Namespace     提供网络隔离能力
* UTS Namespace         提供主机名隔离能力
* PID Namespace         提供进程隔离能力
* User Namespace        提供用户隔离能力

在主机上查看容器的 `bash` 进程、主机的 `bash` 进程以及主机 `systemd` 进程的命名空间：

```bash
[root@localhost ~]\# ls -l /proc/3674/ns
总用量 0
lrwxrwxrwx. 1 root root 0  3月 21 15:42 cgroup -> 'cgroup:[4026532847]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 ipc -> 'ipc:[4026532777]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 mnt -> 'mnt:[4026532775]'
lrwxrwxrwx. 1 root root 0  3月 21 15:41 net -> 'net:[4026532780]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 pid -> 'pid:[4026532778]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 pid_for_children -> 'pid:[4026532778]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 uts -> 'uts:[4026532776]'
[root@localhost ~]\# ls -l /proc/self/ns
总用量 0
lrwxrwxrwx. 1 root root 0  3月 21 15:42 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 net -> 'net:[4026532000]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 root root 0  3月 21 15:42 uts -> 'uts:[4026531838]'
[root@localhost ~]\# ls -l /proc/1/ns
总用量 0
lrwxrwxrwx. 1 root root 0  3月 21 13:05 cgroup -> 'cgroup:[4026531835]'
lrwxrwxrwx. 1 root root 0  3月 21 13:42 ipc -> 'ipc:[4026531839]'
lrwxrwxrwx. 1 root root 0  3月 21 13:42 mnt -> 'mnt:[4026531840]'
lrwxrwxrwx. 1 root root 0  3月 21 13:42 net -> 'net:[4026532000]'
lrwxrwxrwx. 1 root root 0  3月 21 13:42 pid -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0  3月 21 15:50 pid_for_children -> 'pid:[4026531836]'
lrwxrwxrwx. 1 root root 0  3月 21 13:42 time -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0  3月 21 15:50 time_for_children -> 'time:[4026531834]'
lrwxrwxrwx. 1 root root 0  3月 21 13:42 user -> 'user:[4026531837]'
lrwxrwxrwx. 1 root root 0  3月 21 13:42 uts -> 'uts:[4026531838]'
```

可以发现主机的 `bash` 进程和 `systemd` 进程完全一致，但容器中的进程和主机进程中的存在部分差别。

## CGroup 控制组

CGroup 是 Linux 内核提供的一种可以限制、记录、隔离进程组所使用的计算资源（CPU、内存、I/O 等）的机制。CGroup 提供了以下功能：

* 限制进程组可以使用的资源数量
* 进程组的优先级控制
* 记录进程组使用的资源数量
* 进程组隔离
* 进程组控制

容器技术使用 CGroup 技术限制容器对主机资源的使用。

## Containerd 容器运行时

Containerd 是一个强大的工业级容器运行时环境，其脱胎于 docker 的 libcontainerd 后经开源独立以及不断完善从 RunC 发展到现在的 Containerd（谷歌及其他大厂的一系列操作下）。现在谷歌的 K8S 默认集成的容器运行时环境已经是 Containerd，而不是原本的 Docker，Docker 现在更多代表的其实是操纵 Containerd 的一个客户端。

[!Containerd 的云原生架构](images/Containerd-整体架构.bmp)
