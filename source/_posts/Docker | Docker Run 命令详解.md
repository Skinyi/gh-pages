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
$ docker run [选项] IMAGE[:标签|@摘要] [命令] [参数...]
```

`docker run` 命令必须指定从中生成容器的镜像。镜像开发人员可以定义与以下内容相关的镜像默认值：

* 分离或前台运行；
* 容器标识；
* 网络设置；
* 对CPU和内存的运行时约束。

通过 `docker run [选项]` 用户可以添加或覆盖开发人员设置的镜像默认值。此外，用户可以覆盖几乎所有由Docker 运行时本身设置的默认值。也正因为如此，`docker run` 比任何其他 Docker 命令都有更多选项。

> 根据你的 Docker 系统配置，你可能需要在 `docker run` 命令前面加上 `sudo` 以确保命令正常执行生效。为了避免在 docker 命令中使用 sudo，你的系统管理员可以创建一个名为 docker 的 Unix 组并向其中添加 docker 操作用户。

## 仅用户可指定的选项

仅执行 `docker run` 命令的用户可指定以下命令执行时的选项：

* 独立进程或前台执行
  - 独立进程（`-d` 选项）
  - 前台执行
* 容器标识
  - 名称（`--name` 选项）
  - PID equivalent
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
$ docker run -d -p 80:80 my_image service nginx start
```

即使这也会成功的启动容器内部的 nginx 服务，但是它不符合分离的容器范例，因为随着根进程（`service nginx start`）的退出，分离式容器也会跟着停止，从而导致 nginx 服务启动了却不能用。要启动一个进程如 nginx web 服务器应该按照下述方法：

```bash
$ docker run -d -p 80:80 my_image nginx -g 'daemon off;'
```

要使用分离的容器进行输入/输出则必须使用网络连接或共享卷，因为该容器不会监听运行 `docker run` 的命令行。

为了将分离出去的容器转至前台运行，可以使用 `docker attach` 命令。