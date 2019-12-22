---
title: Sevice-Computing | 初识Docker
date: 2019-12-14 19:56:58
categories: 
	- 服务计算
tags: 
	- Docker	
---

> 参考教程：[每天5分钟玩转 Docker 容器技术](https://www.jianshu.com/p/657687b801f0)

{% asset_img bg2018020901.png %}

<!--more-->

### 容器简介

#### 什么是容器

容器是一种轻量级、可移植、自包含的软件打包技术，使应用程序可以在几乎任何地方以相同的方式运行。开发人员在自己笔记本上创建并测试好的容器，无需任何修改就能够在生产系统的虚拟机、物理服务器或公有云主机上运行。

容器由两部分组成：

+ 应用程序本身
+ 依赖：比如应用程序需要的库或其他软件

容器和虚拟机的最大区别是虚拟机需要模拟整个操作系统，而容器运行在Host操作系统的用户空间，与操作系统的其他进程隔离。因此，所有的容器共享同一个 Host OS，这使得容器在体积上要比虚拟机小很多。另外，启动容器不需要启动整个操作系统，所以容器部署和启动速度更快，开销更小，也更容易迁移。

#### 为什么使用容器

使用容器**使软件具备了超强的可移植能力**。由于现如今的软件应用往往依赖多种服务，这些服务有自己的依赖，同时应用可能需要被部署或迁移到不同的环境，那么就需要一种通用的解决办法使得服务在不同的环境下都能顺利的运行。类比于运输行业的集装箱，集装箱解决了不同货物、不同运输环节下的不一致问题。Docker 将集装箱思想运用到软件打包上，为代码提供了一个基于容器的标准化运输系统。

#### 优点

对于开发人员 - Build Once, Run Anywhere

容器意味着环境隔离和可重复性。开发人员只需为应用创建一次运行环境，然后打包成容器便可在其他机器上运行。另外，容器环境与所在的 Host 环境是隔离的，就像虚拟机一样，但更快更简单。

对于运维人员 - Configure Once, Run Anything

只需要配置好标准的 runtime 环境，服务器就可以运行任何容器。这使得运维人员的工作变得更高效，一致和可重复。容器消除了开发、测试、生产环境的不一致性。

### 运行第一个容器

环境选择：

+ 管理工具：Docker Engine

+ runtime：runc，Docker 的默认 runtime

+ 操作系统：Ubuntu 18.04.3 LTS

#### 配置 Docker 的 apt 源

安装包，允许 `apt` 命令 HTTPS 访问 Docker 源。

```/
$ sudo apt-get install \
  apt-transport-https \
  ca-certificates \
  curl \
  software-properties-common
```

添加 Docker 官方的 GPG

```/
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

将 Docker 的源添加到 /etc/apt/sources.list

```/
$ sudo add-apt-repository \
 "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
 $(lsb_release -cs) \
 stable"
```

#### 安装 Docker

```/
$ sudo apt-get update
$ sudo apt-get install docker-ce
```

#### 运行第一个容器

```/
docker run -d -p 80:80 httpd
```

**注意该命令需要`root`权限，否则会提示`permission denied`。**执行结果如下：

{% asset_img image-20191214172104352.png %}

其过程可以简单的描述为：

+ 从 Docker Hub 下载 httpd 镜像。镜像中已经安装好了 Apache HTTP Server。

+ 启动 httpd 容器，并将容器的 80 端口映射到 host 的 80 端口。

打开浏览器输入Ubuntu的IP地址，出现如下界面，表明可以访问容器的 http 服务，容器运行成功！

{% asset_img image-20191214172349022.png %}

通过命令`docker ps`可以查看到当前启动的容器，使用`docker stop 容器ID或容器名`可以停止容器，使用命令`docker images`可以查看下载到本地的镜像。

{% asset_img image-20191214183017650.png %}

### 镜像学习

镜像是 Docker 容器的基石，容器是镜像的运行实例，有了镜像才能启动容器。

#### hello-world镜像

hello-world镜像是Docker 官方提供的一个镜像，我们执行如下操作：

{% asset_img image-20191214183934313.png %}

从上述运行的输出结果我们可以了解到镜像的运行原来，里面涉及到了容器客户端与容器服务器的交互，以及容器和镜像之间的关系。

#### 镜像的分层结构

有一种称为base的镜像，该镜像通常都是各种 Linux 发行版的 Docker 镜像，比如 Ubuntu, Debian, CentOS 等。base 镜像有两层含义：

+ 不依赖其他镜像，从 scratch 构建。

+ 其他镜像可以之为基础进行扩展。

Docker 支持通过扩展现有镜像，创建新的镜像。新镜像是从 base 镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层。这种做法的最大一个好处就是：**共享资源**。例如：有多个镜像都从相同的 base 镜像构建而来，那么 Docker Host 只需在磁盘上保存一份 base 镜像；同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。

但是，虽然低层的镜像被共享了，不同容器对其进行修改却不会造成混乱，所有的修改会被限制在单个容器内。当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称作“容器层”，“容器层”之下的都叫“镜像层”。所有对容器的改动 - 无论添加、删除、还是修改文件都只会发生在容器层中。

+ 添加文件：在容器中创建文件时，新文件被添加到容器层中。
+ 读取文件：在容器中读取某个文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，打开并读入内存。
+ 修改文件：在容器中修改已存在的文件时，Docker 会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后修改之。
+ 删除文件：在容器中删除文件时，Docker 也是从上往下依次在镜像层中查找此文件。找到后，会在容器层中记录下此删除操作。

只有当需要修改时才复制一份数据，这种特性被称作 Copy-on-Write。可见，容器层保存的是镜像变化的部分，不会对镜像本身进行任何修改。

### Dockerfile

构建镜像有两种方式，使用docker commit 命令或通过Dockerfile构建文件，一般使用Dockerfile的方式。

Dockerfile 是一个文本文件，记录了镜像构建的所有步骤。以下是一个Dockerfile的实例，该镜像在 ubuntu base 镜像中安装 vim 并保存为新镜像。

{% asset_img image-20191214191221861.png %}

我们使用`docker build`命令通过这个Dockerfile构建新镜像：

{% asset_img image-20191214192058797.png %}

…此处省略`apt-get install -y vim`的过程。

{% asset_img image-20191214193634106.png %}

…此处省略`apt-get update`的过程。

{% asset_img image-20191214193655161.png %}

上图的结果对应以下流程：

+ 运行`docker build`命令，`-t`将新镜像命名为`ubuntu-with-vim`，命令末尾的`.`指明`build context`为当前目录。Docker 默认会从`build context`中查找 Dockerfile 文件，我们也可以通过`-f`参数指定 Dockerfile 的位置。

+ 然后是镜像真正的构建过程。 首先 Docker 将`build context`中的所有文件发送给`Docker daemon`。`build context` 为镜像构建提供所需要的文件或目录。Dockerfile 中的 ADD、COPY 等命令可以将`build context`中的文件添加到镜像。上图中，`build context` 为当前目录 `/home/daniel/learn_docker`，该目录下的所有文件和子目录都会被发送给`Docker daemon`。所以，使用`build context`就要小心了，不要将多余文件放到`build context`，特别不要把 `/`、`/usr` 作为`build context`，否则构建过程会相当缓慢甚至失败。
+ 新镜像构建：
  + Step 1：执行 FROM，将 ubuntu 作为 base 镜像。ubuntu 镜像 ID 为 775349758637。
  + Step 2：执行 RUN，安装 vim：
    + 启动 ID 为 b59a91983696 的临时容器，在容器中通过 apt-get 安装 vim；
    + 安装成功后，将容器保存为镜像，其 ID 为 7e72a9e24671，这一步底层使用的是类似 docker commit 的命令；
    + 删除临时容器 b59a91983696 。
+ 新镜像7e72a9e24671构建成功。 

通过 docker images 查看镜像信息。 

{% asset_img image-20191214193802027.png %}

镜像 ID 为 7e72a9e24671，与构建时的输出一致。在上面的构建过程中，指令 RUN 的执行过程会在启动的临时容器中执行操作，并通过 commit 保存为新的镜像。

#### docker history

ubuntu-with-vim 是通过在 base 镜像的顶部添加一个新的镜像层而得到的。这个新镜像层的内容由`RUN apt-get update && apt-get install -y vim`生成。这一点我们可以通过`docker history` 命令验证。

{% asset_img image-20191214193900934.png %}

`docker history`会显示镜像的构建历史，也就是 Dockerfile 的执行过程。ubuntu-with-vim 与 ubuntu 镜像相比，确实只是多了顶部的一层 7e72a9e24671，由`apt-get`命令创建，大小为 83.7MB。`docker history`也向我们展示了镜像的分层结构，每一层由上至下排列。

#### 进入ubuntu-with-vim

{% asset_img image-20191214194330041.png %}

`-it` 参数的作用是以交互模式进入容器，并打开终端。`412b30588f4a` 是容器的内部 ID。

### Dockerfile常用指令总结

| 指令       | 作用                                                         |
| ---------- | ------------------------------------------------------------ |
| FROM       | 指定 base 镜像。                                             |
| MAINTAINER | 设置镜像的作者，可以是任意字符串。                           |
| COPY       | 将文件从 build context 复制到镜像。                          |
| ADD        | 与 COPY 类似，从 build context 复制文件到镜像。不同的是，如果 src 是归档文件（tar, zip, tgz, xz 等），文件会被自动解压到 dest。 |
| ENV        | 设置环境变量，环境变量可被后面的指令使用。                   |
| EXPOSE     | 指定容器中的进程会监听某个端口，Docker 可以将该端口暴露出来。 |
| VOLUME     | 将文件或目录声明为 volume。                                  |
| WORKDIR    | 为后面的 RUN, CMD, ENTRYPOINT, ADD 或 COPY 指令设置镜像中的当前工作目录。 |
| RUN        | 在容器中运行指定的命令。                                     |
| CMD        | 容器启动时运行指定的命令。Dockerfile 中可以有多个 CMD 指令，但只有最后一个生效。CMD 可以被 docker run 之后的参数替换。 |
| ENTRYPOINT | 设置容器启动时运行的命令。<br/Dockerfile 中可以有多个 ENTRYPOINT 指令，但只有最后一个生效。CMD 或 docker run 之后的参数会被当做参数传递给 ENTRYPOINT。 |