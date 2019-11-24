---
title: DevOps - GitLab CI
date: 2019-11-24 20:01:15
tags: [GitLab, CI/CD, 运维]
categories: [DevOps]
---

[GitLab CI](https://about.gitlab.com/product/continuous-integration/) 是 GitLab 8.0 后推出的持续集成工具。用好了它，能大大提高我们软件开发、测试、部署的效率。在本文里，你可以了解到如何搭建 GitLab CI 环境及其使用。

<!--more-->

## CI

在了解 GitLab CI 前，我们需要知道持续集成是什么？

持续集成 (Continuous Integration, CI) 是一种软件工程工作流，指将代码持续集成到一个主线。期间经历如测试、构建等工作，并及时反馈工作结果。只有上一个工作按预期结束后，才进行下一个工作。

与持续集成相关的概念，还有持续部署 (Continuous Deployment, CD)。持续部署是指代码经过自动化测试、构建、部署的工作流程。

通过持续集成 / 持续部署，可以让我们产品在快速迭代的同时，也保持高质量。

## GitLab CI

前面我们提到 GitLab CI 是集成到 GitLab 中用于持续集成的工具。我们只有在项目中添加 .gitlab-ci.yml 文件定义 CI 流程， GitLab 将自动调度 GitLab Runner 实例执行流程中的各项工作，并反馈工作结果。

从上面流程可以看出 GitLab Runner 和 `.gitlab-ci.yml` 是 GitLab CI 重要的组成部分，下面就让我们了解它们～

## GitLab Runner

GitLab Runner 是一台执行 CI 工作的机器。GitLab 当要处理集成工作时，就会调起一个 Runner 实例进行处理，然后 Runner 执行工作并反馈结果给 GitLab。

你可能会问为什么不是 GitLab CI 来执行这些工作？一般来说，测试、构建等集成工作都会占用很多系统资源，如果让 GitLab CI 来处理，势必会影响 GitLab 的性能。所以 GitLab 将集成工作下派到 GitLab Runner 来执行，而 GitLab CI 则管理各项目的构建状态。

搭建一台 GitLab Runner 实例的方式有很多种，可以通过 k8s 或者常规机器，甚至共享他人分享的实例机器，下面以常规机器为例说明搭建流程。

### 安装

GitLab Runner 安装很简单，它同时支持 Linux, macOS, Windows 平台。各平台上的安装教程可看[官方文档](https://docs.gitlab.com/runner/install/)，下面以 Linux 为例进行安装：

```sh
# 下载 Linux x86-64 二进制包
$ sudo curl -L --output /usr/local/bin/gitlab-runner https://gitlab-runner-downloads.s3.amazonaws.com/latest/binaries/gitlab-runner-linux-amd64
# 设置可执行权限
$ sudo chmod +x /usr/local/bin/gitlab-runner
# 创建 gitlab-runner 用户
$ sudo useradd --comment 'GitLab Runner' --create-home gitlab-runner --shell /bin/bash
# 安装 gitlab-runner 服务并启动
$ sudo gitlab-runner install --user=gitlab-runner --working-directory=/home/gitlab-runner
$ sudo gitlab-runner start
```

### 注册

在安装完 Runner 后，我们需要进行注册，让将其与 GitLab 进行联通：

```sh
$ sudo gitlab-runner register
```

此时进入注册流程：

1. 输入 GitLab 服务地址: 可以为你的私有 GitLab
2. 输入 GitLab CI Token: 可在项目设置或者系统后台 CI 设置页面中获取，系统后台可将此 Runner 设置为共享
3. 输入 Runner 实例的描述
4. 输入 Runner 实例的 tags: 默认下 Runner 只执行对应 tag 的工作，[相关说明](https://docs.gitlab.com/ee/ci/yaml/#tags)
5. 选择 Runner 实例的执行服务: 各执行服务的区别可看[官方文档](https://docs.gitlab.com/runner/executors/), 推荐使用 docker
6. 上一步选择 docker 后，输入默认的 docker 镜像

通过上述步骤就完成了 Runner 的注册工作。

**Note**: 前面我们选择了 Runner 实例的执行服务，我们需要确保此服务在实例上正常工作。以 docker 为例，我们需要进行安装并启动 docker 的守护进程。

### 设置

1. 项目共享 Runner
如果你的 GitLab Runner 不是系统后台设置的共享 Runner, 而是在项目设置添加的 Runner。你可能会想在其他项目中使用此 Runner, 然而在其他项目的设置中并没看到此共享 Runner。如何设置？可进入此 Runner 编辑页面，将 "Lock to current projects" 解锁，然后就可以在其他项目中启用它。

2. 让 Runner 执行未打标签的工作
辛辛苦苦设置好的 Runner, 发现并没有执行。此时要观察 CI 工作是否未打标签或者标签不一致。如果想让 Runner 执行未打标签的工作，需在 Runner 编辑页面将 "Run untagged jobs" 启用

3. Runner 的高级设置
Runner 启动后，会在 `/etc/gitlab-runner/config.toml` 生成配置文件。我们可通过它对 Runner 进行高级设置。[详细说明](https://docs.gitlab.com/runner/configuration/advanced-configuration.html)

## .gitlab-ci.yml

在本文中，我们提到多次 "流程", "工作"。那在 GitLab CI 中如何定义这些 "流程", "工作" 呢？下面先让我们了解 GitLab CI 中的一些概念。

### 概念

1. Pipeline: 一个工作流，里面包含了多个工作，如测试、构建、部署等
2. Stages: 定义一个工作流经历的各个阶段，为各个工作归类
3. Jobs: 工作单元，一个 Stage 可以并行支持多个 Job
4. Artifacts: 一个 Job 完成后，生成的文件或目录，可在下一个 Job 中使用

### 介绍

在了解上述概念后，对于我们掌握 `.gitlab-ci.yml` 的编写有很大的帮助。`.gitlab-ci.yml` 就是 GitLab 用于编排 CI 工作的配置文件。我们只要将它添加在项目根目录下，GitLab 会检查该文件是否合法，确定合法后就会启用 CI 构建流程。

### 使用

我们先看看 `.gitlab-ci.yml` 怎么编写：

```yml
stages:
  - test
  - build

test_app:
  stage: test
  script: echo "testing app"

build_app:
  stage: build
  script: echo "building app"
```

看起来很简单对吧。我们用 `stages` 定义了 Pipeline 的各个阶段，然后定义了 `test_app` 和 `build_app` 两项工作，其中各个工作的 `script` 表示了当真正要执行的命令。只有在 `test_app` 正常结束后，`build_app` 才会开始执行。

下面我们使用 GitLab CI 搭建一个 GitLab Pages，你可以在[此仓库](https://gitlab.com/jimcheung/jimcheung.gitlab.io/)获取相关代码。

```yml
# .gitlab-ci.yml
image: node:8

stages:
  - build
  - optimize

cache:
  paths:
    - node_modules/

before_script:
  - npm install -g hexo
  - npm install -g font-spider
  - npm install

.deploy:
  artifacts:
    paths:
      - public
  only:
    - source

build:
  stage: build
  extends: .deploy
  script:
    - sed -i "s~https://jinhucheung.github.io~https://jimcheung.gitlab.io~g" _config.yml
    - hexo clean
    - hexo generate

compress_font:
  stage: optimize
  extends: .deploy
  script:
    - sed -i "s~/css/style.css~`pwd`/public/css/style.css~g" ./public/**/*.html
    - font-spider ./public/**/*.html --debug
    - sed -i "s~`pwd`/public/css/style.css~/css/style.css~g" ./public/**/*.html
```

上面的 `.gitlab-ci.yml` 定义了使用 [Hexo](https://hexo.io/) 构建 Pages 的工作流，其中增加了 [font-spider](http://font-spider.org/) 用于压缩字体。

你可以在[官方文档](https://docs.gitlab.com/ee/ci/yaml/)了解到 `.gitlab-ci.yml` 更多的配置参数。

## 总结

GitLab CI 作为 GitLab 内置的持续集成工具。通过使用它不仅为我们节省大部分时间，也让我们更了解 GitLab 整个生态。GitLab CI 的开发工作仍在持续迭代中，它仍有很多地方待我们发觉，完善。

## 参考

1. [持续集成是什么？](http://www.ruanyifeng.com/blog/2015/09/continuous-integration.html)
2. [用 GitLab CI 进行持续集成](https://scarletsky.github.io/2016/07/29/use-gitlab-ci-for-continuous-integration/)