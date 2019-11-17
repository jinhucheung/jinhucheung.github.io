---
title: 使用 Capistrano 部署 Rails 应用
date: 2019-09-28 16:43:42
tags: [Capistrano, Ruby, Rails, Web, Deployment]
categories: [运维]
---

[Capistrano](https://capistranorb.com/) 是一个使用 Ruby 构建的自动化部署工具，它除了可以部署 Ruby 应用外，也支持部署 PHP, Python 等其他语言的应用。下面就让我们开始学习如何使用 Capistrano 部署 Rails 应用吧～

<!--more-->

## 搭建服务器环境

我们目标是将 Rails 应用部署到服务器 deploy 用户下。当然你也可以部署到其他地方，部署流程大同小异:

1. 登录服务器添加 deploy 用户。
2. 切换 deploy 用户并生成 ssh 公钥，上传到代码托管平台（如 github、gitlab)，因为 Capistrano 默认通过 ssh 拉取我们的代码。
3. 安装 ruby 环境并安装 bundler。
4. 安装 node 环境，如果 Rails 应用使用了 Webpack 编译资源，还需要安装 yarn。
5. 安装数据库并启动。

通过以上流程，我们搭建了 Rails 应用的基本部署环境。下面我们需要准备一个 Rails 应用并为其集成 Capistrano。

## 准备 Rails 应用

如果已经有了需要部署的 Rails 应用，可以跳过本章节。

让我们创建一个 Rails 应用，启用 Webpack 编译资源并加入 react 前端库:

```
$ rails new capistrano-demo --webpack=react
```

