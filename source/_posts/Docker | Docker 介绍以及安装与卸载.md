---
title: Docker 介绍以及安装与卸载
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

Docker 是一个开源的应用容器引擎，让开发者可以打包他们的应用以及依赖包到一个可移植的容器中,然后发布到任何流行的 Linux 或 Windows 操作系统的机器上,也可以实现虚拟化,容器是完全使用沙箱机制,相互之间不会有任何接口。

Docker 官方文档地址：[https://docs.docker.com/](https://docs.docker.com/)。在其官网上关于 docker 的整体架构介绍可以找到这张图：

![Docker Architecture Diagram](images/docker/Docker-整体架构.svg)

可以看出 docker 整体是模块化、低耦合且遵循 C/S 架构的，构成它的是功能不同的组件，以下是对这些组件的功能的梳理。

## Docker 组件介绍

* Docker Daemon：Docker 守护进程（`dockerd`）监听 Docker API 请求并管理 Docker 对象，例如镜像、容器、网络和存储卷。守护进程还可以与其他守护进程通信以管理 Docker 服务。

* Docker Client：Docker 客户端（`docker`）是许多 Docker 用户与 Docker 交互的主要方式。当你使用 `docker run` 等命令时，客户端会将这些命令发送给 dockerd，dockerd 会执行这些命令。 docker 命令使用 Docker API。Docker 客户端可以与多个守护进程通信。

* Docker Desktop：是一个易于安装的应用程序，适用于 Mac 或 Windows 环境，能够构建和共享容器化应用程序和微服务。Docker Desktop 包括 Docker 守护程序 (`dockerd`)、Docker 客户端 (`docker`)、Docker Compose、Docker Content Trust、Kubernetes 和 Credential Helper。有关更多信息，请参阅 Docker 桌面。

* Docker Registries：Docker 仓库存储 *Docker 映像*（Docker Image）。DockerHub 是一个任何人都可以使用的公共仓库，并且 Docker 默认配置为在 DockerHub 上查找镜像。甚至可以定制运行自己的私有仓库。

> 当使用 `docker pull` 或 `docker run` 命令时，将从系统所配置的镜像仓库中提取所需的镜像。当使用 `docker push` 命令时，你定制的镜像会被推送到你配置的镜像仓库中。

* Docker Objects：当使用 Docker 时，会涉及到创建和使用图像、容器、网络、卷、插件和其他对象。以下简要概述其中一些对象。

  - Docker Image

  Docker 镜像是一个只读模板，其中包含创建 Docker 容器时的说明。通常，一个镜像基于另一个镜像，并附带有一些额外的自定义。例如，可以构建一个基于 *ubuntu* 镜像的镜像，但会安装 Apache Web 服务器和一些你的应用程序，该镜像还包含使你的应用程序运行所需的配置的详细信息。

  你可以创建自己的镜像，也可以只使用其他人创建并在仓库中发布的镜像。要构建你自己的镜像，你需要使用简单的语法创建一个 *Dockerfile*，用于定义创建和运行镜像所需的步骤。*Dockerfile* 中的每条指令都会在镜像中创建一个层。当你更改 *Dockerfile* 并重建映像时，仅重建那些已更改的层。与其他虚拟化技术相比，这是使镜像如此轻量、小巧和快速的部分原因。

  - Docker Container

  Docker 容器是镜像的可运行实例。你可以使用 Docker API 或 CLI 创建、启动、停止、移动或删除容器。你可以将容器连接到一个或多个网络，将存储附加到它，甚至可以根据其当前状态创建新的镜像。

  默认情况下，一个容器与其他容器以及其宿主机的隔离相对较好。你可以控制容器的网络、存储和其他底层子系统与其他容器或宿主机的隔离程度。

  一个容器由其镜像以及你在创建或启动它时提供给它的任何配置选项进行定义。当容器被移除时，任何未存储在持久存储中的状态更改都会消失。 

​Docker 三大核心组件指的是：Docker Image、Docker Container 以及 Docker Registry。

### Docker 运行示例

以下命令运行 `ubuntu` 容器，以交互方式附加到本地命令行会话，并运行 **/bin/bash**。

```bash
[skinyi@fedora ~]\$ sudo docker run -i -t ubuntu /bin/bash
```

^ 安装 Docker 后如不做其他配置则 `docker run` 命令只能使用 root 权限来执行。 

当你运行此命令时，会发生以下情况（假设你使用的是默认镜像仓库配置）：

1. 如果你在本地没有 `ubuntu` 映像，Docker 会从你配置的镜像仓库（默认 DockerHub）中提取它，就像你手动运行 `docker pull ubuntu` 一样；

2. Docker 使用该镜像创建一个新容器，就像你手动运行了 `docker container create` 命令一样；

3. Docker 为该容器分配一个读写文件系统，作为该容器的底层，这允许正在运行的容器在其本地文件系统中创建或修改文件和目录；

4. 由于没有指定任何其他的网络选项，Docker 会创建一个网络接口来将该容器连接到默认网络，包括为容器分配 IP 地址。默认情况下，容器可以使用主机的网络连接连接到外部网络。

5. Docker 启动容器并执行 `/bin/bash`。由于容器以交互方式运行并附加到您的终端（`-i` 和 `-t` 选项），所以你可以在将输出记录到终端时使用键盘提供输入。

6. 当你键入 `exit` 命令以终止 `/bin/bash` 命令时，容器会停止但不会被删除，你可以重新启动或删除它。

## Docker 的安装与卸载

个人学习使用操作系统为：Fedora 35，Fedora 版本目前仅支持 Fedora 34 和 Fedora 35。

```bash
[skinyi@fedora ~]\$ cat /etc/os-release
NAME="Fedora Linux"
VERSION="35 (Server Edition)"
ID=fedora
VERSION_ID=35
VERSION_CODENAME=""
PLATFORM_ID="platform:f35"
PRETTY_NAME="Fedora Linux 35 (Server Edition)"
ANSI_COLOR="0;38;2;60;110;180"
LOGO=fedora-logo-icon
CPE_NAME="cpe:/o:fedoraproject:fedora:35"
HOME_URL="https://fedoraproject.org/"
DOCUMENTATION_URL="https://docs.fedoraproject.org/en-US/fedora/f35/system-administrators-guide/"
SUPPORT_URL="https://ask.fedoraproject.org/"
BUG_REPORT_URL="https://bugzilla.redhat.com/"
REDHAT_BUGZILLA_PRODUCT="Fedora"
REDHAT_BUGZILLA_PRODUCT_VERSION=35
REDHAT_SUPPORT_PRODUCT="Fedora"
REDHAT_SUPPORT_PRODUCT_VERSION=35
PRIVACY_POLICY_URL="https://fedoraproject.org/wiki/Legal:PrivacyPolicy"
VARIANT="Server Edition"
VARIANT_ID=server
```

> Fedora 操作系统 Docker 安装文档：https://docs.docker.com/engine/install/fedora/。

### 卸载旧版本

```bash
[skinyi@fedora ~]\$ sudo dnf remove docker \
									docker-client \
									docker-client-lastest \
									docker-common \
									docker-lastest-logrotate \
									docker-selinux \
									docker-engine-selinux \
									docker-engine
```

### 使用软件仓库进行 Doker 的安装

​安装 **dnf-plugins-core** 插件。

```bash
[skinyi@fedora ~]$ sudo dnf -y install dnf-plugins-core
```

​添加 docker 的官方软件仓库。

```bash
[skinyi@fedora ~]$ sudo dnf config-manager --add-repo \
	https://download.docker.com/linux/fedora/docker-ce.repo
```

### 安装 Docker 引擎

```bash
[skinyi@fedora ~]$ sudo dnf install docker-ce docker-ce-cli containerd.io
```

### 启动并验证 Docker 是否成功安装

```bash
[skinyi@fedora ~]$ sudo systemctl start docker
[skinyi@fedora ~]$ sudo docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

> 实现非 root 权限执行 docker 命令或者实现一些其他安装后的优化可以参阅：https://docs.docker.com/engine/install/linux-postinstall/。

### 卸载 Docker 引擎

```bash
[skinyi@fedora ~]$ sudo dnf remove docker-ce docker-ce-cli containerd.io
[skinyi@fedora ~]$ sudo rm -rf /var/lib/docker /var/lib/containerd
```

## 启动 Docker 时的执行过程

![Docker 启动过程](images/docker/Docker-启动过程.png)

> 启动过程中的注意点：
>
> 1. Docker 优先使用的存储驱动为 Overlay2；
>
> 2. 加载容器时会设置容器的网络，并创建虚拟网络接口 docker0，该接口使用桥接模式接入网络，并在宿主机防火墙中配置一个 docker zone；
>
> 3. Docker 的守护程序 `dockerd` 初始化成功后会启动 docker 应用容器引擎，然后 docker 引擎通过监听 /run/docker.sock 套接字实现客户端与守护程序之间的通信。
