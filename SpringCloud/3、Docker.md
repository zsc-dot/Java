# 一、初识Docker



## 1.1、什么是Docker

微服务虽然具备各种各样的优势，但服务的拆分通用给部署带来了很大的麻烦。

- 分布式系统中，依赖的组件非常多，不同组件之间部署时往往会产生一些冲突。

- 在数百上千台服务中重复部署，环境不一定一致，会遇到各种问题



### 1.1.1、应用部署的环境问题

大型项目组件较多，运行环境也较为复杂，部署时会碰到一些问题：

- 依赖关系复杂，容易出现兼容性问题

- 开发、测试、生产环境有差异

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819211646591.png" alt="image-20220819211646591" style="zoom: 50%;" />

例如一个项目中，部署时需要依赖于node.js、Redis、RabbitMQ、MySQL等，这些服务部署时所需要的函数库、依赖项各不相同，甚至会有冲突。给部署带来了极大的困难。



### 1.1.2、Docker解决依赖兼容问题

而Docker确巧妙的解决了这些问题，Docker是如何实现的呢？

Docker为了解决依赖的兼容问题的，采用了两个手段：

- 将应用的Libs（函数库）、Deps（依赖）、配置与应用一起打包

- 将每个应用放到一个隔离**容器**去运行，避免互相干扰

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819211908868.png" alt="image-20220819211908868" style="zoom: 50%;" />

这样打包好的应用包中，既包含应用本身，也保护应用所需要的Libs、Deps，无需再操作系统上安装这些，自然就不存在不同应用之间的兼容问题了。

虽然解决了不同应用的兼容问题，但是开发、测试等环境会存在差异，操作系统版本也会有差异，怎么解决这些问题呢？



### 1.1.3、.Docker解决操作系统环境差异

要解决不同操作系统环境差异问题，必须先了解操作系统结构。以一个Ubuntu操作系统为例，结构如下：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819212100509.png" alt="image-20220819212100509" style="zoom: 50%;" />

结构包括：

- 计算机硬件：例如CPU、内存、磁盘等
- 系统内核：所有Linux发行版的内核都是Linux，例如CentOS、Ubuntu、Fedora等。内核可以与计算机硬件交互，对外提供**内核指令**，用于操作计算机硬件。
- 系统应用：操作系统本身提供的应用、函数库。这些函数库是对内核指令的封装，使用更加方便。



应用于计算机交互的流程如下：

- 应用调用操作系统应用（函数库），实现各种功能。用户程序基于系统函数库实现功能

- 系统函数库是对内核指令集的封装，会调用内核指令

- 内核指令操作计算机硬件



Ubuntu和CentOSpringBoot都是基于Linux内核，无非是系统应用不同，提供的函数库有差异：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819212419155.png" alt="image-20220819212419155" style="zoom: 50%;" />

此时，如果将一个Ubuntu版本的MySQL应用安装到CentOS系统，MySQL在调用Ubuntu函数库时，会发现找不到或者不匹配，就会报错了。



Docker如何解决不同系统环境的问题？

- Docker将用户程序与所需要调用的系统(比如Ubuntu)函数库一起打包
- Docker运行到不同操作系统时，直接基于打包的函数库，借助于操作系统的Linux内核来运行

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819212629942.png" alt="image-20220819212629942" style="zoom:50%;" />



### 1.1.4、总结

Docker如何解决大型项目依赖关系复杂，不同组件依赖的兼容性问题？

- Docker允许开发中将应用、依赖、函数库、配置一起**打包**，形成可移植镜像
- Docker应用运行在容器中，使用沙箱机制，相互**隔离**



Docker如何解决开发、测试、生产环境有差异的问题？

- Docker镜像中包含完整运行环境，包括系统函数库，仅依赖系统的Linux内核，因此可以在任意Linux操作系统上运行



Docker是一个快速交付应用、运行应用的技术：

- 可以将程序及其依赖、运行环境一起打包为一个镜像，可以迁移到任意Linux操作系统
- 运行时利用沙箱机制形成隔离容器，各个应用互不干扰
- 启动、移除都可以通过一行命令完成，方便快捷



## 1.2、Docker和虚拟机的区别

Docker可以让一个应用在任何操作系统中非常方便的运行。而以前我们接触的虚拟机，也能在一个操作系统中，运行另外一个操作系统，保护系统中的任何应用。



两者有什么差异呢？



