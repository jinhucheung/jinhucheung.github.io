---
title: DevOps - Docker Compose
date: 2019-11-16 23:33:06
tags: [Docker, Docker Compose, 运维]
categories: [DevOps]
---

在 “[DevOps - Docker 入门](/2019/11/16/9_devops_docker_started/)” 一文中，我们尝试连通多个容器。在这一过程中，我们需要执行很多的命令，这些命令有先后之分。而且随着我们需要连接的容器增多，此工作会更加复杂繁琐。为此 Docker 官方提供了一个编排调度 Docker 容器的工具以解放我们双手。

本文使用的代码可在 GitHub 上的 [jinhucheung/docker-demo:v0.3.0](https://github.com/jinhucheung/docker-demo/tree/v0.3.0) 仓库获取。

如果你已经在使用 Docker Compose，请直接跳过本文。

<!--more-->

## Docker Compose 介绍

[Docker Compose](https://github.com/docker/compose) 是 Docker 的一种编排工具，用于在 Docker 上定义并运行复杂的应用。通过 Docker Compose, 我们可以很容易地用一个配置文件定义一个多容器应用，然后使用一条命令安装这个应用的所有依赖，完成构建。

Docker Compose 有两个重要的概念：

- 服务 (service): 一个应用的容器。实际上可以包括若干运行相同镜像的容器实例。
- 项目 (project): 由一组关联的应用容器组合而成的一个完整业务单元，在 docker-compose.yml 中定义。

一个项目可以由多个服务(容器)组合而成， Compose 面向项目进行管理，通过调用 Compose 的命令对项目中的一组容器进行便捷地生命周期管理。

Docker Compose 实际上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。

## Docker Compose 安装

Docker Compose 独立于 Docker， 其由 Python 编写。Compose 的安装可参考[官方文档](https://docs.docker.com/compose/install/)进行安装。

安装完成后，执行下面命令进行验证：

```
$ docker-compose --version
```

## Docker Compose 应用

下面我们使用 Docker Compose 为 [jinhucheung/docker-demo:v0.2.0](https://github.com/jinhucheung/docker-demo/tree/v0.2.0) 项目编排 Postgres 以及 Docker Demo App 容器。

### 编排容器

首先创建 docker-compose.yml 文件：

```yml
version: '3.4'

x-postgres-variables: &postgres-variables
  POSTGRES_DB: postgres
  POSTGRES_USER: postgres
  POSTGRES_PASSWORD: pass
  POSTGRES_HOST: postgres
  POSTGRES_PORT: 5432

services:
  postgres:
    image: postgres:12
    environment: *postgres-variables

  docker_demo:
    build:
      context: .
      dockerfile: ./Dockerfile
    image: docker_demo_app
    environment: *postgres-variables
    depends_on:
      - postgres
```

更多的配置参数你可以在[官方文档](https://docs.docker.com/compose/compose-file/) 中了解到。

然后执行以下命令部署项目：

```
$ docker-compose up
```

Docker Compose 会自动拉取镜像、生成、执行容器并将服务加入到同一个网络中。之后等待 Compose 最后执行结束。

观察 Compose 输出，你很快会察觉到一个问题，为什么 Docker Demo 连接不上 Postgres？ Postgres 还在启动中。

对的，postgres 服务调度完后马上就启动 docker_demo 服务了。此时 Postgres 守护进程还没有完全启动，从而导致此问题。

为了解决上述的问题，你可以将 `docker-compose up` 分为两步执行：

```
$ docker-compose up -d postgres
$ docker-compose up docker_demo
```

上述命令参数 `-d` 为后台启动。

你也可以参考[官方文档](https://docs.docker.com/compose/startup-order/) 使用一个脚本阻塞服务，等待其依赖服务完全启动后再执行相关脚本。添加 wait-for-postgres.sh 脚本并将其设置为可执行：

```sh
#!/bin/sh
# wait-for-postgres.sh

set -e

cmd="$@"

until PGPASSWORD=$POSTGRES_PASSWORD psql -h "$POSTGRES_HOST" -p $POSTGRES_PORT -U $POSTGRES_USER -c '\q'; do
  >&2 echo "Postgres is unavailable - sleeping"
  sleep 1
done

>&2 echo "Postgres is up - executing command"
exec $cmd
```

由于 wait-for-postgres.sh 依赖于 psql 工具，我们需要更新 Dockerfile 文件：

```Dockerfile
RUN apk update && apk add build-base postgresql-dev postgresql
```

更新 docker-compose.yml 配置文件：

```yml
...
  docker_demo:
    build:
      context: .
      dockerfile: ./Dockerfile
    image: docker_demo_app
    environment: *postgres-variables
    command: ['./wait-for-postgres.sh', 'python', 'run.py']
    depends_on:
      - postgres
```

然后执行下面命令重新构建 docker_demo 镜像：

```
$ docker-compose build docker_demo
```

执行 `$ docker-compose up` 应该正常了。

### 恢复数据

当需要为项目添加初始数据时，我们可以使用 Postgres 镜像预留的 `docker-entrypoint-initdb.d` 目录。只需要将数据放入到上面的目录中，Postgres 镜像生成容器时就会初始化这批数据。

首先，我们使用 `pg_dump` 从本地导出一份数据：

```
$ PGPASSWORD=$database_password pg_dump $database_name -U $database_user -h $database_host -p $database_port -t  public.$tablename -Ox > table.sql
```

填入相关数据库信息后，将导一份数据表 `table.sql`。如果数据量很大，我们往往会将其进行压缩：

```
$ gzip table.sql
```

上面的命令将压缩 `table.sql` 为 `table.sql.gz`。这份数据表你可以在 [GitHub](https://github.com/jinhucheung/docker-demo/blob/v0.3.0/table.sql.gz) 上获得。

然后我们修改 `docker-compose.yml`, 将 `table.sql.gz` 挂载到 `docker-entrypoint-initdb.d` 目录中。

```yml
services:
  postgres:
    ...
    volumes:
      - ./table.sql.gz:/docker-entrypoint-initdb.d/table.sql.gz
```

之后重新构建容器，Postgres 服务将会自动导入数据：

```
$ docker-compose down
$ docker-compose up
```

如果你不想要重新构建容器，想在容器重新启动时就恢复初始数据。此时你可以添加一个 service 容器，并连接到前面 Postgres 容器上，对它进行操作。

```yml
services:
  restore_postgres:
    image: postgres:12
    environment: *postgres-variables
    volumes:
      - ./table.sql.gz:/docker-entrypoint-initdb.d/table.sql.gz
      - ./wait-for-postgres.sh:/usr/bin/wait-for-postgres.sh
      - ./restore_postgres.sh:/usr/bin/restore_postgres.sh
    command: ['./wait-for-postgres.sh', '/usr/bin/restore_postgres.sh']
    depends_on:
      - postgres
```

添加 `restore_postgres.sh` 脚本：

```bash
#!/bin/bash

export PGPASSWORD="${PGPASSWORD:-$POSTGRES_PASSWORD}"

psql=( psql -h $POSTGRES_HOST -p $POSTGRES_PORT -U $POSTGRES_USER -d $POSTGRES_DB )

${psql[@]} -c 'DROP TABLE IF EXISTS public.users CASCADE;'

gunzip -c /docker-entrypoint-initdb.d/table.sql.gz | "${psql[@]}"

unset PGPASSWORD
```

执行 `$ docker-compose up` 启动项目。你会发现 `restore_postgres` 容器正在处理 `postgres` 的数据。

## 参考

1. [Docker(四)：Docker 三剑客之 Docker Compose](http://www.ityouknow.com/docker/2018/03/22/docker-compose.html)