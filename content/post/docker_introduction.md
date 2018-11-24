---
title: "Docker应用导论"
date: 2018-11-23T22:49:22+08:00
draft: false
---

## 一.安装
根据官网指引安装即可：
https://docs.docker.com/install/

## 二.概念解释

镜像（image）& 容器（container）：镜像和容器的关系类似于操作系统中 [程序] 和 [进程] 的关系，镜像中包含应用中所需要的所有文件内容，运行当中的镜像实例，则被称为容器。

服务（service）&  栈（stack）：笔者简单理解为，单机中多容器应用的部署使用的是service工具，在集群当中部署服务时，则需要栈工具。

集群（swarm）：一系列运行docker的主机，使用swarm进行统一的配置和管理。

![Base introduction](https://upload-images.jianshu.io/upload_images/1653617-fe49fc2393d898ee.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 三.应用

###  部署单机多容器开发环境(以开源项目laradock为例)  
- docker-compose

`compose`可以用来定义和管理多容器应用，通过配置yaml格式的文件，定义所需要的容器版本以及环境变量。 

通过`docker-compose build`建立配置中设定的容器。 

docker-compose up和docker-compose down分别管理应用的开启和关闭。  

- 容器的ip地址

每次容器启动时，都会获得一个docker内网ip地址，可通过以下命令获取：

`docker inspect --format='{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' $(docker ps -aq)` 

容器之间通信即使用此ip地址。  

laradok是一个全面的PHP单机开发环境，可选择安装单机应用所需要的容器。
仅需要三步即可在docker中运行PHP应用。
 ```
 1 git clone https://github.com/Laradock/laradock.git 
 2 cp env-example .env 
 3 docker-compose up -d nginx mysql phpmyadmin redis workspace  
 ```
通过浏览器 http://localhost 即可看到页面运行的结果。

laradock默认安装当前最新发行版，对于一些基于老版本开发的项目可能会出现冲突的情况，此时需要修改镜像的版本，重新拉取：
```
1 修改laradock/.env 中容器版本**_VERSION={需要的版本}
2 运行 docker-compose build --no-cache {image-name}
```
laradock 将 [laradock文件所在目录] 映射到/var/www/目录中，
[容器配置文件]在 laradock/{container-name}/中。

- 多站点配置

在一个nginx服务器上部署多个站点
```
1 在laradock/nginx/sites 中按nginx格式编写新的站点的配置，将目录指向container中的路径
如 var/www/hello
2 在本机中的laradock同级目录下 或者 在容器中var/www/目录中添加 hello 文件夹，即得到站点根目录
```
在其中编辑站点，即可根据站点配置域名，进行访问

### 部署多台服务器负载均衡服务
在 `docker-compose.yml` 文件中设置好设备数及要部署的镜像。

`docker-machine`可以用来管理配置和管理多台docker主机(Docker Engine)。在本地操作机器上安装docker-machine工具，即可创建访问docker主机。

创建可以通过docker-machine操作的虚拟机：
```
docker-machine create --driver virtualbox myvm1
docker-machine create --driver virtualbox myvm2
```
运行 `docker-machine ls` 即可看到所创建的虚拟机列表

通过docker-machine访问虚拟机的两种方法：
```
1 docker-machine ssh myvm1 " {bash script} "
bash script可以是在单机docker应用中所用的命令
2 docker-machine env myvm1
```
运行该命令，即可进入myvm1的运行环境中，共享文件以及进行一系列的操作

`docker swarm`是官方实现的集群管理工具，
使用命令 `docker swarm init` 即可初始化一个集群的管理节点；
使用命令 `docker swarm join --token <token> <vm ip>:<ip>` 即可加入一个集群，做为其中的一个从节点。
如果需要离开当前所在集群，运行 `docker swarm leave -force` 即可。

对于管理节点，在 `docker-compose.yml` 所在目录下，运行
`docker stack  deploy -c docker-compose.yml {stack-name}`
即可在集群当中部署我们在yaml文件中所设置的内容
其中
```
docker stack ls 查看stack列表
docker stack ps {stack-name} 查看stack内container运行情况
docker stop/start stack-name 可以选择开启/停止
```

如果swarm各个节点上的http服务运行成功，那么恭喜，站在docker这个巨人的肩膀上，已经成功实现了http服务负载均衡，迈出了云计算的第一步。

## 四.原理介绍
docker使用的是典型的`C/S`架构，由client接受指令，通过与server的通信来指挥容器相关的一系列操作。
也就是说可以通过本机内部以C/S架构使用，也可以与外部部署的容器进行连接。

### 容器发展进程：

![History of container](https://upload-images.jianshu.io/upload_images/1653617-1d296724d01b3fe6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- chroot(change root directory) 

更改root目录，在Linux系统中默认的目录结构是`/`，即是以根 (root) 开始的。而在使用 chroot 之后，系统的目录结构将以指定的位置作为 `/` 位置。

使用chroot之后，系统读到的目录和文件不再是旧系统根下的而是新系统根下的（被指定的新的位置）的目录结构和文件。

作用：
```
    1 增加系统的安全性，限制了用户的权力
    新根下访问不到旧的根目录结构和文件，一般在login之前使用chroot，以此实现用户不能访问到特定的文件。
    2 建立一个与原系统隔离的系统目录结构，方便用户开发
    使用 chroot 后，系统读取的是新根下的目录和文件，这是一个与原系统根下文件不相关的目录结构。在这个新的环境中，可以用来测试软件的静态编译以及一些与系统不相关的独立开发。
    3 切换系统的根目录位置，引导 Linux 系统启动以及急救系统
    切换到一个临时系统
```

- FreeBSD jail

chroot限制进程只能存取某个部分的文件系统，但是FreeBSD jail机制限制了在jail中运行的进程，不能影响操作系统中的其他部分。运行在一个 [沙盒] 上。

```
虚拟化：每个jail都是在主机上执行的虚拟环境，有自己的档案系统，进程，普通用户和超级用户。在jail中运行的进程和实际的操作系统环境几乎是一样的。
安全性：每个软件监狱都是独立运作，与其他软件监狱隔离，因此能够提供额外的安全层级。
容易删除及创建：因为每个软件监狱的运作范围有限，这使得系统管理者可以在不影响整体系统的前提下，以超级使用者的权限，来删除在软件监狱下运作的行程。
```

- LXC(Linux Containers)

一种操作系统虚拟化技术，为Linux内核容器提供用户空间接口。它将应用软件系统打包成一个软件容器（Container），内含应用软件本身的代码，以及所需要的操作系统核心和库。透过统一的名字空间和共享API来分配不同软件容器的可用硬件资源，创造出应用程序的独立沙箱运行环境，使得Linux用户可以容易的创建和管理系统或应用容器。

- cgroups
 
其名称源自控制组群（control groups）的简写，是Linux内核的一个功能，用来限制，控制与分离一个进程组群的资源（如CPU、内存、磁盘输入输出等）。
cgroups的一个设计目标是为不同的应用情况提供统一的接口，从控制单一进程(像nice)到操作系统层虚拟化(像OpenVZ，Linux-VServer，LXC)。

在Linux内核中，提供了cgroups功能，来达成资源的区隔化。它同时也提供了名称空间区隔化的功能，使应用程序看到的操作系统环境被区隔成独立区间，包括进程树，网络，用户id，以及挂载的文件系统。但是cgroups并不一定需要启动任何虚拟机。

LXC利用cgroups与名称空间的功能，提供应用软件一个独立的操作系统环境。LXC不需要Hypervisor这个软件层，软件容器（Container）本身极为轻量化，提升了创建虚拟机的速度。软件Docker被用来管理LXC的环境。

- Docker

容器化-操作系统级虚拟化，容器们各自为政，可以互相通信，所有的容器用的是同一个操作系统内核。
0.9版本以后，不再使用LXC，代替的是go语言开发libcontainer libarary.(2014) 

- runc

根据OCI（Open Container Initiative）规范，生成和运行容器的命令行工具。（OCI,由docker和其他容器行业的领导者制定的容器运行时规范，以及镜像规范）。
0.9版本以后，docker改用了runc作为容器管理组件。

- moby

把docker-ce及部分开源打包成立新项目moby。docker区分为docker-ee和docker-ce版本，开始商业化运作。
其中Linuxkit较有研究价值。

### Docker-ce容器部分简单介绍：

![stucture of engine](https://upload-images.jianshu.io/upload_images/1653617-7fe3b81c23baec9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### containerd的架构

containerd is an industry-standard container runtime with an emphasis on simplicity, robustness and portability. It is available as a daemon for Linux and Windows, which can manage the complete container lifecycle of its host system: image transfer and storage, container execution and supervision, low-level storage and network attachments, etc.

容器运行时管理工具，包含daemon，image transfer and storage，容器执行和监听，网络连接等等。

![structure of containerd](https://upload-images.jianshu.io/upload_images/1653617-c1d0d451d5a15302.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

以下代码使用了containerd所实现的client借口，给出了一系列参数可以对容器进行操作。代码中所演示的是 [拉取redis镜像] 的操作。

```
package main

import (
   "context"
   "log"

   "github.com/containerd/containerd"
   "github.com/containerd/containerd/namespaces"
)

func main() {
   if err := redisExample(); err != nil {
      log.Fatal(err)
   }
}

func redisExample() error {
   client, err := containerd.New("/run/containerd/containerd.sock")
   if err != nil {
      return err
   }
   defer client.Close()

   ctx := namespaces.WithNamespace(context.Background(), "example")
   image, err := client.Pull(ctx, "docker.io/library/redis:alpine", containerd.WithPullUnpack)
   if err != nil {
      return err
   }
   log.Printf("Successfully pulled %s image\n", image.Name())

   return nil
}
```

示例代码运行结果

![result](https://upload-images.jianshu.io/upload_images/1653617-fc98ea53383ac6a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在示例代码中，创建了和container daemon的连接，然后设置系统用户命名空间，在其中从docker.io pull了一个redis镜像到本地。

利用containerd，我们可以定制开发自己的容器管理工具，符合OCI规范，可以使用如docker的image等其他镜像。

#### 图形化操作界面-portainer：

一个轻量级的图形化docker管理工具。
示例网站：http://demo.portainer.io/ ，依赖于docker开源项目client功能开发。

## REFERENCE:

[Docker doc - get started](http://docker.io/)

[理解chroot](https://www.ibm.com/developerworks/cn/linux/l-cn-chroot/index.html)

[github/containers](https://github.com/opencontainers)

[github/seccomp](https://github.com/seccomp/libseccomp)

[opencontainer](https://www.opencontainers.org/)

[github/portrainer](https://github.com/portainer/portainer)