**虚拟机**（virtual machine）是在操作系统中**模拟**硬件设备，然后运行另一个操作系统，比如在 Windows 系统里面运行 Ubuntu 系统，这样就可以运行任意的Ubuntu应用了。

**Docker**仅仅是封装函数库，并没有模拟完整的操作系统，如图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819213124019.png" alt="image-20220819213124019" style="zoom: 50%;" />

对比来看：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819213226421.png" alt="image-20220819213226421" style="zoom: 67%;" />



**总结**

Docker和虚拟机的差异：

- docker是一个系统进程；虚拟机是在操作系统中的操作系统
- docker体积小、启动速度快、性能好；虚拟机体积大、启动速度慢、性能一般



## 1.3、Docker架构



### 1.3.1、镜像和容器

Docker中有几个重要的概念：

**镜像（Image）**：Docker将应用程序及其所需的依赖、函数库、环境、配置等文件打包在一起，称为镜像。

**容器（Container）**：镜像中的应用程序运行后形成的进程就是**容器**，只是Docker会给容器进程做隔离，对外不可见。



一切应用最终都是代码组成，都是硬盘中的一个个的字节形成的**文件**。只有运行时，才会加载到内存，形成进程。



而**镜像**，就是把一个应用在硬盘上的文件、及其运行环境、部分系统函数库文件一起打包形成的文件包。这个文件包是只读的。

**容器**呢，就是将这些文件中编写的程序、函数加载到内存中运行，形成进程，只不过要隔离起来。因此一个镜像可以启动多次，形成多个容器进程。

镜像中的数据只能读不能写，每个容器运行时会把文件拷贝到自己独立的文件系统中，不会对别的容器产生影响。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819213658280.png" alt="image-20220819213658280" style="zoom:50%;" />



例如你下载了一个QQ，如果我们将QQ在磁盘上的运行**文件**及其运行的操作系统依赖打包，形成QQ镜像。然后你可以启动多次，双开、甚至三开QQ，跟多个妹子聊天。



### 1.3.2、DockerHub

开源应用程序非常多，打包这些应用往往是重复的劳动。为了避免这些重复劳动，人们就会将自己打包的应用镜像，例如Redis、MySQL镜像放到网络上，共享使用，就像GitHub的代码共享一样。

