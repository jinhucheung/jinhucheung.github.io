---
title: Dokku —— 搭建自己的 PaaS 系统
date: 2020-05-24 08:25:00
tags: [dokku, docker, heroku, paas]
categories: [运维]
---

背景：如果你对云平台服务 (PaaS) 感兴趣，你应该对 [Heroku](https://www.heroku.com/) 有所耳闻。Heroku 作为最早的云平台之一，深受国外开发者喜爱，其支持多种编程语言及服务，部署应用方式极致简单，开发者推送完代码后，就帮你完成应用构建到上线的工作。

笔者是在《Ruby on Rails 教程》一书中被作者安利，之后就一直使用 Heroku 部署个人应用。但 Heroku 在使用上也存在一些限制：国内访问应用较慢；免费应用一段时间未访问会休眠，重启过程较慢；未绑定信用卡用户最多只能创建 5 个免费应用...

考虑到 Heroku 的这些限制，那国内有没可以代替 Heroku 的 PaaS 平台？目前百度云、新浪云等云平台的应用引擎支持语言、插件较少，此外还需要实名备案。

那有没开源的方案呢？有的。我想起在技术分享群就 Docker 话题讨论时不时冒出的一个名词，那就是 [Dokku](http://dokku.viewdocs.io/dokku/)，一个基于容器的最小 PaaS 实现。

![dokku](https://avatars3.githubusercontent.com/u/13455795?s=400&v=4)

<!--more-->

## 介绍

[Dokku](http://dokku.viewdocs.io/dokku/) 是一个基于 Docker 实现的 PaaS 系统，助力你构建、管理应用的生命周期。Dokku 会为你的应用创建一个 Git 仓库，在你每次推送到这个仓库后，构建、发布应用的 Docker 容器。

Dokku 是 Heroku 的最小实现，具有以下一些特点：

- 支持多种部署方式：支持 Heroku buildpack 及 Docker 等
- 预留部署钩子，支持在部署过程中执行自定义脚本
- 拥有丰富的插件：主流数据库及 SSL 证书等
- 自动配置 Nginx
...

需要注意的是，Dokku 设计初心用于一台 VM 上，目前其仅支持单机部署，不支持集群。

## 安装

Dokku 须安装在 Unix 环境，要求至少 1GB 内存。按照官方文档进行安装，使用 bootstrap 脚本安装，发觉走不下去，提示不支持当前系统（Ubuntu 20.4)，仅支持 Ubuntu 16.04/18.04 x64, Debian 9+ x64 或 CentOS 7 x64。我看到官方文档中有提到 Arch Linux 安装，怀疑是 bootstrap 脚本未更新，抱着试试看，尝试使用 apt 方式进行安装：

```sh
# install docker
wget -nv -O - https://get.docker.com/ | sh

# setup dokku apt repository
wget -nv -O - https://packagecloud.io/dokku/dokku/gpgkey | apt-key add -
export SOURCE="https://packagecloud.io/dokku/dokku/ubuntu/"
export OS_ID="$(lsb_release -cs 2>/dev/null || echo "bionic")"
echo "xenial bionic focal" | grep -q "$OS_ID" || OS_ID="bionic"
echo "deb $SOURCE $OS_ID main" | tee /etc/apt/sources.list.d/dokku.list
apt-get update

# install dokku
apt-get install dokku
dokku plugin:install-dependencies --core # run with root!
```

最后成功安装上 Dokku。

## 配置

Dokku 安装成功后会询问是否使用 Web 方式进行配置。确认后在浏览器输入服务器 IP 打开页面，配置本地的 SSH 公钥及服务域名。

![dokku-config-via-web](/images/posts/17_dokku__build_your_own_paas/dokku_config_via_web.png)

其中域名设置可选择使用子域或者端口方式部署应用。如果选择子域，则需要在域名服务商后台进行泛域名设置。

如果不使用 Web 方式进行配置，则需要在命令行中进行 SSH 和域名设置。

## 使用

### 基本部署

首次部署应用时，需要登录 Dokku 服务端创建应用：

```sh
$ dokku apps:create $app_name
```

Dokku 会在 ~dokku 目录下创建同应用名的目录，并初始一个 git 仓库。

如果应用中使用到数据库，需提取安装相应插件：

```sh
$ sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git
```

然后创建数据库实例：

```sh
$ dokku postgres:create $database_name
```

最后将数据库关联应用：

```sh
$ dokku postgres:link $database_name $app_name
```

Link 会为应用容器添加 `DATABASE_URL` 的环境变量指向数据库实例，所以需要我们在应用中引用这个变量。

回到本地项目，为项目添加 Dokku 远端仓库：

```sh
$ git remote add dokku dokku@xxx.tld:$app_name
```

之后推送代码到 Dokku 仓库就会触发部署流程。

### Buildpack 部署

到这里你可能会有个疑问？为什么 Dokku 会知道应用的部署流程的呢？

Dokku 默认会使用 [Heroku Buildpack](https://devcenter.heroku.com/articles/buildpacks) 方式部署应用。Buildpack 由一系列脚本组成，会自动完成检测、构建、编译、发布等工作。

目前已支持 Ruby, Node.js, Java, Python, PHP, Go 等众多类型的应用。

### Docker 部署

当 Buildpack 方式不能满足你时，你可以使用 Docker 方式进行应用部署。在项目根目录下添加 `Dockerfile` 后，Dokku 将优先使用 Docker 方式部署应用。Dokku 在构建完容器后，将使用 `CMD` 或者 `ENTRYPOINT` 启动容器。

如果你想使用 Buildpack 代替 Docker 方式部署，此时只需要提交 `.buildpacks` 文件。

### 部署钩子

有时候你需要在部署完应用前执行一些任务，比如数据库迁移等。Dokku 支持在部署前后及发布后执行自定义的任务。我们可以在 `app.json` 中设置 `scripts.dokku.predeploy`、`scripts.dokku.postdeploy` 属性来定义部署前后须执行的任务，`Procfile` 则可定义发布后执行的命令。

### HTTPS

Dokku 官方推出了 Let's Encrypt 插件 [dokku-letsencrypt](https://github.com/dokku/dokku-letsencrypt)，使用十分方便。

安装插件：

```sh
$ sudo dokku plugin:install https://github.com/dokku/dokku-letsencrypt.git
```

设置 Let's Encrypt 通知邮箱：

```sh
$ dokku config:set --global DOKKU_LETSENCRYPT_EMAIL=your@email.tld
```

之后就可以为应用申请 SSL 证书，过程全自动：

```sh
$ dokku letsencrypt $app_name
```

最后注册一个 crontab job 用于自动续期证书：

```sh
$ dokku letsencrypt:cron-job --add
```

### 客户端

Dokku 目前没有官方正式的客户端，如果我们需要在本地环境调用服务端上的 Dokku 命令，我们可以通过 SSH 执行，如：

```bash
$ ssh -t dokku@xxx.tld apps:create demo
```

但我们仍能在官方文档中找到一些第三方的客户端，其实现原理大致也是通过 SSH 实现。

### 常用命令

- 获取应用环境变量：`dokku config $app`
- 设置应用环境变量：`dokku config:set $app APP_ID=xxx`
- 应用执行命令：`dokku run $app $command`

## 总结

如果你正在为繁琐的部署工作感到烦恼，那我建议你赶紧找个合适的工具减轻你的工作量。

Dokku 合适个人或小团队使用。当项目处于初期阶段，需要测试、演示环境，频繁部署更新时，Dokku 是你不错的选择之一。

但当你需要对部署流程进一步掌控时，这个时候你或许可以选择 [Jenkins](https://www.jenkins.io/)，甚至 [K8s](https://kubernetes.io/)。

## 参考

1. [国内外公有云对比：功能介绍、性能测试](https://yq.aliyun.com/articles/759721)
2. [国内做PaaS最成功的是哪家？为什么？](https://www.zhihu.com/question/19725441)
3. [Dokku Document](http://dokku.viewdocs.io/dokku/getting-started/installation/)
4. [使用 dokku 部署你的 Rails 应用](https://ruby-china.org/topics/32525)