---
title: DevOps - Docker 入门
date: 2019-11-16 23:13:05
tags: [Docker, 运维]
categories: [DevOps]
---

Docker 是一件令人惊艳的 DevOps 工具。无论在开发、测试、部署工作上，我们可以通过使用它节省大部分时间，即使对个人项目来说也是如此。在本文里，你可以了解到 Docker 是什么，以及如何将它应用于日常工作中。

本文使用的代码可在 GitHub 上的 [jinhucheung/docker-demo](https://github.com/jinhucheung/docker-demo) 仓库获取。

如果你已经很熟悉 Docker，甚至多次尝试应用它，请直接跳过本文。

<!--more-->

## Docker 解决的问题

在 Docker 出现之前，软件开发、部署工作都要在特定的环境上进行安装、配置和运行。如果我们没注意将运行环境的功能模块或数据改动，这可能会造成我们软件不能再工作。当我们需要在其他环境上部署软件时，也可能由于环境不同而出现各种依赖不一致的问题，导致软件安装失败。

由于上述软件开发、部署环境不同造成的各种问题，我们渴望能有一种将软件代码及其运行环境“打包”发布的方案以去解决它们。

如果你对虚拟机有所了解，你可能会有一个疑问 -- “我们不是有虚拟机技术吗？” 对的，我们可以通过虚拟机还原软件的运行环境。但虚拟机往往占用系统资源较多，安装、启动软件步骤较多。

由于虚拟机存在的这些缺点， Linux 发展出另一种虚拟化技术： Linux 容器 (Linux Containers, LXC)。Linux 容器不是模拟一个完整的操作系统，而是对进程进行隔离。或者说，在正常进程外面套了一个保护层。对于容器里面的进程来说，它接触到的各种资源都是虚拟的，从而实现与底层系统的隔离。

正是在此背景下， Docker 出现了。它基于 Linux 容器技术为我们提供了一种轻量、易用地将软件及环境一起打包的方案。

## Docker 是什么？

Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。

Docker 将软件及其依赖环境打包成一个文件，运行此文件生成一个容器。软件在容器里运行，就如同在真实的物理机上运行一样。有了 Docker, 就不用担心环境问题。

## 安装 Docker

Docker 分为社区版 (Docker Engine - Community) 和企业版 (Docker Engine - Enterprise / Docker Enterprise)。企业版包含了一些收费功能，个人开发者一般用不到，不同版本间的区别你可以在[官方文档](https://docs.docker.com/install/overview/)中了解到。下面介绍以社区版进行开展。

Docker 社区版参考以下官方文档进行安装：

- [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
- [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
- [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
- [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
- [Linux 二进制文件](https://docs.docker.com/install/linux/docker-ce/binaries/)
- [Mac](https://docs.docker.com/docker-for-mac/)
- [Windows](https://docs.docker.com/docker-for-windows/)

安装成功后，执行下面命令进行验证:

```
$ docker -v
```

Docker 守护进程默认以 Unix socket 形式运行，而不是 TCP 端口。此 Unix socket 文件属于 root 用户。这意味我们用非 root 用户执行 docker 命令时须加上 `sudo`。如果不想以 `sudo` 开头执行 docker 命令，我们需要创建一个 `docker` 用户组并将用户加入其中：

```
$ sudo groupadd docker
$ sudo usermod -aG docker $USER
```

然后登出用户会话，以确保用户成功加入到 `docker` 用户组中。

当我们执行 docker 命令，需要确保 Docker 守护进程已启动。下面将 docker 守护进程加入开机启动中：

```
# systemd
$ sudo systemctl enable docker

# upstart
$ echo manual | sudo tee /etc/init/docker.override

# chkconfig
$ sudo chkconfig docker on
```

## 开始使用 Docker

下面，我们通过一个简单的例子来感受下 Docker。

需要说明的是，国内访问 Docker 官方仓库 [Docker Hub](https://hub.docker.com/) 较慢，你可以参考此篇文章<sup>[[3]](#boost_docker_hub)</sup>设置镜像库加速访问。

首先执行下面命令将官方仓库的 image 文件(下一节会详细介绍)拉取到本地：

```
$ docker image pull library/hello-world
```

拉取成功后执行命令查看：

```
$ docker images
```

然后使用此文件构建容器：

```
$ docker container run library/hello-world
```

如果运行成功，你应该会看到 `Hello from Docker!` 的输出。

其实 `docker container run` 可以缩写为 `docker run`, 它有自动拉取 image 文件的功能，所以上述命令可简化为：

```
$ docker run library/hello-world
```

最后将以下停用的容器、挂起的 image 文件清理掉：

```
$ docker system prune
```

## 构建镜像

前面我们提到了 image 文件是一个二进制文件。Docker 将应用程序及其依赖环境打包到此文件中。只用通过这个文件才能生成容器。image 可以看做容器的模板，Docker 根据 image 生成容器的实例。同一个 image 可以生成多个同时运行的容器。

在实际运用中，一个 image 文件往往通过继承其他 image 文件，在此基础上，加入特定的设置生成而来。我们可以在 Docker Hub 中获取到满足我们需求的 image 文件，当然我们制作好 image 后，也可以推送到 Docker Hub 上。

下面尝试为一个 python 脚本生成 image 文件，构建第一个镜像。

首先让我们编写 python 脚本 run.py:

```python
# run.py
#!/usr/bin/env python

if __name__ == '__main__':
    print('Hello Python')
```

然后编写 Dockerfile 用于描述 image 如何构建：

```Dockerfile
FROM python:3.7-alpine

COPY . /app

WORKDIR /app

CMD python run.py
```

上面的 Dockerfile 描述了将要生成的 image 继承至 python:3.7-alpine 这个 image。然后需要拷贝当前目录下文件至 image 的 /app 目录下，并设置该目录为工作空间。最后设置 `python run.py` 为此 image 构建出容器的默认执行命令。

之后执行下面命令构建 image：

```
$ docker build -t jinhucheung/docker-demo .
```

最后让我们尝试用此 image 运行一个容器：

```
$ docker run --rm jinhucheung/docker-demo
```

如果运行正常，你应该会看到 `Hello Python` 的输出。另外上面命令的 `--rm` 参数意味着此容器运行完即销毁。

本章节的代码可查看 [jinhucheung/docker-demo:v0.1.0](https://github.com/jinhucheung/docker-demo/tree/v0.1.0)

## 连接容器

为了更好说明容器间如何通信，假定我们遇到了一个需求 - 为前面的 python 脚本连接 PostgreSQL 数据库。

### 完善应用脚本

首先需要在 python 脚本中引入 PostgreSQL 驱动依赖：

```
$ echo psycopg2 >> requirements.txt
$ pip install -r requirements.txt --user
```

如果本地没有安装 PostgreSQL 依赖库， pip install 将会报错。这个时候请不要着急安装，后面我们会使用 Docker 去安装。

在脚本中使用 psycopg2 连接数据库：

```python
# run.py
#!/usr/bin/env python

import psycopg2

if __name__ == '__main__':
    database_name = 'your_postgres_db'
    database_user = 'your_postgres_user'
    database_password = 'your_postgres_password'
    database_host = 'your_postgres_host'
    database_port = 'your_postgres_port'

    conn = psycopg2.connect(dbname=database_name, user=database_user,
        password=database_password, host=database_host, port=database_port)

    cursor = conn.cursor()
    cursor.execute('SELECT 1')
    print(cursor.fetchall())

    conn.close()
```

填入你的 PostgreSQL 数据库配置信息，上面脚本连接数据库成功就会输出 `[(1,)]`。没有可连接的 PostgreSQL 数据库？ 没关系，我们就是想使用 Docker 解决这类问题。

### 启用 Postgre 容器

执行以下命令使用 Docker 启动 PostgreSQL 服务：

```
$ docker run --name postgre-app -p 54321:5432 -e POSTGRES_PASSWORD=1234 -d postgres
```

上面的命令将拉取 Postgres 镜像，并在后台启动一个 Postgre 服务容器。容器将内部 5432 端口暴露在宿主机 54321 端口上，并创建了一个数据库。数据库名称为 postgres, 用户名为 postgres 且密码为 1234。

让我们更新 run.py 脚本的数据库配置信息为 Postgres 容器：

```python
...

if __name__ == '__main__':
    database_name = 'postgres'
    database_user = 'postgres'
    database_password = '1234'
    database_host = '127.0.0.1'
    database_port = '54321'
...
```

如果配置正常，执行 run.py 脚本应该能看到正确输出。

细心的你可能会发现一个问题：如果之后的 Postgres 容器信息变更，脚本里的数据库配置信息也要随之改变。如果我们就这样将脚本打包成镜像，那每次运行此镜像时，都要先进入镜像里修改代码，十分不便。

修改脚本代码引入环境变量，让 PostgreSQL 数据库的配置信息优先读取环境变量：

```python
...
import os

if __name__ == '__main__':
    database_name = os.environ.get('POSTGRES_DB', 'your_postgres_db')
    database_user = os.environ.get('POSTGRES_USER', 'your_postgres_user')
    database_password = os.environ.get('POSTGRES_PASSWORD', 'your_postgres_password')
    database_host = os.environ.get('POSTGRES_HOST', 'your_postgres_host')
    database_port = os.environ.get('POSTGRES_PORT', 'your_postgres_port')
...
```

### 完善应用镜像

接着让我们更新 Dockerfile 文件，加入 psycopg2 安装依赖（使用了阿里云镜像加入构建）：

```Dockerfile
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

RUN apk update && apk add build-base postgresql-dev

RUN pip install -r requirements.txt -i https://mirrors.aliyun.com/pypi/simple
```

然后重新构建应用镜像：

```
$ docker build -t jinhucheung/docker-demo .
```

创建 .env 文件并写入 Postgres 配置信息：

```
# .env
POSTGRES_DB=postgres
POSTGRES_USER=postgres
POSTGRES_PASSWORD=1234
POSTGRES_HOST=127.0.0.1
POSTGRES_PORT=54321
```

加入环境变量并启动应用容器：

```
$ docker run --rm --env-file .env jinhucheung/docker-demo
```

emm...连接不上？观察我们 Postgres 配置信息是否有错误 -- 注意 POSTGRES_HOST。对的，127.0.0.1 指向的是 docker-demo 容器内，并不是宿主机。尝试将 POSTGRES_HOST 改完宿主机的内网地址，如 `192.168.1.103`。Bingo! 连通了。

### 创建网络

虽然通过上述的办法，可以连通两个容器。但如果此时宿主机 IP 变更，这意味着我们将重新设置。有没一个方法可以一劳永逸呢？ 有 -- 创建 Docker 网络。

执行以下命令创建 docker-demo-network：

```
$ docker network create docker-demo-network
```

销毁之前 Postgres 容器以释放 postgre-app 名称，之后重新构建容器并将其加入网络中：

```
$ docker stop postgre-app
$ docker rm postgre-app
$ docker run --rm --name postgre-app --network docker-demo-network -e POSTGRES_PASSWORD=1234 -d postgres
```

更新 docker-demo 的 `.env` 文件，将 `POSTGRES_HOST` 修改为 `postgre-app`, `POSTGRES_PORT` 改为 `5432`。然后将 docker-demo 容器加入上面的网络中：

```
$ docker run --rm --network docker-demo-network --env-file .env jinhucheung/docker-demo
```

如果两个容器连通，应该能得到正常的信息输出。

当你有多个容器需要相互连接时，推荐使用 [Docker Compose](https://docs.docker.com/compose/) 编排、调度它们。后面我们会介绍到它。

本章节的代码可查看 [jinhucheung/docker-demo:v0.2.0](https://github.com/jinhucheung/docker-demo/tree/v0.2.0)

## 总结

这个教程简单介绍了 Docker 出现的背景以及如何使用。通过使用 Docker, 我们不仅可以节约了部署时间和成本，也可以获得愉悦的开发和部署体验。

## 参考

1. [Docker 入门教程](https://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html)
2. [Docker 传奇之 dotCloud](https://www.oschina.net/news/57838/docker-dotcloud)
3.  <a name='boost_docker_hub'></a>[Docker Hub 镜像加速器](https://juejin.im/post/5cd2cf01f265da0374189441)
4. [Docker 容器连接](https://www.runoob.com/docker/docker-container-connection.html)