- DockerHub：DockerHub是一个官方的Docker镜像的托管平台。这样的平台称为Docker Registry。
- 国内也有类似于DockerHub 的公开服务，比如 [网易云镜像服务](https://c.163yun.com/hub)、[阿里云镜像库](https://cr.console.aliyun.com/)等。



我们一方面可以将自己的镜像共享到DockerHub，另一方面也可以从DockerHub拉取镜像：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819214034106.png" alt="image-20220819214034106" style="zoom: 50%;" />



### 1.3.3、Docker架构

我们要使用Docker来操作镜像、容器，就必须要安装Docker。

Docker是一个CS架构的程序，由两部分组成：

- 服务端(server)：Docker守护进程，负责处理Docker指令，管理镜像、容器等
- 客户端(client)：通过命令或RestAPI向Docker服务端发送指令。可以在本地或远程向服务端发送指令。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819214320586.png" alt="image-20220819214320586" style="zoom:50%;" />



### 1.3.4、总结

镜像：

- 将应用程序及其依赖、环境、配置打包在一起

容器：

- 镜像运行起来就是容器，一个镜像可以运行多个容器

Docker结构：

- 服务端：接收命令或远程请求，操作镜像或容器

- 客户端：发送命令或者请求到Docker服务端

DockerHub：

- 一个镜像托管的服务器，类似的还有阿里云镜像服务，统称为DockerRegistry



## 1.4、安装Docker

企业部署一般都是采用Linux操作系统，而其中又数CentOS发行版占比最多，因此我们在CentOS下安装Docker。

Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费，支持周期 7 个月），EE 即企业版，强调安全，付费使用，支持周期 24 个月。

Docker CE 分为 `stable` `test` 和 `nightly` 三个更新频道。

官方网站上有各种环境下的 [安装指南](https://docs.docker.com/install/)，这里主要介绍 Docker CE 在 CentOS上的安装。



### 1.4.1、CentOS安装Docker

Docker CE 支持 64 位版本 CentOS 7，并且要求内核版本不低于 3.10， CentOS 7 满足最低内核的要求，所以我们在CentOS 7安装Docker。



#### 1、卸载（可选）

如果之前安装过旧版本的Docker，可以使用下面命令卸载：

```sh
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-selinux \
                  docker-engine-selinux \
                  docker-engine \
                  docker-ce
```



#### 2、安装docker

首先需要大家虚拟机联网，安装yum工具

```sh
yum install -y yum-utils \
           device-mapper-persistent-data \
           lvm2 --skip-broken
```

然后更新本地镜像源：

```sh
# 设置docker镜像源
yum-config-manager \
    --add-repo \
    https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
    
sed -i 's/download.docker.com/mirrors.aliyun.com\/docker-ce/g' /etc/yum.repos.d/docker-ce.repo

yum makecache fast
```



然后输入命令：

```sh
yum install -y docker-ce
```

docker-ce为社区免费版本。稍等片刻，docker即可安装成功。



#### 3、启动docker

Docker应用需要用到各种端口，逐一去修改防火墙设置。非常麻烦，因此建议大家直接关闭防火墙！

启动docker前，一定要关闭防火墙！

```sh
# 关闭
systemctl stop firewalld
# 禁止开机启动防火墙
systemctl disable firewalld
```



通过命令启动docker：

```sh
systemctl start docker  # 启动docker服务

systemctl stop docker  # 停止docker服务

systemctl restart docker  # 重启docker服务
```



然后输入命令，可以查看docker版本：

```sh
docker -v
```

![image-20220819220717933](https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819220717933.png)



#### 4、配置镜像加速

docker官方镜像仓库网速较差，我们需要设置国内镜像服务：

参考阿里云的镜像加速文档：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors



# 2、Docker的基本操作



## 2.1、镜像操作

### 2.1.1、镜像名称

首先来看下镜像的名称组成：

- 镜名称一般分两部分组成：[repository]:[tag]。
- 在没有指定tag时，默认是latest，代表最新版本的镜像

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819221357729.png" alt="image-20220819221357729" style="zoom: 67%;" />

这里的mysql就是repository，5.7就是tag，合一起就是镜像名称，代表5.7版本的MySQL镜像。



### 2.1.2、镜像命令

常见的镜像操作命令如图：

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819221603247.png" alt="image-20220819221603247" style="zoom: 67%;" />



### 2.1.3、案例1-拉取、查看镜像

需求：从DockerHub中拉取一个nginx镜像并查看

1. 首先去镜像仓库搜索nginx镜像，比如[DockerHub](https://hub.docker.com/):

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819221850924.png" alt="image-20220819221850924" style="zoom: 67%;" />

2. 根据查看到的镜像名称，拉取自己需要的镜像，通过命令：docker pull nginx

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819222120140.png" alt="image-20220819222120140" style="zoom: 80%;" />

3. 通过命令：docker images 查看拉取到的镜像

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819222202477.png" alt="image-20220819222202477" style="zoom:80%;" />



### 2.1.4、案例2-保存、导入镜像

需求：利用docker save将nginx镜像导出磁盘，然后再通过load加载回来

1. 利用docker xx --help命令查看docker save和docker load的语法

   例如，查看save命令用法，可以输入命令：

   ```sh
   docker save --help
   ```

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819222400934.png" alt="image-20220819222400934" style="zoom:80%;" />

   命令格式：

   ```sh
   docker save -o [保存的目标文件名称] [镜像名称]
   ```

2. 使用docker save导出镜像到磁盘 

   运行命令：

   ```sh
   docker save -o nginx.tar nginx:latest
   ```

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819222636372.png" alt="image-20220819222636372" style="zoom:80%;" />

3. 使用docker load加载镜像

   先删除本地的nginx镜像：

   ```sh
   docker rmi nginx:latest
   ```

   然后运行命令，加载本地文件：

   ```sh'
   docker load -i nginx.tar
   ```

   <img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220819222806846.png" alt="image-20220819222806846" style="zoom:80%;" />



### 2.1.5、练习

需求：去DockerHub搜索并拉取一个Redis镜像

目标：

1. 去DockerHub搜索Redis镜像
2. 查看Redis镜像的名称和版本
3. 利用docker pull命令拉取镜像
4. 利用docker save命令将 redis:latest打包为一个redis.tar包
5. 利用docker rmi 删除本地的redis:latest
6. 利用docker load 重新加载 redis.tar文件



## 2.2、容器操作