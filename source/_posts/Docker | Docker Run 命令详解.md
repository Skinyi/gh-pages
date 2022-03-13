---
title: Docker Run 命令详解
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
Docker 在隔离的容器中运行进程。容器是在主机上运行的进程。主机可以是本地或远程的。当执行 `docker run` 时，运行的容器进程是独立的，因为它有自己的文件系统、自己的网络和自己的独立于主机的进程树。

以下介绍了如何在运行时使用 `docker run` 命令定义容器的资源。

## `docker run` 命令的一般形式

基本的 `docker run` 命令采用以下形式：

```bash
\$ docker run [选项] IMAGE[:标签|@摘要] [命令] [参数...]
```

`docker run` 命令必须指定从中生成容器的镜像。镜像开发人员可以定义与以下内容相关的镜像默认值：

* 分离或前台运行；
* 容器标识；
* 网络设置；
* 对CPU和内存的运行时约束。

通过 `docker run [选项]` 用户可以添加或覆盖开发人员设置的镜像默认值。此外，用户可以覆盖几乎所有由 Docker 运行时本身设置的默认值。也正因为如此，`docker run` 比任何其他 Docker 命令都有更多选项。

> 根据你的 Docker 系统配置，你可能需要在 `docker run` 命令前面加上 `sudo` 以确保命令正常执行生效。为了避免在 docker 命令中使用 sudo，你的系统管理员可以创建一个名为 docker 的 Unix 组并向其中添加 docker 操作用户。

## 仅用户可指定的选项

仅执行 `docker run` 命令的用户可指定以下命令执行时的选项：

* 独立进程或前台执行
  * 独立进程（`-d` 选项）
  * 前台执行
* 容器标识
  * 名称（`--name` 选项）
  * PID 值
* IPC 设置（`--ipc`）
* 网络设置
* 重启策略（`--restart`）
* 清理（`--rm`）
* 运行时资源约束
* 运行时特权及 Linux capabilities

### 独立进程或前台执行

当启动了一个 Docker 容器，你必须先决定你是否想在后台以“分离”的模式或者默认在前台运行容器，可以使用 `-d[=<true|false>]` 选项来指定分离式模式，容器会在后台运行并打印出该容器的 id。

根据设计，当用于运行容器的根进程退出时，以分离模式启动的容器也会退出，除非你还指定了 `-rm` 选项。如果将 `-d` 与 `-rm` 一起使用，当容器退出或守护程序退出时(无论哪个先退出)，容器将被删除。

不要向分离模式运行的容器传递 `service xxx start` 的命令。如以下命令尝试创建容器后启动 nginx 服务。

```bash
\$ docker run -d -p 80:80 my_image service nginx start
```

即使这也会成功的启动容器内部的 nginx 服务，但是它不符合分离的容器范例，因为随着根进程（`service nginx start`）的退出，分离式容器也会跟着停止，从而导致 nginx 服务启动了却不能用。要启动一个进程如 nginx web 服务器应该按照下述方法：

```bash
\$ docker run -d -p 80:80 my_image nginx -g 'daemon off;'
```

要使用分离的容器进行输入/输出则必须使用网络连接或共享卷，因为该容器不会监听运行 `docker run` 的命令行。

为了将分离出去的容器转至前台运行，可以使用 `docker attach` 命令。

### 前台运行

在前台模式下（默认模式当未指定 `-d` 选项），`docker run` 可以启动容器中的进程并且将当前终端附加到该进程的标准输入、输出以及标准错误流中。它甚至可以“假装”成为一个 TTY 会话（如大多数命令行可执行文件期望的那样）并且传递信号。以下这些选项都是可配置的：

> -a=[]           ：附加到标准输入、标准输出和 / 或标准错误流
> -t              ：分配一个伪 TTY
> --sig-proxy=true：将所有收到的信号代理到进程(仅限非TTY模式)
> -i              ：即使没有连接，也保持标准输入流打开

如果你没有指定 `-a` 选项，Docker 就会将进程的标准输入、错误流附加到当前终端中，你也可以通过 `-a` 选项指定哪个流进行附加，比如：

