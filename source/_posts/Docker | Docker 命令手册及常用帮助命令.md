---
title: Docker 命令手册及常用帮助命令
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

## 常用命令速查表格

| 命令                 | 用途                                                         |
| -------------------- | ------------------------------------------------------------ |
| docker version       | 查看 docker 客户端及服务端（docker 引擎）的版本及环境信息    |
| docker info          | 查看 docker 的系统信息，包括客户端、服务端配置、镜像与容器数量以及一些其他常见配置 |
| docker [命令] --help | 查看 `docker [命令]` 使用的帮助信息，比如 `docker image --help` |

> Docker 命令速查手册地址：https://docs.docker.com/engine/reference/run/ 。

## 常用帮助命令介绍

### `docker version` 查看 docker 版本信息

```bash
[skinyi@fedora ~]\$ sudo docker version
[sudo] skinyi 的密码：
Client: Docker Engine - Community
 Version:           20.10.12
 API version:       1.41
 Go version:        go1.16.12
 Git commit:        e91ed57
 Built:             Mon Dec 13 11:46:03 2021
 OS/Arch:           linux/amd64
 Context:           default
 Experimental:      true

Server: Docker Engine - Community
 Engine:
  Version:          20.10.12
  API version:      1.41 (minimum version 1.12)
  Go version:       go1.16.12
  Git commit:       459d0df
  Built:            Mon Dec 13 11:43:48 2021
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.4.12
  GitCommit:        7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc:
  Version:          1.0.2
  GitCommit:        v1.0.2-0-g52b36a2
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0

```

### `docker info` 查看 docker 的系统信息

```bash
[skinyi@fedora ~]\$ sudo docker info
[sudo] skinyi 的密码：
Client:
 Context:    default
 Debug Mode: false
 Plugins:
  app: Docker App (Docker Inc., v0.9.1-beta3)
  buildx: Docker Buildx (Docker Inc., v0.7.1-docker)
  scan: Docker Scan (Docker Inc., v0.12.0)

Server:
 Containers: 2
  Running: 0
  Paused: 0
  Stopped: 2
 Images: 1
 Server Version: 20.10.12
 Storage Driver: overlay2
  Backing Filesystem: xfs
  Supports d_type: true
  Native Overlay Diff: true
  userxattr: false
 Logging Driver: json-file
 Cgroup Driver: systemd
 Cgroup Version: 2
 Plugins:
  Volume: local
  Network: bridge host ipvlan macvlan null overlay
  Log: awslogs fluentd gcplogs gelf journald json-file local logentries splunk syslog
 Swarm: inactive
 Runtimes: runc io.containerd.runc.v2 io.containerd.runtime.v1.linux
 Default Runtime: runc
 Init Binary: docker-init
 containerd version: 7b11cfaabd73bb80907dd23182b9347b4245eb5d
 runc version: v1.0.2-0-g52b36a2
 init version: de40ad0
 Security Options:
  seccomp
   Profile: default
  cgroupns
 Kernel Version: 5.16.8-200.fc35.x86_64
 Operating System: Fedora Linux 35 (Server Edition)
 OSType: linux
 Architecture: x86_64
 CPUs: 2
 Total Memory: 3.788GiB
 Name: fedora
 ID: I5R6:5NIE:OTIL:XBI4:R6WP:5XSO:BZTG:QGV6:6RHG:OU7U:P3EP:JTEC
 Docker Root Dir: /var/lib/docker
 Debug Mode: false
 Registry: https://index.docker.io/v1/
 Labels:
 Experimental: false
 Insecure Registries:
  127.0.0.0/8
 Live Restore Enabled: false

```

### `docker [命令] --help` 查看 `docker [命令]` 的帮助信息

```bash
[skinyi@fedora ~]\$ docker image --help

Usage:  docker image COMMAND

Manage images

Commands:
  build       Build an image from a Dockerfile
  history     Show the history of an image
  import      Import the contents from a tarball to create a filesystem image
  inspect     Display detailed information on one or more images
  load        Load an image from a tar archive or STDIN
  ls          List images
  prune       Remove unused images
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rm          Remove one or more images
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE

Run 'docker image COMMAND --help' for more information on a command.
```

