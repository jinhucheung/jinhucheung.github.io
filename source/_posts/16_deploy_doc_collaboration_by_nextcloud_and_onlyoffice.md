---
title: Nextcloud + OnlyOffice 搭建企业内部网盘及文档协作系统
date: 2020-03-29 17:17:53
tags: [Nextcloud, OnlyOffice, Wiki, DocCollaboration, Docker]
categories: [运维]
---

你正需要搭建一个企业内部网盘或者文档协作系统吗？ 相信我，Nextcloud + OnlyOffice 是你一个不错的选择。

<!--more-->

## 介绍

[Nextcloud](https://nextcloud.com/) 是一款由 PHP 编写的开源网盘软件，在 [ownCloud](https://owncloud.org/) 基础上开发而来。Nextcloud 功能丰富，支持外部扩展，只需要在官方 [应用商店](https://apps.nextcloud.com/) 下载相应插件即可。Nextcloud 客户端支持多个平台。

[OnlyOffice](https://www.onlyoffice.com/) 是一款网上的文档协作系统，包含服务端和客户端套件，其中服务端 [DocumentServer](https://github.com/ONLYOFFICE/DocumentServer) 可独立部署。OnlyOffice 支持对 Office 文档的实时编辑。现已被 Nextcloud, ownCloud, Seafile 等众多软件支持集成。

## 安装

目前 OnlyOffice 官方已发布集成好 OnlyOffice 插件的 Nextcloud Docker 应用，我在此基础上加入数据库服务和一些 Nextcloud 插件。

首先下载仓库:

```
$ git clone https://github.com/jinhucheung/docker-onlyoffice-nextcloud
$ cd docker-onlyoffice-nextcloud
```

然后拷贝 docker-compose 配置文件:

```
$ cp docker-compose.yml.example docker-compose.yml
```

最后启动应用:

```
$ docker-compose up -d
```

之后打开浏览器访问 `http://localhost:4080` 对 Nextcloud 进行初始设置。

## 使用

我们需要对 Nextcloud 进行一些配置，包含应用默认语言及设置 OnlyOffice 插件等:

```
$ ./nextcloud/bin/config.sh
```

执行上面命令对 Nextcloud 设置默认语言及设置 [nextcloud/plugins](https://github.com/jinhucheung/docker-onlyoffice-nextcloud/tree/master/nextcloud/plugins) 目录中的插件。

如果需要批量注册用户，可以下命令：

```
$ ./nextcloud/bin/add_users.sh $users_csv_file_path $user_default_password
```

其中 `$users_csv_file_path` 为用户 CSV 文件路径。文件格式参考 [users.csv.example](https://github.com/jinhucheung/docker-onlyoffice-nextcloud/blob/master/nextcloud/public/data/users.csv.example)。

而 `$user_default_password` 为用户默认密码。如果要设置简单密码，需要先在 Nextcloud 后台设置密码校验。

## Q&A

### 宿主机访问不了 Docker 容器桥接网络？

此问题有很多原因，可以从 IPv4, IPv6(容器网络绑定到 IPv6 上), 防火墙，SELinux 方面上进行分析。