```bash
\$ docker run -a stdin -a stdout -i -t ubuntu /bin/bash
```

对于交互式进程（如命令行 shell），必须一起指定 `-i` 和 `-t` 选项来为容器进程分配一个 TTY，这两个选项常被简写为 `-it`。当客户端从管道中接收标准输入流时指定 `-t` 选项通常是被禁止的，比如：

```bash
\$ echo test | docker run -i busybox cat
```

> 注意：在容器中运行的 PID 为 1 的进程会被 Linux 特殊看待：它会忽略任何信号的默认行为。最终就会导致收到 *SIGINT* 和 *SIGTERM* 信号的进程不会终止除非它被编码进行指定。

## 容器标识

### 容器名称（--name）

用户可以通过三种方式来标识容器：

| 标识符类型 | 例子                                                 |
|------------|----------------------------------------------------|
| UUID 长标识符 | “f78375b1c487e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778” |
| UUID 短标识符 | “f78375b1c487” |
| 名称 | “evil_ptolemy” |

其中，UUID 标识符由 Docker 守护程序生成。如果未通过 `--name` 选项来给容器赋予名称，则守护程序会生成一串随机字符来自动命名该容器。给容器进行有意义的命名是一个比较好的习惯。如果你制定了容器的名称，就可以在 Docker 网络中引用容器时使用它，不论是前台还是后台运行的容器。

> 默认网桥网络上的容器必须被链接以通过名称进行通信。

最后，为了便于自动化，你可以让 Docker 将容器的 ID 写入到你选择的文件中。这类似于一些程序会将自身进程 ID 写入到文件中去，正如你见过的 PID 文件。

```bash
--cidfile=""：将容器 ID 写入文件
```

### 镜像[\:标签]

虽然严格来说这不是一种标识容器的方法，但是你可以通过将 image[\:tag] 添加到命令中来指定您想要运行容器的镜像版本。比如：`docker run ubuntu:14.04`。

### 镜像[\@摘要]

使用 v2 或更高版本镜像格式的镜像有一个称为摘要的内容可寻址标识符。只要用于生成镜像的输入不变，摘要值就是可预测和可参考的。

以下示例使用在运行 alpine 镜像时使用了 sha256:9cacb71397b640ECA97488cf08582AE4068513101088e9f96c9814bfda95e0 摘要：

```bash
\$ docker run alpine@sha256:9cacb71397b640eca97488cf08582ae4e4068513101088e9f96c9814bfda95e0 date
```

### PID 设置（--pid）

> --pid="" ：为容器设置 PID（进程） 命名空间模式，
>             'container:\<name\|id\>'：加入其他容器的 PID 命名空间，
>             'host'：使用容器中的主机的 PID 命名空间。

默认情况下，所有容器都启用了 PID 命名空间。

PID 命名空间提供了进程的分离。PID 命名空间删除了系统进程的视图，并允许重用进程 ID，包括 pid 1。

在某些情况下，你希望你的容器共享主机的进程名称空间，基本上允许容器内的进程看到系统上的所有进程。例如，您可以使用像 `strace` 或 `gdb` 这样的调试工具构建一个容器，但是在调试容器中的进程时，你希望使用这些工具。

#### 例子：在容器中执行 \`htop\`

创建 Dockerfile:

```dockerfile
FROM alpine:latest
RUN apk add --update htop && rm -rf /var/cache/apk/*
CMD ["htop"]
```

构建此 Dockerfile  并将镜像标记为 myhtop：

```bash
\$ docker build -t myhtop .
```

使用以下命令在容器中运行 `htop` 命令：

```bash
\$ docker run -it --rm --pid=host myhtop
```

加入其他可以被用来调试该容器的 pid 命名空间。

#### 例子

开启一个运行 redis 服务的容器：

```bash
\$ docker run --name myredis -d redis
```

通过运行含有 `strace` 的其他容器来调试 redis 容器：

```bash
\$ docker run -it --pid=container:my-redis my_strace_docker_image bash
\$ strace -p 1
```
