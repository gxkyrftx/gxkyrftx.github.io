---
layout:     post
title:      从docker到kubernetes学习
subtitle:   学习一下docker
date:       2019-9-22
author:     gxkyrftx
header-img: img/post-bg-unix-linux.jpg
catalog: true
tags:
    - 测试
    - c++
---


# 1. 基础知识

## 1.1 Paas平台

在docker之前出现过PaaS（platform as a service）平台

Paas平台的范围和内容：

1. 确定产品定位和需求，确定首次迭代的范围
2. 制作界面原型
3. 技术选型，根据技术选型为开发者搭建开发环境和技术栈
4. 构建基础技术框架和服务
5. 模拟用户容量，构建测试环境
6. 开始编写真正的业务代码，实现产品功能
7. 迭代开发/测试

老一代平台存在很多问题，达到了一个比项目成熟，但未让人完全满意的阶段，新一代Paas平台诞生

新一代的云应用平台技术实现了全方位的应用生命周期管理，具备以下特征：

1. 加速应用交付
2. 多语言和框架
3. 多服务
4. 多云部署和多IaaS

## 1.2 docker

docker是第三代Paas平台

docker是虚拟化的一种轻量级替代技术，不依赖任何语言、框架、系统，可以将app编程一种标准化的，可以值得，自管理的组件，并脱离服务器硬件在任何主流系统中开发、调试和运行

- 简单的说，在Linux系统上创建一个容器，并在容器上部署和运行应用程序，并通过配置文件可以轻松实现应用程序的自动化安装、部署和升级，非常方便。
- 容器技术将生产环境和开发环境分离，互不影响

### 1.2.1 docker核心技术

docker 0.1-0.6：LXC，cgroups，AUFS

docker 0.8+：AUFS，BTRFS，Jails，LXC，SE Linux，service-discovery

#### 1.2.1.1 cgroups

系统中有限制进程资源的需求，于是出现了cgroups（controller group），在一个group中，有分配好的特定比例的cpu时间，IO时间，可用内存大小等。

cgroups是将任意进程进行分组化管理的Linux内核方法，主要流程是：

- 挂载子系统——》创建cgroup节点——》节点控制进程id的写入，并将控制的属性写入。

cgroup资源组可以限制诸多资源，例如：cpu，mem，iops，iobandwide，net，device

#### 1.2.1.2 LXC

LXC（Linux containers），基于容器的操作系统层级的虚拟化技术，借助于namespace的隔离机制和cgroup的限额功能。

特点：

- LXC提供了一套统一的API和工具管理container
- LXC被整合进入内核，不用单独为内核打补丁
- 容器在提供隔离的同时，通过共享内核资源，节省开销

各方面比较：

- 性能方面：LXC>>KVM>>XEN
- 内存利用率：LXC>>KVM>>XEN
- 隔离程度：XEN>>KVM>>LXC

#### 1.2.1.3 AUFS

AUFS（Advanced multi-layered Unification File System），一个能透明覆盖一或多个现有文件系统的层状文件系统。可以将不同目录联合在一起，组成一个单一的目录。

特点：

- AUFS允许docker把某些镜像作为容器的基础。
- docker具有版本容器的镜像能力，每个版本都是一个与之前版本的简单差异改动，有效的保持镜像文件的最小化

#### 1.2.1.4 app打包

在LXC基础之上，docker提供了标准统一的打包部署运行方案。

为了最大限度的重用image，容器运行的环境实际上是由具有依赖关系的多个layer组成的

有了层级image做基础，不同app可以共用底层的文件系统，相关的以来工具

#### 1.2.1.5 全生命开发周期

常用的中间件，进行标准化

### 1.2.2 docker image

- 极度精简的Linux程序运行环境
- 需要定制化Build的一个‘安装包’
- 不建议有运行期需要修改的配置文件
- dockerfile用来创建一个自定义的image，包含用户指定的软件依赖。使用build创建新的image
- docker image 最佳实践尽量重用公开镜像

