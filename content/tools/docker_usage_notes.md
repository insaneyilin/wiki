---
title: "Docker notes"
date: 2019-11-07 00:01
---

Docker 是一个用于开发，交付和运行应用程序的开放平台。Docker 使您能够 **将应用程序与基础架构分开** ，从而可以 **快速交付软件** 。借助 Docker，您可以与管理应用程序相同的方式来管理基础架构。通过利用 Docker 的方法来快速交付，测试和部署代码，您可以大大减少编写代码和在生产环境中运行代码之间的延迟。

参考：

https://www.runoob.com/docker/docker-tutorial.html

[TOC]

---

## Docker 是什么

Docker 是一个开源的应用容器引擎，基于 Go 语言开发并遵从 Apache2.0 协议开源。

Docker 可以让开发者打包他们的应用以及依赖包到一个轻量级、可移植的 **容器** 中，然后发布到任何流行的 Linux 机器上，也可以实现 **虚拟化** 。

容器是完全使用 **沙箱机制** ，相互之间不会有任何接口（类似 iPhone 的 app）,更重要的是容器性能开销极低。

Docker 从 17.03 版本之后分为 CE（Community Edition: 社区版） 和 EE（Enterprise Edition: 企业版）。

---

## Docker 基本概念

### 镜像、容器和仓库

- 镜像（Image）：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
- 容器（Container）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- 仓库（Repository）：仓库可看作一个代码控制中心，用来保存镜像。

![](/wiki/attach/images/docker_usage_notes/docker_concepts.png)

- Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。
- Docker 容器通过 Docker 镜像来创建。
- 容器与镜像的关系类似于面向对象编程中的对象与类。

Docker 客户端(Client)：Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。

Docker 主机(Host)：一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。

Docker 仓库(Registry) ：Docker 仓库用来保存镜像，可以理解为代码控制中的代码仓库。Docker Hub(https://hub.docker.com) 提供了庞大的镜像集合供使用。一个 Docker Registry 中可以包含多个仓库（Repository）；每个仓库可以包含多个标签（Tag）；每个标签对应一个镜像。

通常，一个仓库会包含同一个软件不同版本的镜像，而标签就常用于对应该软件的各个版本。我们可以通过 `<仓库名>:<标签>` 的格式来指定具体是这个软件哪个版本的镜像。如果不给出标签，将以 `latest` 作为默认标签。

Docker Machine：Docker Machine是一个简化Docker安装的命令行工具，通过一个简单的命令行即可在相应的平台上安装Docker，比如VirtualBox、 Digital Ocean、Microsoft Azure。

---

## 使用 Docker

### 运行 Docker 容器

可以直接运行 Docker 容器完成一些特定的任务：

```bash
docker run ubuntu:15.10 /bin/echo "Hello world"
Hello world

# docker: Docker 的二进制执行文件。
# run: 与前面的 docker 组合来运行一个容器。
# ubuntu:15.10: 指定要运行的镜像，Docker 首先从本地主机上查找镜像是否存在，如果不存在，Docker 就会从镜像仓库 Docker Hub 下载公共镜像。
# /bin/echo "Hello world": 在启动的容器里执行的命令
```

交互式运行：

```bash
docker run -i -t ubuntu:15.10 /bin/bash
# -t: 在新容器内指定一个伪终端或终端。
# -i: 允许你对容器内的标准输入 (STDIN) 进行交互。
```

后台模式启动容器：

```bash
docker run -d ubuntu:14.04 /bin/sh -c "while true; do echo hello world; sleep 1; done"
faae59a0c897a1392f0e38212c106f2fa2a23b4ec23f9d75893bdd2ca9339bca
# `faae59a0c897a1392f0e38212c106f2fa2a23b4ec23f9d75893bdd2ca9339bca` 这个是 **容器 ID** 。

```

可以通过 `docker ps` 来查看容器运行情况。

- 使用 `docker logs <NAME>` 命令，查看容器内的标准输出。
- `docker logs -f <NAME>` 类似于 `tail -f` 方式查看容器的标准输出。
- `docker attach <NAME>` 可以进入相应容器。
- `docker start <NAME>` 启动相应容器。
- `docker stop <NAME>` 可以停止相应容器。
- `docker restart <NAME>` 可以重启相应容器。
- `docker rm <NAME>` 可以删除相应容器。

推荐利用 `docker exec` 方式进入容器：

```bash
sudo docker exec -it <CONTAINER ID> /bin/bash
```

查看 docker 客户端命令帮助：

```bash
docker <command> --help
```

### Docker 镜像的使用

当运行容器时，使用的镜像如果在本地中不存在，docker 就会自动从 docker 镜像仓库中下载，默认是从 Docker Hub 公共镜像源下载。

可以使用 docker images 来列出本地主机上的镜像。

同一仓库源可以有多个 TAG，代表这个仓库源的不同个版本，我们使用 `REPOSITORY:TAG` 来定义不同的镜像。

```bash
docker run -t -i ubuntu:14.04 /bin/bash
docker run -t -i ubuntu:15.10 /bin/bash
```

当我们在本地主机上使用一个不存在的镜像时 Docker 就会自动下载这个镜像。如果我们想预先下载这个镜像，我们可以使用 `docker pull` 命令来下载它。

```bash
docker pull ubuntu:13.10
```

我们可以从 Docker Hub 网站来搜索镜像，Docker Hub 网址为： https://hub.docker.com/

也可以用 `docker search` 命令来搜索镜像。搜索之后用 `docker pull` 拉取镜像


加载一个本地镜像文件：

```bash
docker load -i <docker_image_file>
```

### 创建镜像

当我们从docker镜像仓库中下载的镜像不能满足我们的需求时，我们可以通过以下两种方式对镜像进行更改。

1. 从已经创建的容器中更新镜像，并且提交这个镜像
2. 使用 `Dockerfile` 来创建一个新的镜像

#### 1. 更新已有镜像

更新镜像之前，我们需要使用镜像来创建一个容器。

```bash
docker run -t -i ubuntu:15.10 /bin/bash
```

在运行的容器内使用 `apt-get update` 命令进行更新，然后退出这个容器。

```bash
docker commit -m="has update" -a="runoob" <container id> runoob/ubuntu:v2

# -m:提交的描述信息
# -a:指定镜像作者
# <container id>：容器ID
# runoob/ubuntu:v2:指定要创建的目标镜像名

```

#### 2. 使用 Dockerfile

我们使用命令 `docker build` ， 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。

一个 Dockerfile 例子：

```
FROM    centos:6.7  # 指定使用哪个镜像源
MAINTAINER      Fisher "fisher@sudops.com"
RUN     /bin/echo 'root:123456' |chpasswd    # 在镜像内执行什么指令
RUN     useradd runoob
RUN     /bin/echo 'runoob:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22    # EXPOSE 表示暴露容器运行时的监听端口给外部
EXPOSE  80
CMD     /usr/sbin/sshd -D    # CMD 为容器运行时默认会执行的命令，一个Dockerfile仅仅最后一个CMD起作用
```

使用 `docker build` 来构建新镜像：

```bash
docker build -t runoob/centos:6.7 .
```

使用 `docker tag` 来设置镜像标签:

```bash
docker tag <image id> runoob/centos:dev
```

