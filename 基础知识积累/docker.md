物理机
![[Pasted image 20240410161436.png](attach/Pasted%20image%2020240410161436.png)
虚拟机
![[Pasted image 20240410161455.png](attach/Pasted%20image%2020240410161455.png)
docker
![[Pasted image 20240410161509.png](attach/Pasted%20image%2020240410161509.png)

特点：
+ 可移植性
+ 占地小，只打包必要的依赖库
+ 共享bin和lib
##### 1. 什么是docker容器
是一种流行的开源软件平台，可以简化创建、管理、运行和分发应用程序的过程。使用容器来打包应用程序及其依赖项。可以将容器视为Docker镜像运行时的实例。

##### 2. Docker和虚拟机有什么不同
Docker是轻量级沙盒，其中运行的只是应用，虚拟机还包括额外的系统

##### 3. 什么是DockerFile
是一个文本文件，包含我们需要运行以及构建Docker镜像的所有命令，每一条命令构建一层，每一条指令描述该层如何构建。然后交给Docker，使用``docker build``根据其中指令自动构建镜像。
``` Docker
From: 为后续指令建立基础映像，是第一条指令
LABEL: 用于组织项目映像，模块，许可等，可以指定键值对，配置Docker
RUN：在当前层执行任何一条命令并创建一个新层，用于在映像层中添加一个功能层
CMD：为执行的容器提供默认值。若添加多个CMD指令，只有最后的会运行
```

##### 4. 使用Docker Compose时如何保证容器A先于容器B运行
+ Docker compose是一个用来定义和运行复杂应用的Docker工具。使用Docker容器的应用，通常由多个容器组成。使用Docker Compose不再需要shell脚本来启动容器。Compose通过一个配置文件来管理多个Docker容器。可以理解为Docker的管理工具
Docker Compose在继续下一个容器前不会等待容器准备就绪，需要我们来控制执行顺序，可以使用取决于条件，``depends_on``，如下面docker-compose.yml文件中的使用实例
```yml
version: "2.4"

services:

 backend:

   build: .    # 构建自定义镜像

   depends_on:

     - db

 db:

   image: mysql
```

##### 5. 一个完整的Docker由哪些部分组成？
+ DockerClient客户端
+ Docker Daemon守护进程
+ Docker Image镜像
+ DockerContainer容器

##### 6. 常用命令

+ 查看本地所用镜像：docker images
+ 搜索镜像：docker search mysql
+ 下载镜像：docker pull mysql
+ 下载指定版本镜像：docker pull mysql:5.7
+ 删除镜像：docker rmi -f 镜像id 镜像id

##### 7. 描述Docker容器的生命周期
+ 创建容器
+ 运行容器
+ 暂停容器（可选）
+ 取消暂停容器（可选）
+ 启动容器
+ 停止容器
+ 重启容器
+ 杀死容器
+ 销毁容器

##### 8. docker之间的容器如何隔离
+ 采用linux的namespace机制隔离资源，相当于限制了应用的视线，只能看到特定的内容，但还是可以访问系统资源，如cpu等
+ 采用Control Groups技术，控制容器中进程对系统资源的消耗
两者组合，共同构成Docker的容器隔离功能

##### 9. k8s功能
docker痛点问题：
+ 如何协调、调度、管理大量容器
+ 如何在升级应用时不中断服务
+ 如何监视应用的运行状况
+ 如何批量重新启动容器里的程序

k8s（这类软件统称编排系统）功能：
+ 处理大量容器和用户
+ 负载均衡
+ 鉴权和安全性
+ 管理服务通信
+ 多平台部署