### 1.2.3 docker container

- docker container是image的实例，共享内核
- docker container里可以运行不同os的image
- docker container不建议内部开启一个sshd服务，1.3版本后增加docker exec命令，进入容器排查问题
- docker container没有ip地址，通常不会有服务端口暴露

### 1.2.4 docker daemon

- docker daemon是创建和运行container的linux守护进程，最主要的核心组件
- docker daemon是docker container的container
- docker daemon可以绑定本地端口并提供Rest API服务，用来访问和控制

### 1.2.5 docker registry/hub

- 官方registry（docker hub）上可以轻松下载大量的已经容器化好的应用镜像，即拉即用
- 可以在docker hub中绑定代码托管系统，配套自动生成镜像功能

## 1.3 必备技能

### 1.3.1 linux相关

磁盘，文件，日志，用户，权限，安全，网络

### 1.3.2 虚拟机相关

vmware workstation/virtbox熟练使用，虚机克隆，组网，host-only网络，nat网络

# 2 docker基础命令

## 2.1 部署安装

### 2.1.1 个人开发测试环境

boot2docker是一个linux虚拟机，分别为windows和mac做了一个可执行文件

### 2.1.2 集群

通过docker native运行centos，ubuntu，coreos等

### 2.1.3 安装命令

- yum install docker 默认安装1.13，查看版本

```
docker version
```

![1569057923402](C:\Users\dell\Desktop\笔记\docker到k8s\1569057923402.png)

- 启动docker进程

```
service docker start
```

![1569058007786](C:\Users\dell\Desktop\笔记\docker到k8s\1569058007786.png)

- docker网桥，显示了一个私有地址，只能通过本机访问，所有容器都只能在docker0网段被分配地址

![1569064155036](C:\Users\dell\Desktop\笔记\docker到k8s\1569064155036.png)



## 2.2 配置文件与日志

- docker配置文件信息一般位于以下位置

```
/etc/sysconfig/docker
```

![1569064832791](C:\Users\dell\Desktop\笔记\docker到k8s\1569064832791.png)

- docker服务的配置文件位于

```
/usr/lib/systemd/system/docker.service
```

![1569065024955](C:\Users\dell\Desktop\笔记\docker到k8s\1569065024955.png)

- docker的日志文件位于

```
/var/log/message
```

可以使用以下命令查找

```
cat /var/log/message |grep docker
```

![1569065719336](C:\Users\dell\Desktop\笔记\docker到k8s\1569065719336.png)

## 2.3 基础命令详解

- docker search

可以找到各种各样的docker镜像

![1569150102998](C:\Users\dell\Desktop\笔记\docker到k8s\1569150102998.png)

另外一种寻找镜像的方法是通过官网https://hub.docker.com/ 寻找

- docker pull

可以拉取一个镜像到本地，默认最新的

![1569150188388](C:\Users\dell\Desktop\笔记\docker到k8s\1569150188388.png)

- docker images

显示本地的镜像

![1569150220606](C:\Users\dell\Desktop\笔记\docker到k8s\1569150220606.png)

- docker run

运行一个镜像

![1569150472916](C:\Users\dell\Desktop\笔记\docker到k8s\1569150472916.png)

```
docker run [options] image [:tag][commend][arg..]
```

-d参数会使容器运行在后台模式，可以通过docker exec进入

- docker create/start/stop/pause/unpause

容器生命周期相关的指令

# 3 自定义docker镜像

## 3.1 将容器变成镜像

docker commit  <container>  [repo:tag]

官网api地址：https://docs.docker.com/engine/reference/commandline/create/

优点：

方便，快速，适合于开发测试阶段，在本地验证结果

缺点：

不规范，无法自动化创建镜像

![1569152292077](C:\Users\dell\Desktop\笔记\docker到k8s\1569152292077.png)

## 3.2 dockerfile语法

官方文档：https://docs.docker.com/engine/reference/commandline/build/