```bash
[skinyi@fedora ~]$ sudo docker container list --help

Usage:  docker container ls [OPTIONS]

List containers

Aliases:
  ls, ps, list

Options:
  -a, --all             Show all containers (default shows just running)
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print containers using a Go template
  -n, --last int        Show n last created containers (includes all states) (default -1)
  -l, --latest          Show the latest created container (includes all states)
      --no-trunc        Don't truncate output
  -q, --quiet           Only display container IDs
  -s, --size            Display total file sizes
```

## `docker` 子命令功能简介

| 子命令                                                       | 描述                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [docker attach](https://docs.docker.com/engine/reference/commandline/attach/) | 将本地标准输入输出和错误流附加到正在运行的容器 |
| [docker build](https://docs.docker.com/engine/reference/commandline/build/) | 从 Dockerfile 构建一个镜像                             |
| [docker builder](https://docs.docker.com/engine/reference/commandline/builder/) | 管理镜像构建器                                                |
| [docker checkpoint](https://docs.docker.com/engine/reference/commandline/checkpoint/) | 管理检查点                                           |
| [docker commit](https://docs.docker.com/engine/reference/commandline/commit/) | 从一个容器的所有更改创建一个新的镜像                                   |
| [docker config](https://docs.docker.com/engine/reference/commandline/config/) | 管理 Docker 的配置                                              |
| [docker container](https://docs.docker.com/engine/reference/commandline/container/) | 管理容器                                              |
| [docker context](https://docs.docker.com/engine/reference/commandline/context/) | 管理 docker 上下文                                              |
| [docker cp](https://docs.docker.com/engine/reference/commandline/cp/) | 在容器和本地存储之间拷贝文件或目录                                       |
| [docker create](https://docs.docker.com/engine/reference/commandline/create/) | 创建一个新的容器                                               |
| [docker diff](https://docs.docker.com/engine/reference/commandline/diff/) | 监视容器上的文件或目录的变化                                         |
| [docker events](https://docs.docker.com/engine/reference/commandline/events/) | 从 docker 守护服务上获取实时事件                                 |
| [docker exec](https://docs.docker.com/engine/reference/commandline/exec/) | 在一个运行的容器上运行命令                                           |
| [docker export](https://docs.docker.com/engine/reference/commandline/export/) | 将一个容器的文件系统以 tar 包的形式导出                           |
| [docker history](https://docs.docker.com/engine/reference/commandline/history/) | 展示一个镜像的提交历史                                  |
| [docker image](https://docs.docker.com/engine/reference/commandline/image/) | 管理 docker 镜像                                                |
| [docker images](https://docs.docker.com/engine/reference/commandline/images/) | 列出镜像                                                      |
| [docker import](https://docs.docker.com/engine/reference/commandline/import/) | 从 tar 包中导入内容来创建一个文件系统镜像                         |
| [docker info](https://docs.docker.com/engine/reference/commandline/info/) | 显示系统范围的信息                                                |
| [docker inspect](https://docs.docker.com/engine/reference/commandline/inspect/) | 返回 docker 对象的底层信息                            |
| [docker kill](https://docs.docker.com/engine/reference/commandline/kill/) | 杀掉一个或多个运行中的容器                                         |
| [docker load](https://docs.docker.com/engine/reference/commandline/load/) | 从一个 tar 包或标准输入来加载镜像                                  |
| [docker login](https://docs.docker.com/engine/reference/commandline/login/) | 登录 Docker registry                                        |
| [docker logout](https://docs.docker.com/engine/reference/commandline/logout/) | 登出 Docker registry                                    |
| [docker logs](https://docs.docker.com/engine/reference/commandline/logs/) | 获取一个容器的日志                                            |
| [docker manifest](https://docs.docker.com/engine/reference/commandline/manifest/) | 管理 Docker 镜像清单和清单列表                   |
