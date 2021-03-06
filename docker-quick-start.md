# Docker 快速入门指南

[TOC]

## 0. 简介

本文档不是一篇 Docker 的完全手册，而更倾向于任务驱动型的快速指南 + 简单的原理解说，旨在帮助入门用户快速建立 Docker 的基本概念、使用逻辑、解决常用问题、并具备一定的自主分析的能力。

本文档不能取代规范、全面的学习，以下几本参考书比较完整的介绍了 Docker 的使用。

> 参考书：
>
> - [Docker -- 从入门到实践](https://yeasy.gitbooks.io/Docker_practice/)
> - [第一本Docker书（修订版）](./ebooks/%E7%AC%AC%E4%B8%80%E6%9C%ACDOCKER%E4%B9%A6%20%E4%BF%AE%E8%AE%A2%E7%89%88.pdf)
> - [Docker 官方文档（英语）](https://docs.Docker.com/)



## 1. Docker是什么？

从官方文档到各种教程都没有给 Docker 一个精确的定义。大家分别从 Docker 的发展历史、Docker 的应用范围、Docker 和 VM（虚拟机）之间的对比等几个方面来**描述** Docker，但从来没有精确地定义过 Docker。

从一个 Docker 使用者（应用开发人员、运维人员）的角度来看，需要从以下几个方面来理解它：

### 1.1 Docker 和 VM（虚拟机）

- Docker 从行为上非常类似于 VM，它们都是运行在宿主机上相互隔离的独立空间中；

- 在使用上 Docker 和 VM也非常类似，都用于**部署**独立运行的“程序”（更精确的说法是：进程），这些“程序”相互隔离，也与宿主机隔离，各自拥有自己的运行环境（如：环境变量、软件环境等）。

但是，Docker 和 VM 有着本质上的不同。下面这张图，完美解释了 Docker 和 VM 之间的区别：

![Docker与虚拟机的区别](./images/difference-between-docker-and-vm.png)

- Docker 是操作系统虚拟化；VM 是硬件虚拟化。
- Docker 虚拟化出来的操作系统和宿主机相同；VM 在虚拟硬件上可以安装任何操作系统。
  - 由于 Docker 是在 Linux 容器化技术上发展而来的，因此早期只能是 Linux。无论是 Windows 还是 macOS，都是先安装了一个 Linux 虚拟机，然后在这个虚拟机上安装 Docker ，因此 Docker 中的操作系统只能是 Linux。
  - 随着 Microsoft 发布了 windows 容器（windows 10 内置），我们也可以使用 windows 原生容器来运行 windows 应用。
- Docker 的启动要比 VM 快得多，而且消耗的资源要小得多。
- Docker 的性能更好，接近原生系统的性能。

### 1.2 Docker 的使用范围

初次接触 Docker，不免产生这样的疑问：VM 可以让我们在 windows 上运行 Linux，在 macOS 上运行 windows；而 Docker 只能在 Linux 上运行 Linux，这有什么用处呢？

#### 1.2.1 快速、一致、可靠地部署软件

Linux 以灵活性著称，它保持了高度的可定制性。但是，灵活性带来的是软件部署的复杂性、不一致性、以及可靠性的降低。

- 作为 Linux 的新手，都无数次遇到：明明是按照网上教程进行部署的，但是就是不成功，还是需要花大量的时间去搜索解决方案。即使是 Linux 的资深专家，也不能保证一次性部署成功。

- 安装没有预编译版本的软件，配置编译环境就是一件让 Linux 专家也不愿意做的事儿。
- ......

**Docker 解决方案：**

- 在 Docker 的世界里，只需要成功安装一次，就可以生成一个镜像，可以把安装镜像拷贝到任意机器中，只需运行该镜像就可以了，无须重复部署。
- 在 Docker 的世界里，有一个名为：https://hub.docker.com 的公共镜像仓库，无数人为它提供软件镜像，大多数软件，无须安装，只需要下载改镜像，并运行就可以了。

#### 1.2.2 运行环境隔离

在开源的世界里，不同的软件需要依赖于（dependent）同一软件的不同版本，或者同一软件的不同配置是非常常见的现象。然而，在开源的世界里，，并不能保证软件和软件之间不冲突，也不保证同一软件的不同版本之间兼容。例如：python 2 和 python 3 并不兼容；node、vue 的不同版本也不能保证前后向兼容。由于历史的积累，老的代码可能需要老的运行环境，而新的代码势必要使用新的运行环境来保证新的特性。

虽然各个软件为版本升级、迁移，都提出类似的解决方案，例如：python 用 anaconda、node 用 nvm，但是没有一个统一、一致的方法。

**Docker 解决方案：**

- 在 Docker 的世界里，这个问题变得非常简单，为每个软件运行一个独立的 Docker 容器就可以了。由于容器之间是隔离的，可以为每个软件部署独立的软件运行环境。

#### 1.2.3 使用 Docker 后的软件部署架构

以一个前后端分离的软件系统为例，前端为：vue-ui，后端为：node + koa、spring boot，使用了 mysql、redis 数据库，并使用了 mqtt 消息队列服务的系统，其部署结构可能如下图所示：

![Docker 软件部署结构](./images/containerized-software-deployment.png)

- 上图列出的所有容器均有官方镜像
  - vue 运行环境可以使用 nginx 官方镜像，或者直接加载在 envoy 容器中
  - node 运行环境可以使用 PM2 官方镜像
  - java 运行环境可以使用 tomcat 官方镜像

- 使用 Docker-compose 编写部署脚本，一条命令部署系统，非常方便研发团队统一开发环境，测试团队建立测试环境、以及实施团队部署到客户环境。

### 1.3 术语

- **镜像（image）：**
  - 镜像包含了容器运行时所需的程序、库、资源、配置等文件。
  - 从软件使用者角度来看，它类似于可执行文件，包含了软件执行所需的代码、数据、配置等。
  - 从软件开发者的角度来看，它类似于类定义。
  - 从 VM 使用者角度来看，它和 VM 镜像的作用是相同的，但是存取原理不一样。
  - Docker 镜像是一个特殊的文件系统，由 Docker 命令进行管理，用户并不能直接访问该文件系统。
- **容器（container）：**
  - 容器是镜像运行时的实体（实例）。
  - 容器可以被创建、启动、停止、删除、暂停等。
  - 从软件开发者的角度来看，它类似于类实例。
  - 一个镜像可以对应多个容器。
- **仓库（repository）：**
  - 用来集中存放镜像文件的地方，类似于 git 代码仓库。
  - 和 git 代码仓库类似，一个镜像可以有多个版本，称之为 tag（标签），用`<仓库名>:<标签>`来唯一确定一个镜像。



## 2. 安装、配置 Docker

### 2.1 安装 Docker

Docker 安装很简单，可以参考[官方安装指南](https://docs.Docker.com/get-Docker/)。

#### 2.1.1 insight Docker installation

- Docker 从起源上来说是基于 Linux 的容器化技术发展而来的，因此，在 Linux 上安装 Docker 是原生的。ubuntu 可以通过添加官方软件源来安装 Docker。
- 在 windows、macOS 上通过 `Docker Desktop for Windows/Mac` 来安装 Docker。该安装程序先安装 Linux 虚拟机，然后在虚拟机上安装 Linux 原生 Docker，最后安装 desktop 应用来操控虚拟机中的 Docker。
- windows 上安装 Docker 又复杂一些
  - Docker desktop for windows 上使用 Hyper-V 虚拟机，必须硬件开启 Hyper-V。
  - macOS Boot Camp 不支持 Hyper-V，无法成功安装启动 Docker desktop for windows。
  - windows 10 (1607) 或 windows server 2016 以上版本，可以切换到 windows 容器，来运行 windows 应用。
  - windows 容器比较新，相关的应用镜像也很少，基本上需要靠自己来构建镜像。

### 2.2 使用 Docker Registry Mirror

#### 2.2.1 术语

- **registry：**为了方便大家共享镜像仓库，需要建立一个存储、管理、查找、下载镜像的服务，这个服务称之为 registry，类似于 github 之于 git。
- **Docker Registry：**Docker 官方的 registry，地址为 https://hub.docker.com

#### 2.2.2 Docker Registry Mirror

Docker Registry 在国内访问非常慢，可以通过使用国内镜像来进行加速。Docker 官方、Microsoft azure 云、网易云、有道云、七牛云均提供镜像服务。

推荐使用：

- Microsoft Azure：https://dockerhub.azk8s.cn，实测比官方镜像更稳定、快速。
- 官方镜像：https://registry.docker-cn.com，官方镜像，不用解释。
- 网易镜像：https://hub-mirror.c.163.com

设置方法，可以自行 baidu/google。

### 2.3 Docker ID

**Docker ID** 是用户访问 Docker Registry 的账号。不过 Docker Registry 是可以匿名访问的，如果不需要把自制镜像推送到 Docker Registry ，则无须注册、登录 Docker ID。



## 3. 获取、运行镜像

以获取、运行 MySQL 为例：

### 3.1 搜索 MySQL 镜像

到 https://hub.docker.com 去搜索 MySQL 镜像。

- 除了官方镜像（Offical Image）外，还有很多其它组织、个人提供的定制镜像，当然官方镜像是最可靠的。在搜索结果的右上角，标明了是否是 Offical Image。
- 镜像仓库中包括了多个版本的镜像，在 Docker 中称之为 `tag`。在 `Tags` 标签下，可以浏览仓库里所有的镜像版本。
- 使用 `<仓库名>:<tag>` 来唯一标记某个镜像，例如：`mysql:5.7` 是 mysql 5.7 版镜像。
- `tag` 标记了一个镜像，而一个镜像可以对应多个 `tag`，它们并不是一一对应的关系。例如：`mysql:5.7` 和 `mysql:5` 是同一个镜像。
- `DIGEST` 和镜像之间是一一对应关系，它位于镜像信息的左下角。
- 不带 `tag` 的镜像引用，缺省对应 `:latest` 标签，例如：`mysql` 等同于 `mysql:latest`（注意，通过 `DIGEST`，可以发现 `mysql:latest` 与 `mysql:8.0.19`、 `mysql:8.0`、`mysql:8` 是同一个镜像）。

### 3.2 下载 MySQL 5.7 镜像

```bash
$ docker pull mysql:5.7 # 下载镜像
$ docker images # 浏览已经下载到本地的镜像
```

这个没啥好解释的。

### 3.3 运行 MySQL 5.7

```bash
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:5.7
```

- 通常，如何运行一个镜像，可以在仓库的 `Description` 中找到
- 命令解释：
  - `run`：运行一个容器
  - `--name some-mysql`：命名该容器为 `some-mysql`，在后续命令中，可以通过容器名（`<container-name>`）来操作该容器，通常不需要对容器进行命名。
  - `-e MYSQL_ROOT_PASSWORD=my-secret-pw`：设置环境变量 `MYSQL_ROOT_PASSWORD` 为 `my-secret-pw`。通常，可以通过环境变量对容器进行初始化设置，例如：spring boot 中，环境变量配置的优先级高于配置文件，可以通过环境变量来改变缺省的数据库连接。这里，通过环境变量配置 MySQL 的 root 密码。
  - `-d`：在后台运行该容器。
  - `mysql:5.7`：运行的镜像。

#### 3.3.1 检查容器是否运行成功

```bash
$ docker ps -a # 列出所有容器
```

- `-a`：列出所有容器，无论是否是本用户创建的。
  - 在 Linux 上运行涉及到其它用户创建容器的命令，需要使用 `sudo`。

该命令可能的输出：

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                 NAMES
b132028b32e9        mysql:5.7           "docker-entrypoint.s…"   7 minutes ago       Up 7 minutes        3306/tcp, 33060/tcp   some-mysql
```

- `CONTAINER ID - b132028b32e9`：容器 ID，全局唯一，可以通过 ID 来操控该容器。和 git 类似，一般来说，ID 的前 2 - 4 位就足以唯一确定这个容器了，因此在引用该容器时，只需要前 2 - 4 位就可以了。
- `IMAGE - mysql:5.7`：生成该容器的镜像名。
- `COMMAND - "docker-entrypoint.s..."`：该容器执行的命令，详见“insight Docker run”。
- `CREATED - 7 minutes ago`：容器创建时间。
- `STATUS - Up 7 minutes`：容器当前状态，`Up` 表示该容器在后台运行；`Exit` 表示该容器已经退出。
- `PORTS - 3306/tcp, 33060/tcp`：容器使用的端口。
- `NAMES - some-mysql`：容器的名字，可以通过容器名来引用该容器。

#### 3.3.2 insight `docker run`

Docker 镜像可以视为”一个应用软件及其运行环境的集合“；而运行一个 Docker 镜像可以视为”在一个隔离的进程中运行该应用软件及其依赖软件包“。因此，镜像通常会指定在开始运行时自动启动的应用软件，就像操作系统启动时，会自动启动某些应用程序一样。可以通过 Docker 命令 `history` 或 `inspect` 来查看容器到底启动了什么应用软件。

```bash
$ docker history mysql:5.7
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
d5cea958d330        2 days ago          /bin/sh -c #(nop)  CMD ["mysqld"]               0B
<missing>           2 days ago          /bin/sh -c #(nop)  EXPOSE 3306 33060            0B
<missing>           2 days ago          /bin/sh -c #(nop)  ENTRYPOINT ["docker-entry…   0B
<missing>           2 days ago          /bin/sh -c ln -s usr/local/bin/Docker-entryp…   34B
<missing>           2 days ago          /bin/sh -c #(nop) COPY file:3f9ea5eebe1c6044…   12.8kB
<missing>           2 days ago          /bin/sh -c #(nop)  VOLUME [/var/lib/mysql]      0B
<missing>           2 days ago          /bin/sh -c {   echo mysql-community-server m…   320MB
<missing>           2 days ago          /bin/sh -c echo "deb http://repo.mysql.com/a…   56B
<missing>           2 days ago          /bin/sh -c #(nop)  ENV MYSQL_VERSION=5.7.29-…   0B
<missing>           2 days ago          /bin/sh -c #(nop)  ENV MYSQL_MAJOR=5.7          0B
<missing>           2 days ago          /bin/sh -c set -ex;  key='A4A9406876FCBD3C45…   30.2kB
<missing>           2 days ago          /bin/sh -c apt-get update && apt-get install…   50.2MB
<missing>           3 weeks ago         /bin/sh -c mkdir /Docker-entrypoint-initdb.d    0B
<missing>           3 weeks ago         /bin/sh -c set -x  && apt-get update && apt-…   4.44MB
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV GOSU_VERSION=1.7         0B
<missing>           3 weeks ago         /bin/sh -c apt-get update && apt-get install…   10.2MB
<missing>           3 weeks ago         /bin/sh -c groupadd -r mysql && useradd -r -…   329kB
<missing>           3 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B
<missing>           3 weeks ago         /bin/sh -c #(nop) ADD file:003d2bac85e72555e…   55.3MB
```

- 注意第一行输出：`... CMD ["mysqld"]`，说明该镜像启动时，会自动执行 `mysqld`，也就是 MySQL 服务进程！

- Docker 关于自动启动的设计比上述还略微复杂一点，事实上，自动启动命令有两个：第三行输出 `... ENTRYPOINT ["docker-entrypoint.sh"]` 也用于指定自动启动命令。`mysql:5.7` 镜像完整的自动启动命令为：`docker-entrypoint.sh mysqld`。这样的设计是有其特殊目的的，随后的章节我们会继续讨论这个话题。

#### 3.3.3 映射 MySQL 服务端口

使用 `docker ps -a` 命令可以看到 `some-mysql` 容器打开了两个端口：`3306/tcp, 33060/tcp`，显然这是 `mysqld` 使用的服务端口。但是请注意：该端口是容器内部端口，宿主机是无法直接访问到的（这个论断也不完全正确，这里暂时不讨论 Docker network 的相关内容，后面的章节会进一步讨论），因此需要把容器内端口映射到宿主机。

```bash
$ docker run --name mysql-1 -p 4000:3306 -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
```

- 不能创建同名容器，此容器命名为 `mysql-1`。
- `-p 4000:3306`：将容器内部端口 `3306` 映射到主机端口 `4000`。

#### 3.3.4 测试 MySQL 5.7 服务

```bash
$ mysql -h 127.0.0.1 -P 4000 -u root -p
```

- mysql root 用户密码为：`root`
- 注意：不能使用 `localhost` 来进行连接，因为 `mysql` 对 `localhost` 连接使用了 unix socket：`/tmp/mysql.sock`，而这个套接字存在在容器内部，而不是宿主机上。

#### 3.3.5 数据持久化

**“数据持久化”**是一个专业术语，大白话就是：将数据存储在宿主机上。

容器是一个与宿主机隔离的进程，存储在容器内部的数据，当容器删除后也会删除。为了保证服务重启、迁移时数据不丢失（称之为：**持久化**），必须将数据存储在宿主机上。

持久化的原理很简单：将容器内的目录映射到宿主机的某个目录。

```bash
$ docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root -v /data/mysql:/var/lib/mysql -d mysql:5.7
```

- `-v /data/mysql:/var/lib/mysql`：将容器内的 `/var/lib/mysql` 目录映射到宿主机上的 `/data/mysql` 目录。由于 MySQL 的数据存储在 `/var/lib/mysql` 目录中，映射后就存储在宿主机的 `/data/mysql` 目录中。
- 容器停止、删除，均不会影响宿主机上的数据；重新部署容器，可以恢复以前的数据。
- 宿主机上的数据可以单独备份、恢复和迁移。
- 请注意：不要手工创建宿主机上的映射目录，手工创建容易发生读写权限问题。但是，必须保证启动用户有创建宿主机映射目录的权限。最简单的方式是：将宿主机映射目录指定在启动用户的用户目录下。

> **“数据持久化”**应用范围很广，不仅仅用于保存用户数据，还常常用于：
>
> - 用宿主机上的配置文件取代容器内部的缺省配置文件，实现自定义配置。
> - 将宿主机上的源程序编译目录映射到容器内部，实现编译即发布的功能。

### 3.4 停止、删除容器

```bash
$ docker stop some-mysql mysql-1 # 停止 some-mysql mysql-1 容器
$ docker rm some-mysql mysql-1 # 删除 some-mysql mysql-1 容器
```

- 必须要先停止容器，才能删除

### 3.5 使用 docker-compose 启动 MySQL

启动容器的参数越来越长，容易发生输入错误或疏漏。`docker-compose` 是 Docker 自带的**容器编排工具**，可以通过配置文件来启动一系列容器的工具。

> - `docker-compose` 的配置文件采用了 `YAML` 格式，完整语法可以参考：[YAML 语言教程 - 阮一峰](https://www.ruanyifeng.com/blog/2016/07/yaml.html)
> - 注意！
>   - YAML 文件使用缩进来表示层级关系，同层级需要对其；上下层级使用缩进。因此，格式非常重要。
>   - YAML 文件中不允许使用 `Tab`，所有的空白需要使用“空格”

使用 `docker-compose` 启动 MySQL 的配置文件 `mysql.yml`：

```yaml
version: "3.7"  # docker-compose 版本，当然越新越好，目前最新版本为 '3.7'

services:     # 服务列表，可以支持启动多个容器
  mysql:      # 以下是 mysql 容器的配置项
    container_name: mysql-1
    image: "mysql:5.7"
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - ./data/mysql:/var/lib/mysql
```

`docker-compose` 启动、停止命令：

```bash
$ docker-compose -f mysql.yml up -d
$ docker-compose -f mysql.yml down
```

- `-f mysql.yml`：指定启动文件。如果不指定，`docker-compose` 会寻找当前目录下的 `docker-compose.yml` 作为配置文件。
- `up -d`：以后台运行方式启动。
- `down`：停止容器，并自动删除容器。



## 4. 进阶 - 部署一组容器

上一节详细描述了如何部署一个容器化服务，从宿主机上访问该服务，并对容器化服务进行数据持久化。

但是，如果某个系统是由一组容器化服务构成的（通常都是这样的），就必须考虑容器之间的通信。

按照上述的部署方式，需要将各个容器端口均映射到宿主机，容器之间可以通过宿主机端口相互访问，并对外提供服务。但是，这样部署的缺点在于：

- 将内部服务暴露在外，削弱了安全性。
- 如果将容器组部署在多台机器上，容器相互之间的访问就与宿主机的 IP 相关，不能做到透明部署，削弱了部署的一致性和可靠性。

针对以上问题，Docker 的解决方案是：引入网络层。也就是说：由 Docker 构建透明的网络层，屏蔽 cluster 带来的部署复杂性。

### 4.1 Docker 网络模型

Docker 网络模型比较简单，对中小型系统来说，使用 `bridge` 网络模型，就可以解决大多数单机部署的问题。

#### 4.1.1 Docker 网络配置

以部署 phpMyAdmin，并将它与 MySQL 连接起来为例：

```bash
$ docker network create my-admin		# 创建一个名为 my-admin 的网络
$ docker run --name dbhost --network my-admin -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
$ docker run --network my-admin -e PMA_HOST=dbhost -p 9000:80 -d phpmyadmin/phpmyadmin
```

- 部署后，宿主机不能直接访问 MySQL 服务，只能通过 http://localhost:9000 访问 phpMyAdmin，间接管理 MySQL，大大增强了 MySQL 的数据安全性。
- 命令解释：
  - `--network my-admin`：连接到名为 `my-admin` 的网络。两个容器都连接到同一个 Docker 网络上。
  - `-e PMA_HOST=dbhost`：将 `phpMyAdmin` 连接到名为 `dbhost` 的主机上。在同一个 Docker 网络上的容器，可以通过容器名相互访问。

> **Tip：**如果 `docker run` 命令引用了没有 `pull` 到本地的镜像，`docker` 会自动 `pull` 它。

#### 4.1.2 insight Docker bridge network

Docker bridge 网络模式为容器创建了独立的网络栈，保证容器内的进程使用独立的网络环境，实现容器之间、容器与宿主机之间的网络栈隔离。同时，通过宿主机上的网桥，容器可以与宿主机乃至外界进行网络通信。

![img](./images/docker-network-bridge.png)

如上图所示，可以这么理解 bridge 网络模式：

- `docker network create my-admin`：Docker 创建了一个“内部局域网”，名为 `my-admin`。
- “`my-admin`局域网“中有一个名为 `docker0` 的设备，这个设备可以视为是一个"router"，它为连接在“`my-admin`局域网”上的所有容器提供以下服务：
  - DHCP服务，为每个容器分配 IP 地址。
  - 名字服务（name service），提供主机名（容器名）和 IP 的解析。
  - NAT 转发服务及端口映射服务，让容器能和宿主机、外部网络进行相互通信。
- 对容器而言，它们连接在"`my-admin`局域网"上，通过名字服务使用“主机名”进行相互访问；并通过“router”访问宿主机和外部网络。
- 对宿主机而言，`docker0` 设备是一个网桥，桥接了宿主机的 `eth0` 网卡和内部的“router”，因此在宿主机无需给 `docker0` 分配 IP 地址，而使用 `eth0` 的 IP 地址。外部网络通过宿主机 IP 及映射到宿主机的 port，就可以访问容器提供的服务。

#### 4.1.3 删除 Docker network

删除容器后，可以删除 network 以释放资源。

```bash
$ docker network rm my-admin		# 删除名为 my-admin 的网络
```

更简单的方式是删除所有未使用的 network（没有容器关联的网络）。

```bash
$ docker network prune
```

### 4.2  用 docker-compose 启动容器组

使用 docker-compose 能简化容器组的启动，上面的例子用 docker-compose 来部署，配置文件为：

```yaml
version: "3"

services:
  mysql:
    image: "mysql:5.7"
    networks:
      my-admin:
        aliases:
          - dbhost
    environment:
      - MYSQL_ROOT_PASSWORD=root
    volumes:
      - ./data/mysql:/var/lib/mysql

  phpmyadmin:
    image: "phpmyadmin/phpmyadmin"
    networks:
      - my-admin
    environment:
      - PMA_HOST=dbhost
    ports:
      - "9000:80"

networks:
  my-admin:
```

- 这里没有用容器名来指定容器主机名，而是在 `networks` 配置中，用 `aliases` 来指定容器主机名。这种方式更加灵活，可以指定多个 `aliases` 作为容器主机名。

> **Tips：**`docker-compose down` 关闭容器组后，会自动删除所有的资源，包括：容器、网络、数据卷等。



## 5. 其它一些需要知道的事儿

### 5.1 覆盖自动启动

有些情况下，并不需要执行镜像指定的自动启动命令，可以由命令行指定用户命令覆盖自动启动命令。

```bash
$ docker run -it --rm mysql:5.7 mysql -h 172.17.0.2 -u root -p
```

- `-it`：与容器进行交互，实际上就是接管容器的输入、输出，把后台运行的容器应用带到前台来。
- `--rm`：容器退出时，自动删除该容器。本例子中：当从 mysql 退出时，自动删除容器。
- `mysql -h 172.17.0.2 -u root -p`：运行 mysql 命令（mysql 的 CLI 客户端）来取代镜像指定的 mysqld。

### 5.2 和容器内部的 shell 进行交互

有两种方式进入到容器内部：

#### 5.2.1 在启动的时候与容器内部的 shell 进行交互

在启动容器时，跳过镜像指定的自动启动命令（5.1 覆盖自动启动），运行容器内部的 shell，并与之交互。

```bash
$ docker run -it --rm mysql:5.7 /bin/sh
```

- 每个镜像（windows 容器镜像除外）都是基于 Linux 的，因此无论哪个镜像都必然有 shell - `/bin/sh`，但是并不一定有 `/bin/bash`。

- **用途：**这个用法的用处并不多，通常用于：
  - 查看容器中应用程序的缺省配置。
  - 构建自定义镜像时（详见后面的论述），在容器内部测试构建命令是否正确。
- **注意！**不要试图使用这个命令进入容器后，去修改容器的文件和配置。因为这种修改是临时性的，当容器被删除后，修改就丢失了。修改文件和配置有三种方式：
  - 使用 `-e` 命令选项，通过环境变量，在容器启动时配置应用程序。
  - 使用 `-v` 命令选项，将宿主机上的文件映射到容器内部，实现动态修改。
  - 构建自定义镜像。

#### 5.2.2 与正在运行的容器进行交互

```bash
$ docker exec -it some-mysql /bin/sh
```

- `exec`：执行运行中容器的内部命令。
- `-it`：与之交互。
- `some-mysql`：容器名，或容器ID，可以通过 `docker ps` 命令获得。
- `/bin/sh`：执行的容器内部命令。
- **用途：**
  - 在容器运行时，查看容器内部的一些信息。
  - 调试自定义镜像。

### 5.3 再讨论一下 `run` 和 `exec` 命令

`run` 和 `exec` 非常类似，只是：`run` 是启动一个新的容器，并运行指定命令；而 `exec` 是在当前容器内部执行命令。从下面这个例子，我们来实战分析一下这两个命令的差异：

**任务：**启动一个容器化的 MySQL 服务，并通过 mysql-client 来连接这个服务。

#### 5.3.1 通过宿主机上的 mysql-client 连接容器化的 MySQL 服务

```bash
$ docker run -e MYSQL_ROOT_PASSWORD=root -p 3306:3306 -d mysql:5.7
$ mysql -h 127.0.0.1 -u root -p
Enter password: root
```

**分析：**

- 宿主机要访问容器化的 MySQL 服务，则必须将容器服务端口映射到宿主机，因此使用 `-p` 选项来映射端口。
- 容器和宿主机等同于两台“主机”，宿主机访问容器化 MySQL 服务需通过 `-h` 选项指定主机 IP 地址。

#### 5.3.2 启动另一个容器中的 mysql-client 来连接容器化的 MySQL 服务

```bash
$ docker network create net-mysql
$ docker run --name mysqlSrv --network net-mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
$ docker run --network net-mysql -it --rm mysql:5.7 mysql -h mysqlSrv -u root -p
Enter password: root
```

**分析：**

- 两次执行 `run` 命令，会启动两个容器，相当于两台“主机”。
- 这两个容器要相互通信，需要在同一个 docker 网络中。
- 在同一个 docker 网络中，容器之间可以通过主机名（容器名）来相互访问。因此，在运行 mysql-client 的容器中，通过 `-h mysqlSrv` 选项来指定 MySQL 服务主机。

#### 5.3.3 在运行 MySQL 服务的容器内部执行 mysql-client

```bash
$ docker run --name mysqlSrv -e MYSQL_ROOT_PASSWORD=root -d mysql:5.7
$ docker exec -it mysqlSrv mysql -u root -p
Enter password: root
```

**分析：**

- 在运行 MySQL 服务的容器内部执行 mysql-client，实质上是在同一台”主机“上运行 MySQL 服务，以及 mysql-client，因此无需指定主机。



## 6. 实战

**任务：**部署一个前后端分离的系统，前端使用 vue-ui，后端使用 node，数据库为 MySQL，并部署 phpMyAdmin 来操作 MySQL。

**系统功能：**数据库中存有 `users` 表，前端调用 node 服务，实现对 `users` 表的搜索功能。

**源码目录：**./src

### 6.1 Step 1：构建 PM2 自动部署环境

PM2 是守护进程（daemon）管理器，它帮助 node 开发者管理和保持应用程序在线。使用 PM2 Docker image 构建一个自动部署 node 应用的环境。

配置文件 step1/docker-compose.yml：

```yaml
version: "3"
services:
  pm2:
    image: keymetrics/pm2:latest-alpine
    volumes:
      - ./api:/api
      - ./conf/pm2.json:/pm2.json
    expose: 
      - "3000"
    ports: 
      - "3000:3000"
    command: ["pm2-runtime", "start", "pm2.json"]
```

- `image: keymetrics/pm2:latest-alpine`：使用 PM2 官方镜像。
  - PM2 官方镜像针对不同版本的 node 有不同镜像，latest 对应了 node 13。
  - `latest-alpine` 是基于 alpine linux 构建的镜像，alpine 是一个面向安全的轻型 Linux 发行版本，基于 alpine 构建的镜像远远小于基于 Ubuntu 或 Debian 构建的镜像。
- `volumes`：将 `./api` 目录映射到容器的 `/api` 目录；将 `./conf/pm2.json` 文件映射到容器的 `/pm2.json` 文件。
  - **目的1：**编辑 `./api` 中的源码时，容器内的代码也同时被修改。
  - **目的2：**编辑 `./conf/pm2.json` 文件，就可以改变容器中 PM2 的启动配置。
- `expose`：公开 3000 端口。
  - 在构建 pm2 容器时，只公开了 80、443、43554 端口，为了让 node 应用被外部访问，需要公开 3000 端口。
- `ports`：将容器的 3000 端口映射到宿主机的 3000 端口。
  - 开发阶段，为了方便测试，将 api 端口公开给宿主机。
- `command: ["pm2-runtime", "start", "pm2.json"]`：覆盖容器自动启动命令。
  - PM2 使用 ecosystem file 来定义 PM2 启动配置，完整的配置说明参见[PM2 - Ecosystem File](https://pm2.keymetrics.io/docs/usage/application-declaration/)。

Ecosystem file - step1/conf/pm2.json：

```json
{
  "name": "apiSrv",
  "script": "index.js",
  "instances": 1,
  "cwd": "/api",
  "watch": true,
  "watch_options": {
    "usePolling": true
  }
}
```

- `"name": "apiSrv"`：应用的名称为 `apiSrv`。
- `"script": "index.js"`：启动脚本为 `index.js`。
- `"instances": 1`：启动 1 个实例。PM2 自带集群（cluster），可以启动多个实例构成集群，提高服务性能。
- `"cwd": "/api"`：指定工作目录为 `/api`，因此启动脚本的完整路径为 `/api/index.js`。
- `"watch": true`：监视工作目录，当工作目录中文件发生改变时，自动重启服务。结合 `"cwd"` 参数，当容器 `/api` 目录中文件，也就是宿主机 `./api` 目录中文件发生改变时，将自动重启服务。从而实现了**当源码发生改变时，引发自动发布**的功能。
- `"usePolling": true`：容器化的 PM2 无法通过 fsevents 来监视映射卷中文件的变化，需要使用 polling（轮询）方法。不过，polling 方法比较消耗 CPU 资源，在开发阶段可以这么用，部署不应使用 watch 功能。

> **Tips：**开发期间，可以使用 `docker-compose up` 来启动容器。这样，容器会在前台运行，在终端上可以看到容器内部的 console 输出，从而看到后台应用输出的调试信息。

#### 6.1.1 监控容器化 PM2 的运行状态

| Command                                         | Description                        |
| :---------------------------------------------- | ---------------------------------- |
| `docker exec -it <container-id> pm2 monit`      | 监控每个任务的 CPU/Usage，以及 log |
| `docker exec -it <container-id> pm2 list`       | 列出所有的任务                     |
| `docker exec -it <container-id> pm2 show`       | 获取某个任务的详细信息             |
| `docker exec -it <container-id> pm2 reload all` | 重新载入所有的任务                 |

### 6.2 Step 2：构建 Vue 自动部署环境

Nginx 是最流行的 Web 服务器和反向代理服务器。使用 Nginx Docker image 来部署 Vue 项目。

配置文件 step2/docker-compose.yml：

```yaml
version: "3"
services:
  pm2:
    image: keymetrics/pm2:latest-alpine
    networks:
      simple-app:
        aliases:
          - apiSrv
    volumes:
      - ./api:/api
      - ./conf/pm2.json:/pm2.json
    expose:
      - "3000"
    ports:
      - "3000:3000"
    command: ["pm2-runtime", "start", "pm2.json"]

  nginx:
    image: nginx:alpine
    networks:
      - simple-app
    volumes:
      - ./web/dist:/web
      - ./conf/nginx.conf:/etc/nginx/conf.d/default.conf
    ports:
      - 80:80

networks:
  simple-app:
```

- 没有啥新的命令，可以自己分析。
- 仍然将 PM2 容器的 3000 端口映射到宿主机，是为了保证 vue 项目的开发、调试。正式部署时，应取消这一映射。
- `dist` 目录是 vue 项目的编译发布目录。
- `/etc/nginx/conf.d/default.conf` 是 nginx 的网站配置文件。

nginx 的网站配置文件 step2/conf/nginx.conf：

```
server {
  listen 80;
  server_name localhost;

  location / {
    root  /web;
    index index.html;
  }

  location /api/hello {
    proxy_pass  http://apiSrv:3000;
  }
}
```

- nginx 配置请自行 baidu/google，作为 vue + node.js 项目的快速部署，可以参考：[Nginx 入门教程](https://xuexb.github.io/learn-nginx/example/)。
- 这里的重点是：nginx 配置了一个反向代理，把 node 应用反向代理到 `/api/hello`。这样，vue 项目中可以通过 `/api/hello` 来访问后台服务。
  - 注意：在同一 docker 网络 - `simple-app` 下，可以使用主机名 `apiSrv` 来访问 PM2 容器。
- 为了保证开发和部署的一致性，在 vue 开发服务器中也应该加入反向代理。
  - vue 项目中添加 `vue.config.js`，可以配置开发服务器的反向代理。step2/web/vue.config.js：

```javascript
module.exports = {
  devServer: {
    proxy: {
      "/api/hello": {
        target: "http://localhost:3000",
        changeOrigin: true,
        pathRewrite: {
          "^/api/hello": "/"
        }
      }
    }
  }
};
```

### 6.3 Step 3：部署 MySQL 容器及数据的备份与恢复

前面已经部署过了 MySQL + phpMyAdmin 容器，本节从实战出发，对部署过程进行优化，并讨论如何初始化、备份、恢复数据库。

#### 6.3.1 使用数据卷来存放 MySQL 数据库

前面我们讨论了使用 `-v <local directory>:<container directory>` 的命令选项将本地目录挂载到容器中的方式，来实现**数据持久化**。这种方式，在 Docker 中有一个专有名词，称为：**Bind mounts**。但是，Docker 更推荐使用 **Volumes** 的方式来实现数据持久化。相较于 Bind mounts 的方式，Volumes 方式有以下优点：

- Volumes 由 Docker 管理，不依赖于宿主机的目录结构，没有读写权限的问题。
- Volumes 更容易实现数据的备份、恢复、迁移。
- Volumes 可以同时工作在 Linux 容器和 Windows 容器上。
- Volumes 可以安全被多个容器共享。
- Volumes 可以存放在远程主机和云上，可以被加密或提供更多的管理功能。

使用 CLI 命令的方式启动一个 MySQL 容器，并挂载 Volumes 作数据持久化：

```bash
$ docker volume create my-data	# 创建一个名为 my-data 的数据卷
# 把 my-data 挂载到容器中 mysql 的数据目录
$ docker run --name mysql -e MYSQL_ROOT_PASSWORD=root -v my-data:/var/lib/mysql -d mysql:5.7
# 关闭、删除服务后，数据卷依然存在
$ docker stop mysql && docker rm mysql
$ docker volume ls -q		# 显示数据卷名
my-data
# 创建新容器，并挂载 my-data，可以恢复数据。注意：这里无需指定 root_password，因为数据库已经被初始化了。
$ docker run --name mysql-2 -v my-data:/var/lib/mysql -d mysql:5.7
# 当不再需要该数据库时，可以删除数据卷，以节约存储空间
$ docker stop mysql-2 && docker rm mysql-2
$ docker volume rm my-data
```

使用 docker-compose 来进行部署，配置文件 step3/docker-compose.yml：

```yaml
version: "3"

services:
  mysql:
    image: mysql:5.7
    container_name: mysqldb
    networks:
      simple-app:
        aliases:
          - dbhost
    volumes:
      - "my-data:/var/lib/mysql"
    environment:
      MYSQL_ROOT_PASSWORD: "root"

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    networks:
      - simple-app
    environment:
      PMA_HOST: dbhost
    ports:
      - "9000:80"

volumes:
  my-data:

networks:
  simple-app:
```

- `docker-compose down` 关闭容器后，会自动删除容器和网络，但是不会删除 Volumes。再启动时，会自动重连上次创建的 Volumes，恢复数据库。

#### 6.3.2 Volumes 和 Bind mounts 的使用场景

Volumes 和 Bind mounts 都能实现数据持久化，它们分别适用于下列的场景：

- 如果数据是属于容器内部的数据，宿主机和外部只能通过间接的方式（客户端/WEB客户端）来访问这些数据，例如：MySQL 的数据只能通过 mysql-client 或诸如 phpMyAdmin 的客户端进行访问，这类数据应该优先使用 **Volumes**，能够充分利用 Docker 的能力。
- 对经常需要通过宿主机来修改的数据，例如：配置文件、源码、编译后的可执行代码等，为了方便动态实时部署，应采用 **Bind mounts**，来保证数据更新的灵活性。

#### 6.3.3 备份、恢复、迁移 MySQL 数据

```bash
# 备份 MySQL 数据
$ docker-compose up -d	# 启动容器组
$ docker run --rm --volumes-from mysqldb -v $(pwd):/backup ubuntu tar cvzf /backup/my-data.tar.gz /var/lib/mysql
```

- 在容器运行中就可以备份数据库。
- 通过 ubuntu 的 tar 来备份数据库。
- `--volumes-from mysqldb`：将 mount 到容器 `mysqldb` 上的所有卷 mount 到新建的容器中。容器名 `mysqldb` 在上一节，通过 `container_name` 进行了定义。
- `-v $(pwd):/backup`：把宿主机当前目录 mount 到新建容器的 `/backup` 目录上。也就是：备份文件将存储在宿主机当前目录下。
- `tar cvzf /backup/my-data.tar.gz /var/lib/mysql`：执行 `tar` 命令将 `/var/lib/mysql` 目录（MySQL 的数据目录，通过 `--volumes-from mysqldb` 从容器 `mysqldb` mount 到本容器内的）备份到 `/backup/my-data.tar.gz` 文件中。
- 注意！`mysqldb` 容器运行时，不要直接备份宿主机上的目录，因为由于缓存的存在，宿主机上的目录不一定是最新的数据，应该直接备份 `mysqldb` 容器内的目录。

```bash
# 迁移、恢复 MySQL 数据
$ docker-compose up -d	# 启动容器组，只有 MySQL 容器存在时，才能恢复数据
$ docker run --rm --volumes-from mysqldb -v $(pwd):/backup ubuntu tar xvzf /backup/my-data.tar.gz
$ docker-compose restart	# 重启服务，让数据生效
```

### 6.4 Final：整合系统

为了进一步的安全性，这个系统分为两个网络：front-end、back-end。

![最终部署方案](./images/vue-node-mysql-docker-deploy-plan.png)

配置文件 final/docker-compose.yml：

```yaml
version: "3"
services:
  nginx:
    image: nginx:alpine
    networks:
      - front-end
    volumes:
      - ./web/dist:/web
      - ./conf/nginx.conf:/etc/nginx/conf.d/default.conf
    ports:
      - 80:80

  pm2:
    image: keymetrics/pm2:latest-alpine
    networks: # api 服务同时连接到两个网络，前后端网络隔离
      front-end: # 前端网络，用于为前端提供 api 服务
        aliases:
          - apiSrv # api 服务，在前端网络中的主机名为：apiSrv
      back-end: # 后端网络，用于 api 服务连接 MySQL 数据库
    volumes:
      - ./api:/api
      - ./conf/pm2.json:/pm2.json
    expose:
      - "3000"
    command: ["pm2-runtime", "start", "pm2.json"]

  mysql:
    image: mysql:5.7
    container_name: mysqldb
    networks:
      back-end:
        aliases:
          - dbhost # MySQL 在后端网络中的主机名为 dbhost
    volumes:
      - "my-data:/var/lib/mysql"
    environment:
      MYSQL_ROOT_PASSWORD: "root"

  phpmyadmin:
    image: phpmyadmin/phpmyadmin
    networks:
      - back-end
    environment:
      PMA_HOST: dbhost
    ports:
      - "9000:80"

volumes:
  my-data:

networks:
  front-end:
  back-end:
```



### 3.6 命令小结

| `docker run`                  | 解释                          |
| ----------------------------- | ----------------------------- |
| `--name container-name`       | 命名容器                      |
| `-p host-port:container-port` | 将容器 port 映射到宿主机 port |
| `-e env-variable=value`       | 指定环境变量                  |
| `-d`                          | 在后台运行容器                |
| `-it`                         | 与容器进行交互                |
| `--rm`                        | 当容器退出时，自动删除容器    |

