# Docker introduction
* Docker 是一个开源的应用容器引擎, 它基于 Go 语言开发并遵从Apache2.0协议开源。
* Docker 的使用场景: 
  * 创建一致的开发、测试、生产环境；
  * 创建资源隔离的运行时环境；
  * 云平台的基础设施之一；
  * 高性能、超大规模宿主机部署；
* Docker 是一个开源的商业产品，有两个版本：社区版（Community Edition，缩写为 CE）和企业版（Enterprise Edition，缩写为 EE）。企业版包含了一些收费服务，个人开发者一般用不到。下面的介绍都针对社区版。

# Docker architecture

## Docker Engine
Docker使用了传统的Client-Server架构, Docker daemon、Docker client与API共同组成Docker Engine。

Docker daemon: Docker最核心的后台进程,一般运行在宿主机之上，负责响应来自Docker client的请求，根据请求类型创建出指定的Job，完成构建、分发和运行Docker容器的工作

Docker API: 负责提供client与docker daemon交互的接口。

Docker 客户端(Client) : 通过命令行或者其他工具使用 [Docker API](https://docs.docker.com/reference/api/docker_remote_api) 与 Docker 的守护进程通信。

![docker engine](images/engine-components-flow.png)

一般的总架构下图所示:

![docker-arch](images/docker-arch.jpeg)

用户通过Docker client与Docker daemon通过API建立通信。daemon响应来自Docker client的请求，根据请求类型做出相应的行为。

Docker技术使用Docker镜像来创建实例容器。镜像可以从远程的仓库拉取，用户也可以上传镜像到仓库中

Docker 镜像(Image): 用于创建 Docker 容器的模板。

Docker 容器(Container) : 独立运行的一个或一组应用。

Docker 主机(Host): 一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。

Docker 仓库(Registry): 用来保存镜像，可以理解为代码控制中的代码仓库。[Docker Hub](https://hub.docker.com) 提供了庞大的镜像集合供使用。

## Docker image
Docker镜像（Image）

Docker镜像是由文件系统叠加而成。最底端是一个文件引导系统，即bootfs。Docker用户不会与引导文件系统有直接的交互。Docker镜像的第二层是root文件系统rootfs，通常是一种或多种操作系统，例如ubuntu等。在Docker中，文件系统永远都是只读的，在每次修改时，都是进行拷贝叠加从而形成最终的文件系统。Docker称这样的文件为镜像。一个镜像可以迭代在另一个镜像的顶部。位于下方的镜像称之为父镜像，最底层的镜像称之为基础镜像。最后，当从一个镜像启动容器时，Docker会在最顶层加载一个读写文件系统作为容器。

镜像是不会改变的, 如果文件系统需要发生改变, 变化会应用到最顶层, 也就是读写系统的容器.这种机制也被称为写时复制( copy on write)

![Docker Image](./images/docker_file_system.jfif)

简单的说, Docker镜像可以认为是容器的模板, 它包含容器所要运行时所需的程序、库、资源以及相关配置。镜像不包含任何动态数据，其内容在构建之后不会被改变。一个运行着的Docker容器是一个镜像的实例，多个容器可以共用一个镜像。
### Image commands
镜像常见的命令：
1. ```docker images``` 用于列出当前docker 主机上的可用镜像. ```docker image list``` or ```docker image ls```也可以有同样的效果
2. ```docker search``` 查看仓库里的镜像
3. ```docker pull``` 拉取镜像
   试试命令```docker pull alpine```  
   他实际上是执行了 ```docker pull registry.hub.docker.com/alpine:latest```  
   可以通过tag来限定版本
4. ```docker rmi``` 删除镜像
5. ```docker image prune``` 删除unused镜像
6. ```docker push```  把镜像提交到仓库
7. ```docker inspect``` 查看详细信息
   ```docker inspect -f '{{.ContainerConfig.Hostname}}' $(docker images -aq)``` Go语言的模板语法可以用于某些命令  
   ```docker inspect -f '{{.LogPath}}' $(docker ps -aq)```

## Docker container
容器（Container）

容器是Docker镜像创建的实例，是静态镜像的运行时的实体。其本质是一个与宿主机系统共享内核但与系统中的其他进程资源相隔离的进程，它可以被启动、停止、删除。容器中会运行特定的应用，包含代码和相关的依赖文件。每个运行着的容器都有一个可写层（writable layer,也称为容器层 container layer），它位于若干只读层之上。运行时的所有变化，包括对文件的写和更新，都会保存在这个层中。
### container commands
![container-lifecycle](images/container_lifecycle.png)
1. ```docker run``` 运行容器  
   ```docker run -ti --name my_container --restart=on-failure:1 alpine /bin/sh -c "while true; do echo hello world; sleep 1; done"``` 使用镜像运行一个容器
   ```-d ``` 后台运行
   ```-p -P``` 指定端口
   ```-v``` 使用持久化的存储
2. ```docker ps``` 查看所有的容器 和 ```docker container ls```同效
3. ```docker attach``` 重新附着容器
4. ```docker exec``` 在容器内容启动新进程
5. 生命周期管理， 暂停，恢复，停止，启动 pause, unpause, stop, start
6. ```docker logs``` 查看log
7. ```docker inspect``` 检查某个具体的容器
8. ```docker rm``` 删除容器
9. ```docker commit```，对容器做了修改后，把改动后的容器，再次转换为镜像
10. docker rm `docker ps -a -q` -f 删除所有容器
    
## Docker repository 
仓库（Repository）

仓库是集中存放镜像文件的场所，每个仓库中可包含多个具备不同标签的镜像。仓库类似于Git工具，当用户创建自己的镜像后可以上传到公有或私有的仓库，当需要使用时，从仓库下载过来即可。

[Docker Hub](https://hub.docker.com) 

[IBM CIO repository](https://pages.github.ibm.com/CIOCloud/cio-blog/#/artifactory/?id=building-and-pushing-image)
# Docker practice

## Build your own image
### Docker commit  
  ```docker commit -m "commit test" container-id image-name:tag```  
  不推荐使用这种方式, 因为难以复用, 不能很好的和其它方式兼容;
  
### Docker build & Dockerfile
Dockerfile是Docker用来构建镜像的脚本文件，包含自定义的指令和格式。用户可以用统一的语法命令来根据需求进行配置，在不同的平台上进行分发，简化开发人员构建镜像的复杂过程。  

#### Dockerfile commands
* COPY 复制文件
* ADD 更高级的复制文件
* CMD 容器启动命令
* ENTRYPOINT 入口点
* ENV 设置环境变量
* ARG 构建参数
* VOLUME 定义匿名卷
* EXPOSE 暴露端口
* WORKDIR 指定工作目录
* USER 指定当前用户
* HEALTHCHECK 健康检查
* ONBUILD 为他人作嫁衣裳

#### Practic
```docker build -t ubuntu:nginx -f ./Dockerfile01 .```   
```docker port``` 查看宿主分配的端口
## Start your container
```docker run -d -p 8080:80 --name nginx_test ubuntu:nginx``` 
## Interact with your container
```docker exec```
```docker logs```

# Other Topic
## Other docker command
* ```docker top```
* ```docker stats```
* ```docker system```
* ```docker info```

## Compare with VM

虚拟机和Docker都能够给一台宿主机上的应用提供隔离的运行环境。区别是什么呢？

传统的虚拟机通过在宿主主机中运行 hypervisor 来模拟一整套完整的硬件环境提供给虚拟机的操作系统。虚拟机系统看到的环境是可限制的，也是彼此隔离的。 这种直接的做法实现了对资源最完整的封装，但很多时候往往意味着系统资源的浪费。 例如，以宿主机和虚拟机系统都为 Linux 系统为例，虚拟机中运行的应用其实可以利用宿主机系统中的运行环境。

实际上在操作系统中，包括内核、文件系统、网络、PID、UID、IPC、内存、硬盘、CPU 等等，所有的资源都是应用进程直接共享的。 要想实现虚拟化，除了要实现对内存、CPU、网络IO、硬盘IO、存储空间等的限制外，还要实现文件系统、网络、PID、UID、IPC等等的相互隔离。 前者相对容易实现一些，后者则需要宿主机系统的深入支持。

随着 Linux 系统对于命名空间功能的完善实现，程序员已经可以实现上面的所有需求，让某些进程在彼此隔离的命名空间中运行。大家虽然都共用一个内核和某些运行时环境（例如一些系统命令和系统库），但是彼此却看不到，都以为系统中只有自己的存在。这种机制就是容器（Container），利用命名空间来做权限的隔离控制，利用 cgroups 来做资源分配

![compare](images/dockerVsVM0.png)

从上图右边虚拟机架构图能看出，虚拟机里在宿主操作系统和物理硬件之间多了一个中间层：Hypervisor。

Hypervisor是一种运行在物理服务器和操作系统之间的中间软件层，可允许多个操作系统和应用共享一套基础物理硬件，事实上成为虚拟环境中的“元”操作系统，Hypervisor可以协调访问服务器上的所有物理设备和虚拟机，也称为虚拟机监视器（Virtual Machine Monitor）。Hypervisor是所有虚拟化技术的核心。当服务器启动并执行Hypervisor时，它会给每一台虚拟机分配适量的内存、CPU、网络和磁盘，并加载所有虚拟机的客户操作系统，每台虚拟机有自己的虚拟操作系统和存储空间，因此需要消耗宿主机大量的物理资源，同时也需要花费一定时间来启动。

而上图左边，Docker直接运行在宿主机的操作系统上，没有Hypervisor这个中间层。Docker实际上就是运行于操作系统上的普通进程，通过Linux Primitives实现的彼此隔离，但是共享同一个操作系统内核。

正因为这种共享性，使得Docker的资源占用远小于虚拟机，而且启动速度也远远快于虚拟机