当前代码可看: [capistrano-demo:v0.0.1](https://github.com/jinhucheung/capistrano-demo/tree/v0.0.1)

## "Capify" Rails

### 集成 Capistrano 环境

首先我们需要为 Rails 应用集成 Capistrano 环境。

将 [capistrano](https://github.com/capistrano/capistrano) 和 [capistrano-rails](https://github.com/capistrano/rails) 这两个 gem 加入到 Gemfile 中。

如果前面服务器通过 rvm 或者 rbenv 安装 ruby 环境，还需要加入相应的 [capistrano-rvm](https://github.com/capistrano/rvm) 或者 [capistrano-rbenv](https://github.com/capistrano/rbenv) gem，以让 Capistrano 在服务器部署时找到相应的 ruby 环境。

如果应用使用 Puma 或者 Passenger 等服务部署，还可以使用 [capistrano3-puma](https://github.com/seuros/capistrano-puma) 或 [capistrano-passenger](https://github.com/capistrano/passenger) 等 gem 管理 Web 服务。

下面是我的 Gemfile:

```ruby
# Gemfile
group :development do
  gem 'capistrano', '~> 3.11', require: false
  gem 'capistrano-rails', '~> 1.4', require: false
  gem 'capistrano-rvm'
  gem 'capistrano3-puma'
end
```

执行 `$ bundle` 安装依赖，之后我们就可以通过 `$ cap -T` 来查看 Capistrano 任务。

### 编写 Capfile

执行以下命令生成 Capfile:

```
$ cap install
```

然后 Capistrano 会生成以下文件:

```
├── Capfile
├── config
│   ├── deploy
│   │   ├── production.rb
│   │   └── staging.rb
│   └── deploy.rb
└── lib
    └── capistrano
            └── tasks
```

其中 `Capfile` 作为入口文件，用于引入依赖库。 `config/deploy.rb` 是各部署环境的通用配置，而 `config/deploy/` 下的文件则是针对各环境的具体配置， `lib/capistrano/tasks/` 可自定义些 rake 任务便于部署。

之后我们就要编写 Capistrano 配置文件

```ruby
# Capfile

# 引入之前加入的 Gem 包
require "capistrano/rvm"   # 根据服务器 ruby 环境决定是否引入 rvm
require "capistrano/bundler"
require "capistrano/rails/assets"
require "capistrano/rails/migrations"
require 'capistrano/puma'
install_plugin Capistrano::Puma
```

```ruby
# config/deploy.rb

# 部署应用名
set :application, "capistrano-demo"
# 远端仓库地址
set :repo_url, "git@github.com:jinhucheung/capistrano-demo.git"
# Rails 部署环境
set :rails_env, 'production'
# 服务器部署目录
set :deploy_to, "/home/deploy/capistrano-demo"
# 服务器部署时共享的文件，通常是每次发版都不会变更的文件，它们保存在服务器的 ${deploy_to}/shared 目录下
# 如果你使用的 Rails 5.2 以下版本，将 `config/master.key`，改为 `config/secrets.yml`
append :linked_files, "config/master.key", "config/database.yml"
append :linked_dirs, "log", "tmp/pids", "tmp/cache", "tmp/sockets", "public/system", "public/packs", "node_modules"
```

```ruby
# config/deploy/production.rb

# 配置服务器 IP 地址及部署用户，其中 roles 标明此台服务器的角色，如 db 会只执行迁移任务， web 只编译资源等。
server "127.0.0.1", user: "deploy", roles: %w{app db web}
# 配置服务器环境变量
# 这里配置 Path，避免服务器找不到 yarn，你也可以在系统级别上安装 yarn
set :default_env, {
  path: "/home/deploy/.nvm/versions/node/v12.11.0/bin:$PATH"
}
```

编写好配置文件后，使用下面命令检查部署流程:

```
$ cap production deploy --dry-run
```

之后你可以在终端中看到 Capistrano 处理步骤，如果各步骤没有，那就开始我们的部署吧:

```
$ cap production deploy
```

当部署成功后，你可以看到服务器 deploy 用户多了 capistrano-demo 目录，其结构类似下面:

```
├── current -> /home/deploy/capistrano-demo/releases/20190927081948
├── releases
│   ├── 20190926100918
│   └── 20190927081948
├── repo
│   ├── branches
│   ├── config
│   ├── description
│   ├── FETCH_HEAD
│   ├── HEAD
│   ├── hooks
│   ├── info
│   ├── objects
│   ├── packed-refs
│   └── refs
└── shared
    ├── bundle
    ├── config
    ├── log
    ├── node_modules
    ├── public
    ├── puma.rb
    └── tmp
```

其中 `releases` 保存了每次部署的代码库，`current` 指向了最新的代码库, `repo` 含有最新代码库的 git 信息, `shared` 是共享文件。

当前代码可看: [capistrano-demo:v0.1.0](https://github.com/jinhucheung/capistrano-demo/tree/v0.1.0)

## 上传私密文件

如果是首次使用 Capistrano 部署，应该很快会得到一些错误。比如: master.key 不存在？ database.yml 不存在？

因为上面这些文件不入版本库，Capistrano 拉下的仓库并没有这些文件，其做软连接到这些文件就会报错了。

为了解决这个问题，我们需要将本地的这些私密文件上传到服务器。

## 创建数据库

部署还有错误，Capistrano 执行迁移任务时没有找到数据库？

因为我们首次部署时，数据库并不存在。而 Capistrano 可能出于安全还是低频使用的考虑，并没有为此增加相关任务，我们需要连接服务器手工创建数据库。

## Capistrano & Puma

Capistrano 可以通过 `capistrano3-puma` 扩展原本的部署任务，增加些管理 Puma 的任务。

使用下面的命令启动服务器的 Puma:

```
$ cap production puma:start
```

如果需要修改 Puma 的配置，则执行:

```
$ cap production puma:config
```

其会上传修改后的 Puma 配置文件至服务器，然后重启就可。

服务器 Puma 配置文件在 shared 文件夹，我们配置 NGINX 需要使用其中的信息。

## Capistrano & Sidekiq

如果 Rails 应用中使用了 Sidekiq 作为后台处理, 我们还可以通过集成 [capistrano-sidekiq](https://github.com/seuros/capistrano-sidekiq) 管理服务器中的 Sidekiq。

使用下面的命令就可启动服务器的 Sidekiq:

```
$ cap production sidekiq:start
```

当前代码可看: [capistrano-demo:v0.1.1](https://github.com/jinhucheung/capistrano-demo/tree/v0.1.1)

## 添加 NGINX 配置

为我们部署的应用添加 NGINX 配置:

```
upstream capistrano_demo {
  server unix:/home/deploy/capistrano-demo/shared/tmp/sockets/puma.sock fail_timeout=0; # 你部署应用的 puma.sock
}

server {
  listen 80;
  server_name capistrano_demo.com;
  root /home/deploy/capistrano-demo/current/public; # 你部署应用的 public 目录
  try_files $uri/index.html $uri @capistrano_demo;

  client_max_body_size 4G;
  keepalive_timeout 10;

  error_page 500 502 504 /500.html;

  location @capistrano_demo {
    proxy_pass http://capistrano_demo;

    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_redirect off;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_set_header  X-Forwarded-Port $server_port;
    proxy_set_header  X-Forwarded-Host $host;

    access_log /home/deploy/capistrano-demo/shared/log/nginx.access.log;
    error_log /home/deploy/capistrano-demo/shared/log/nginx.error.log;
  }

  location ^~ /assets/ { # 避免 Rails 找不到资源， Rails 4 后默认在生产模式禁止从 public 中提供静态资源，通过 RAILS_SERVE_STATIC_FILES 变量控制
    expires max;
    add_header Cache-Control public;
  }

  location ^~ /packs/ {
    expires max;
    add_header Cache-Control public;
  }
}

```

重启 NGINX，查看是否能正常访问我们的应用～

**Note**: 当我们应用使用 unix 协议时， NGINX 进程所属用户可能没有权限访问我们部署目录，这时候将会获得 403，我们需要在 NGINX 配置文件中调整 NGINX 进程用户或者将应用改成 tcp 协议。 **没有权限这是由于我们部署到 deploy 用户上 home 目录下，须确保 NGINX 用户可读到 deploy 的 home 目录。**

## Capistrano 部署流

Capistrano 从部署到回滚都有自己的工作流，我们可以在工作流中通过钩子，添加自己的定制任务。

下面是 Capistrano 部署流:

```
deploy:starting    - start a deployment, make sure everything is ready
deploy:started     - started hook (for custom tasks)
deploy:updating    - update server(s) with a new release
deploy:updated     - updated hook
deploy:publishing  - publish the new release
deploy:published   - published hook
deploy:finishing   - finish the deployment, clean up everything
deploy:finished    - finished hook
```

而 Capistrano 回滚流如下:

```
deploy:starting
deploy:started
deploy:reverting           - revert server(s) to previous release
deploy:reverted            - reverted hook
deploy:publishing
deploy:published
deploy:finishing_rollback  - finish the rollback, clean up everything
deploy:finished
```

如果需要在每次部署完，重启 sidekiq，我们可以在 `deploy:finished` 钩子中添加自己的任务:

```
task :restart_sidekiq do
  on roles(:worker) do
    execute :service, "sidekiq restart"
  end
end
after "deploy:published", "restart_sidekiq"
```

## Q&A

1. Rails 5.2 使用 Credentials 机制加密 secrets.yml，如果在部署过程中 bundle install 出现 ActiveSupport::MessageEncryptor::InvalidMessage，该怎么办?
  这是问题多半是我们修改了 config/master.key，导致解密 credentials.yml.enc 错误引起的，此时需要重新生成这两个文件，参考 [How to regenerate the master key for Rails 5.2 credentials](https://gist.github.com/db0sch/19c321cbc727917bc0e12849a7565af9)

2. 为什么 NGINX 默认 nobody 用户开启， TCP/IP domain sockets 获取资源没有权限问题， 而 UNIX domain socket 有权限问题？
  因为 UNIX domain socket 需要读取 sock 文件, 须确保 NGINX 进程用户有权限读取到此文件。

## 参考

1. [linux命令之远程登录/无密码登录-ssh,ssh-keygen,ssh-copy-id](https://blog.csdn.net/wangjunjun2008/article/details/20037101)
2. [Capistrano 3 实现 Rails 自动化部署](https://ruby-china.org/topics/18616)
3. [從零到有，用 Capistrano 將 Rails 專案部署自動化](https://blog.niclin.tw/2019/04/22/%E5%BE%9E%E9%9B%B6%E5%88%B0%E6%9C%89%E7%94%A8-capistrano-%E5%B0%87-rails-%E5%B0%88%E6%A1%88%E9%83%A8%E7%BD%B2%E8%87%AA%E5%8B%95%E5%8C%96/)
4. [RVM 实用指南](https://ruby-china.org/wiki/rvm-guide)
5. [How to regenerate the master key for Rails 5.2 credentials](https://gist.github.com/db0sch/19c321cbc727917bc0e12849a7565af9)
6. [Cannot deploy an application using Rails 5.1 Webpacker](https://github.com/koenpunt/capistrano-nvm/issues/25#issuecomment-320806448)
7. [Rails 5.2 + Puma + Capistrano3 + Nginx + Sidekiq 自动化部署](https://ruby-china.org/topics/36924)
8. [Sidekiq 精通 36 分钟](https://ruby-china.org/topics/19891)
9. [How can I serve assets in /public that are not part of the asset pipeline with puma/nginx?](https://stackoverflow.com/questions/34963529/how-can-i-serve-assets-in-public-that-are-not-part-of-the-asset-pipeline-with-p)
10. [What's the difference between Unix socket and TCP/IP socket?](https://serverfault.com/questions/124517/whats-the-difference-between-unix-socket-and-tcp-ip-socket)
11. [TCP/IP Socket和UNIX Socket区别](https://my.oschina.net/u/1433006/blog/1612446)