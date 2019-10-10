---
title: 认识 Ruby Rack
date: 2018-08-24 23:14:13
tags: [Ruby, Rack, Web]
categories: [Web Backend]
---

你可能听说过 [Rails](https://rubyonrails.org/)、[Sinatra](http://sinatrarb.com/) 这些 Web 框架，也使用过其中一二。但如果你不知道 [Rack](https://rack.github.io/)，那只能说你 Web 开发停留在表面上。Rack 是这些 Web 框架的基础，它们正是构建在 Rack 之上。了解 Rack 可以让更好你理解 Rails、Sinatra 这些 Web 框架背后运作机制，处理问题时得心应手。下面就开始我们的 Rack 之旅吧～

<!--more-->

## 什么是 Rack

Rack 提供一组轻量接口，实现了 Web 服务器与 Rack 应用间的交互。其交互过程下图所示：

![Web Server 与 Rack App 交互](/images/posts/1_meet_ruby_rack/web_service_and_rack_app_connection.png)

当用户在 User Agent 发起 HTTP 请求后，Rack Web Server 接收请求并调起 Rack App (如 Rails、Sinatra　等)响应， 最后 Web Server 将响应返回给 User Agent。


那么一个 Rack 应用如何实现呢？下面是一个最简单的 Rack 应用：

```ruby
# app.rb -v1

app = proc do |env|
  [200, {'Content-Type' => 'text/plain'}, ['hello rack']]
end
```

一个 Rack 应用是一个能响应 call 方法的对象，其中 call 方法接收一个 env 参数，并返回含有三个元素数组：

1. 第一个元素是 HTTP 状态码

2. 第二个元素是 HTTP Headers，使用 hash 定义

3. 第三个元素是 HTTP Body，使用一个响应 each 方法的对象定义

为了让上面的 Rack 应用能成功运行，还需加上两行代码，引入 Rack Server，代码如下:

```ruby
# app.rb -v1

require 'rack'

app = proc do |env|
  [200, {'Content-Type' => 'text/plain'}, ['hello rack']]
end

Rack::Handler::WEBrick.run(app)
```

如果你已经安装过 Rails、Sinatra，那么 rack 已被默认安装了。还没有？ 赶紧 `gem install rack` 吧。

执行 `ruby app.rb`，访问 localhost:8080 hello rack ~

## Rack Middleware

Rack 真的怎么这么简单么？ 其实从 Rack 应用定义中已经透露了 Rack 无限的可能性(Rack Middleware):

> 任何能响应 call 方法并接收指定参数，返回特定数组的对象都是 Rack 应用

下面是一个简单的 Rack Middleware

```ruby
# timing.rb -v1

class Timing
  def initialize(app)
    @app = app
  end

  def call(env)
    ts = Time.now
    status, headers, body = @app.call(env)
    ts = Time.now - ts
    [status, headers, ["app call #{ts} s"]]
  end
end
```

在 Rack app 中使用 Timing Middleware

```ruby
# app.rb -v2

require 'rack'
require './timing'

app = proc do |env|
  [200, {'Content-Type' => 'text/plain'}, ['hello rack']]
end

Rack::Handler::WEBrick.run(Timing.new(app))
```

快试试我们 Rack 应用起了什么变化吧～

在上面例子中, Timing Middleware 包装了原本的 Rack app，其接管了从 Rack Server 发给 Rack app 的请求。当 Timing 接收到 Rack Server 请求后，记录当前时间并调用 Rack app 响应，最后将执行时间显示处理。

从中我们可发现，Rack Middleware 其实也是一个 Rack 应用，其可以一层层嵌套。最外层的 Middleware 最早接收到 Rack Server 请求，最后作出响应。故在 Rack Middleware 栈中最上的 Middleware 更具有话语权。

利用 Rack Middleware 我们可以很好地实现用户鉴权等各种各样的业务场景。

## Rack::Builder、rackup

[Rack::Builder](https://github.com/rack/rack/blob/master/lib/rack/builder.rb) 是 Rack 内置的模块，通常与命令 [rackup](https://github.com/rack/rack/blob/master/bin/rackup) 一起使用。

让我们用 Rack::Builder 改造下之前 rack app

```ruby
# app.rb -v3

require 'rack'
require './timing'

app = Rack::Builder.new do
  use Timing
  run proc {|env| [200, {'Content-Type' => 'text/plain'}, ['hello rack']]}
end

Rack::Handler::WEBrick.run(app)
```

上面使用 Rack::Builder 去挂载 Timing、 app。

你说和之前什么差别？接着往下看哈

```ruby
# app.rb -v3

class App
  def self.call(env)
    [200, {'Content-Type' => 'text/plain'}, ['hello rack']]
  end
end
```

```ruby
# config.ru -v1

require './app'
require './timing'

use Timing
run App
```

代码结构是不是更清晰了，完美～　赶紧执行 `rackup` 看看吧

你还可以在 `config.ru` 中配置路由!!

我们先添加下面的 app2

```ruby
# app2.rb -v1

class App2
  def self.call(env)
    [200, {'Content-Type' => 'text/plain'}, ['Hello App2']]
  end
end
```

在 `config.ru` 中定义路由 `/app2` 指向 App2

```ruby
# config.ru -v2

require './app'
require './app2'
require './timing'

map '/app2' do
  run App2
end

map '/' do
  use Timing
  run App
end
```

这样以 `/app2` 开头的路由都会去访问 App2，其他路由则访问 App

另外 Rack Middleware 也可添加参数，下面对 Timing 添加 options 参数

```ruby
# timing.rb -v2

class Timing
  def initialize(app, options = {}, &block)
    @app = app
    @options = options
    yield if block_given?
  end

  def call(env)
    ts = Time.now
    status, headers, body = @app.call(env)
    ts = Time.now - ts
    [status, headers, ["app run pid #{Process.pid if @options[:pid]}, executed time #{ts} s"]]
  end
end
```

```ruby
# config.ru -v3

require './app'
require './app2'
require './timing'

map '/app2' do
  run App2
end

map '/' do
  use Timing, pid: true do
    puts "hello Timing"
  end
  run App
end
```

## Rails on Rack

我们之前提到 Rails 是一个 Rack 应用，在 Rails 项目根目录下你可以发现 `config.ru` 文件

```ruby
# This file is used by Rack-based servers to start the application.

require_relative 'config/environment'

run Rails.application
```

Rails 没有使用 Middleware？ 不存在的

执行 `rails middleware`，你会发现 Rails 默认已启用了很多 Middleware

```
use Rack::Sendfile
use ActionDispatch::Static
use ActionDispatch::Executor
use ActiveSupport::Cache::Strategy::LocalCache::Middleware
use Rack::Runtime
use Rack::MethodOverride
use ActionDispatch::RequestId
use Sprockets::Rails::QuietAssets
use Rails::Rack::Logger
use ActionDispatch::ShowExceptions
use WebConsole::Middleware
use ActionDispatch::DebugExceptions
use ActionDispatch::RemoteIp
use ActionDispatch::Reloader
use ActionDispatch::Callbacks
use ActiveRecord::Migration::CheckPending
use ActionDispatch::Cookies
use ActionDispatch::Session::CookieStore
use ActionDispatch::Flash
use Rack::Head
use Rack::ConditionalGet
use Rack::ETag
run Milog::Application.routes
```

## Rack Env

前面提到 Rack 应用接收了一个 env 变量作为输入。其实这个 env 变量包含了 HTTP 请求的所有信息。

现在让我们检看下其有什么信息，在 app 中打印 env 信息

```ruby
# app.rb -v4
class App
  def self.call(env)
    env.each {|key, value| puts "#{key}=#{value}"}
    [200, {'Content-Type' => 'text/plain'}, ['hello rack']]
  end
end
```

其包含以下信息

```
GATEWAY_INTERFACE=CGI/1.1
PATH_INFO=/favicon.ico
QUERY_STRING=
REMOTE_ADDR=::1
REMOTE_HOST=::1
REQUEST_METHOD=GET
REQUEST_URI=http://localhost:9292/favicon.ico
SCRIPT_NAME=
SERVER_NAME=localhost
SERVER_PORT=9292
SERVER_PROTOCOL=HTTP/1.1
SERVER_SOFTWARE=WEBrick/1.4.2 (Ruby/2.6.0/2018-08-25)
HTTP_HOST=localhost:9292
HTTP_CONNECTION=keep-alive
HTTP_USER_AGENT=Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/62.0.3202.89 Safari/537.36
HTTP_ACCEPT=image/webp,image/apng,image/*,*/*;q=0.8
HTTP_DNT=1
HTTP_REFERER=http://localhost:9292/
HTTP_ACCEPT_ENCODING=gzip, deflate, br
HTTP_ACCEPT_LANGUAGE=zh-CN,zh;q=0.9,en;q=0.8,ja;q=0.7
HTTP_COOKIE=user_locale=zh-CN; Hm_lvt_24f17767262929947cc3631f99bfd274=1532421006,1533536332,1533791498,1533867618; oschina_new_user=false
rack.version=[1, 3]
rack.input=#<Rack::Lint::InputWrapper:0x000055b88351bc60>
rack.errors=#<Rack::Lint::ErrorWrapper:0x000055b88351bc38>
rack.multithread=true
rack.multiprocess=false
rack.run_once=false
rack.url_scheme=http
rack.hijack?=true
rack.hijack=#<Proc:0x000055b88351bf30@/home/jinhu/.rvm/gems/ruby-head/gems/rack-2.0.5/lib/rack/lint.rb:525>
rack.hijack_io=
HTTP_VERSION=HTTP/1.1
REQUEST_PATH=/favicon.ico
rack.tempfiles=[]
```

可以发现这些信息分为大写的 [CGI 变量](https://getfullstack.com/web_server/server_programming/cgi.html)以及 [rack 变量](https://www.rubydoc.info/github/rack/rack/master/file/SPEC#label-The+Environment)


## References

1. [Rack](http://rack.github.io/)

2. [Ruby Rack 及其应用 (上)](https://ruby-china.org/topics/31592)