**docker build** 命令用于使用 Dockerfile 创建镜像。提供了打包的标准化工具，一个简单的例子如下：

```
FROM nimmis/ubuntu:14.04		#基础开发环境
MAINTAINER nimmis<kjell.havneskold@gmail.com>		#维护者信息
# disable interactive functions				
ENV DEBIAN_FRONTEND noninteractive		#环境变量信息
# set default java environment variable
ENV JAVA_HOME /usr/lib/jvm/java-8-openjdk-amd64		#增加环境变量
RUN apt-get install -y software-properties-common && \add-apt-repository ppa:openjdk-r/ppa -y && \apt-get update && \apt-get install -y --no-install-recommends openjdk-8-jre && \rm -rf /var/lib/apt/lists/*		#运行命令		
```

docker的build过程，是在容器中执行的，而不是本地的操作系统

```
ADD 本地文件的位置 容器中的位置		#添加一个文件
EXPOSE 22		#开放一个端口
```

使用supervisor监管

```
[supervisord]
nodaemon=true

[program:sshd]
command=/usr/sbin/sshd -D

[program:apache2]
command=/bin/bash -c 'source /etc/apache2/envvars && exec /usr/sbin/apache2 -DFOREGROUND
```

## 3.3 镜像制作中的常见问题

从容器打包的角度考虑，一个容器究竟运行几个程序？程序参数和配置文件？程序日志输出的问题等等，都是需要考虑的



# 4 容器互联

## 4.1 基于Volume的互联

### 4.1.1 Volume简介

- Docker的文件系统是这样工作的：
  1. Docker镜像是由多个文件系统（只读层）叠加而成。
  2. 当我们启动一个容器的时候，Docker会加载只读镜像层并在其上添加一个读写层。
  3. 如果运行中的容器修改了现有的一个已经存在的文件，那该文件将会从读写层下面的只读层复制到读写层，该文件的只读版本仍然存在，只是已经被读写层中该文件的副本所隐藏。
  4. 当删除Docker容器，并通过该镜像重新启动时，之前的更改将会丢失。在Docker中，只读层及在顶部的读写层的组合被称为 Union File System（联合文件系统）
- 基于Volume的互联可以解决跨主机的共享问题。简单来说，Volume就是目录或者文件，它可以绕过默认的联合文件系统，而以正常的文件或者目录的形式存在于宿主机上。基于Volume的互联可以解决跨主机的共享问题

### 4.1.2 Volume启动方式

两种方式来初始化Volume

1.

```
docker run -it --name container-test -h CONTAINER -v /data java /bin/bash
```

上面的命令会将`/data`挂载到容器中，并绕过联合文件系统，可以在主机上直接操作该目录。任何在该镜像`/data`路径的文件将会被复制到Volume。可以使用`docker inspect`命令找到Volume在主机上的存储位置,命令如下：

```
docker inspect container-test | grep Source
```

![1569225034891](C:\Users\dell\Desktop\笔记\docker到k8s\1569225034891.png)

从主机上添加文件到此文件夹下，然后进入容器查看

![1569225351267](C:\Users\dell\Desktop\笔记\docker到k8s\1569225351267.png)

![1569225392835](C:\Users\dell\Desktop\笔记\docker到k8s\1569225392835.png)

只要将主机的目录挂载到容器的目录上，那改变就会立即生效。

2.

在Dockerfile中通过使用`VOLUME`指令来达到相同的目的

```
FROM java:wheezy
VOLUME /data
```

注意：在容器上挂载指定的主机目录是只有-v参数可以做的

```
docker run -v /home/data:/data java ls /data
```

当使用`-v`参数时，镜像目录下的任何文件都不会被复制到Volume中。（Volume会复制到镜像目录，镜像不会复制到Volume）

### 4.1.3 数据共享

如果要授权一个容器访问另一个容器的Volume，可以使用`-volumes-from`参数来执行`docker run`

```
docker run -it -h NEWCONTAINER --volumes-from container-test java /bin/bash
```

![1569228856029](C:\Users\dell\Desktop\笔记\docker到k8s\1569228856029.png)

不管container-test是否运行，它都会起作用。只要有容器连接Volume，它就不会被删除。

### 4.1.4 数据容器

常见的使用场景是使用纯数据容器来持久化数据库、配置文件或者数据文件等。

```
docker run --name dbdata postgres echo "Data-only container for postgres"
```

该命令将会创建一个已经包含在Dockerfile里定义过Volume的postgres镜像，运行`echo`命令然后退出。在运行`docker ps`命令时，`echo`可以帮助我们识别某镜像的用途。可以用`-volumes-from`命令来识别其它容器的Volume：

```
docker run -d --volumes-from dbdata --name db1 postgres
```

使用数据容器的两个注意点：

- 不要运行数据容器，这纯粹是在浪费资源。
- 不要为了数据容器而使用“最小的镜像”，只使用数据库镜像本身就可以。

### 4.1.5 备份

```
docker run --rm --volumes-from dbdata -v $(pwd):/backup debian tar cvf /backup/backup.tar /var/lib/postgresql/data

```

该示例应该会将Volume里所有的东西压缩为一个tar包（官方的postgres Dockerfile在/var/lib/postgresql/data目录下定义了一个Volume）

### 4.1.6 权限与许可

通常你需要设置Volume的权限或者为Volume初始化一些默认数据或者配置文件。要注意的关键点是，在Dockerfile的`VOLUME`指令后的任何东西都不能改变该Volume

### 4.1.7 删除Volumes

如果已经使用`docker rm` 来删除你的容器，那可能有很多的孤立的Volume仍在占用着空间。

Volume只有在下列情况下才能被删除：

- 该容器是用`docker rm －v`命令来删除的（`-v`是必不可少的）。
- `docker run`中使用了`--rm`参数


即使用以上两种命令，也只能删除没有容器连接的Volume。连接到用户指定主机目录的Volume永远不会被docker删除。

## 4.2 基于Link的互联

docker run --link可以用来链接2个容器，使得源容器（被链接的容器）和接收容器（主动去链接的容器）之间可以互相通信，并且接收容器可以获取源容器的一些数据。一个应用如下：

```bash
// 1、创建容器 test1
docker run -d --name test1 nginx

// 2、创建容器 test2并 link 到 test1。
docker run -d --name test2 --link test1 nginx

// 3、进入test2，并 ping test1，发现是可以 ping 通的。
docker exec -it test2 /bin/sh
ping test1

```

实例：

![1569231848189](C:\Users\dell\Desktop\笔记\docker到k8s\1569231848189.png)

![1569231878126](C:\Users\dell\Desktop\笔记\docker到k8s\1569231878126.png)

## 4.3 基于网络的互联



# 15 出现的问题

## 9.21

### 1.ssh连接缓慢

解决办法：

- https://www.cnblogs.com/herry52/p/5795406.html 没用
- 关闭防火墙,失败

```
firewall-cmd --state
systemctl stop firewalld.service
```



### 2.ssh连接出现警告

```
WARNING! The remote SSH server rejected X11 forwarding request
```

解决办法：

- https://blog.csdn.net/wugenqiang/article/details/86554753 已经解决

### 3.xshell登陆无法联网

xshell登陆后无法联网，在centos中使用

```
service network restart
```

重启服务后，xshell连接断开。

解决办法：

- https://blog.csdn.net/q2826621520/article/details/79920332 修改network配置文件，打开之后发现不符合
- https://blog.csdn.net/qq_34625397/article/details/80039127 配置静态ip，失败

### 4.最终解决办法

更改虚拟机网关为xxx.xxx.xxx.2,解决

![1569149847962](C:\Users\dell\Desktop\笔记\docker到k8s\1569149847962.png)

![1569149893344](C:\Users\dell\Desktop\笔记\docker到k8s\1569149893344.png)

## 9.22



