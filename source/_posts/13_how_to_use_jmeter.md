---
title: Apache JMeter 接口测试
date: 2020-02-29 14:45:39
tags: [Java, JMeter, CI/CD]
categories: [测试]
---

如果你正在寻找一款简单易用且免费的接口或性能测试工具，我相信 JMeter 是你不错的选择之一。

<!--more-->

## 介绍

[Apache JMeter](https://jmeter.apache.org/) 是 Apache 下的一款开源性能测试工具，它使用 Java 编写并提供插件扩展功能。由于 JMeter 上手简单，现在也被社区作为单元测试工具用于 HTTP, JDBC 等接口测试场景。

## 安装

### 安装 Java

JMeter 运行依赖于 Java 环境，所以我们首先需要安装 Java，安装流程参考[官方文档](https://java.com/en/download/help/download_options.xml)。

**Note**: 目前最新版本的 JMeter 要求 Java 8 以上。

### 安装 JMeter

JMeter 安装很简单，下载即可使用。我们可以从 JMeter [官方网站](https://jmeter.apache.org/download_jmeter.cgi)进行下载，而 Arch Linux 用户可以从 AUR 源下载 JMeter。推荐下载最新版本的 JMeter，JMeter 4 对于 HTTP Put 请求支持不太好(会忽略 Request Payload)。

### JMeter Plugins Manager

前面我们提到 JMeter 支持插件扩展，虽然 JMeter 默认安装的插件已满足基本需要了。但如果你遇到特别的使用场景(如 WebSocket)或者需要修改他人编写的 JMeter 测试文件，而其中用到一些非默认的插件。此时你就需要安装 JMeter Plugins Manager 来解决插件问题了。

安装 JMeter Plugins Manager 只需要下载其 JAR 文件并放到 JMeter 目录的 `lib/ext`。从[官方地址](https://jmeter-plugins.org/install/Install/)下载 JAR。

## 使用

### 定个小计划

在安装完 JMeter 后，我们以一个简单的例子快速掌握 JMeter 的使用吧～首先我们定个测试计划：使用 [GitHub API v3](https://developer.github.com/v3/) 获取你 [GitHub](https://github.com) 帐号 star 的特定仓库并在 [GitHub Actions](https://github.com/features/actions) 中集成我们 JMeter 测试。你说没有 GitHub 帐号？那现在注册一个吧～

下面使用的测试文件及代码可在 [jinhucheung/apache-jmeter-demo](https://github.com/jinhucheung/apache-jmeter-demo) 中获取。当然也可以 star 这个仓库进行下面操作～

### 获取接口

在准备测试接口前，我们需要对接口有一定的了解。为了实现我们前面定下的计划，我们先查看 [GitHub API v3](https://developer.github.com/v3/)。

获取我们帐号下 star 的仓库使用 `https://api.github.com/user/starred`，其要求我们提供帐号和密码，请求如下：

```sh
curl -X GET -u 'your_github_login:your_github_password' -H "Accept: application/vnd.github.v3+json" https://api.github.com/user/starred
```

其会返回一个 json 格式的我们 star 仓库列表。之后我们筛选出特定名称的仓库，从中获取到相应 URL 请求仓库的详细信息。

为了更全面展示 JMeter 的功能，我们增加点难度。我们首先获取帐号的信息，再根据返回得到帐号的 star url, 最后再请求仓库 URL。前面描述过程如下：

1. 获取授权用户的信息: `https://api.github.com/user/` - 获取到 star url
2. 获取授权用户已 star 的仓库列表: `https://api.github.com/users/${your_github_login}/starred` - 筛选出特定的仓库 URL
3. 获取特定仓库的信息: `https://api.github.com/repos/${repo_path}`

那我们开始吧～

### 获取用户信息

我们现在运行 JMeter，编写测试计划。

首先右击左侧目录树的 `Test Plan`，增加一个 `Thread Group`, 如下图：

![jmeter-add-thread-group](/images/posts/13_how_to_use_jmeter/add_thread_group.png)

其中 `Thread Group` 可以设置线程数量(用户数)、线程数量增长时间、循环次数等参数用于压力测试。

之后右击 `Thread Group` 添加 `Sampler/HTTP Request`，填写我们的第一个请求(获取授权用户的信息)，填入 HTTP 协议、请求域名、方法及路径等，如下图：

![jmeter-1-http-request](/images/posts/13_how_to_use_jmeter/1_http_request.png)

如果此时你点击了工具栏中的绿色箭头(运行图标)，JMeter 没有结果展示。让我们右击 `Thread Group` 添加 `Listener/View Results Tree`，此组件将会显示运行结果。

![jmeter-1-http-request-result](/images/posts/13_how_to_use_jmeter/1_http_request_result.png)

观察上面运行结果，由于少传了用户的帐号和密码参数，接口返回认证失败。通常我们会根据认证接口传递相应的参数，而 GitHub API 要求 Basic Auth，所以这里需要添加相应的组件。右击 `Thread Group` 添加 `Config Element/HTTP Authorization Manager`，添加用户信息，如下图：

![jmeter-http-authorization-manager](/images/posts/13_how_to_use_jmeter/http_authorization_manager.png)

再次点击运行，这次应该请求成功了～

然后我们可以为请求添加断言，JMeter HTTP Request 默认断言是返回码为 20x。右击第一个 `HTTP Request` 添加 `Assertions/JSON Assertion`，增加一个 JSON 断言，断言请求返回的用户名是我们传入的用户名，如下图：

![jmeter-1-http-request-assertion](/images/posts/13_how_to_use_jmeter/1_http_request_assertion.png)

为了查看断言结果，需要右击 `Thread Group` 添加 `Listener/Assertion Results`。

记得我们需要从请求返回信息中提取到 star url 么？查看请求返回信息的 `starred_url`，和我们期待的 url 有点出入？多了 `{/owner}{/repo}` 这部分。我们添加一个允许自定义脚本的提取组件来处理多余的 Path, 右击 `HTTP Request` 添加 `Post Processors/JSR223 PostProcessor`，选择你较为熟悉的脚本语言，并根据[官方文档](https://jmeter.apache.org/usermanual/functions.html)调用相关变量和函数：

![jmeter-1-http-request-postprocesser](/images/posts/13_how_to_use_jmeter/1_http_request_postprocesser.png)

上面的脚本获取到请求返回的 body，并将 body 解析成 JSON 以提取 `starred_url`，之后过滤掉 `starred_url` 多余的部分，最后赋值到 `starred_url` 变量中，为之后使用准备。

**Note**: 如果你正确设置了 HTTP Authorization Manager，执行时接口经常性返回验证失败，可以参考文档<sup>[[2]](#jmeter_basic_authentication)</sup>的做法。

### 参数变量化

可能你已经察觉到我们测试计划中存在一些参数重复使用，如 GitHub 帐号。这会带来一个麻烦，当我们需要修改某一参数时，就不得不排查所有测试请求了。一劳永逸的办法是添加一个变量组件，然后将参数替换成我们定义好的变量。

右击 `Thread Group` 添加 `Config Element/User Defined Variables` 组件，创建两个变量分别填入你的 GitHub 帐号和密码，如下图：

![jmeter-defined-variables](/images/posts/13_how_to_use_jmeter/defined_variables.png)

然后我们修改 `HTTP Request` 的断言组件，引用上一步定义好的变量替换相应的值，格式 `${variable_name}`：

![jmeter-use-variable-in-1-http-request](/images/posts/13_how_to_use_jmeter/use_variable_in_1_http_request.png)

如果需要将自定义脚本中使用的参数变量化，使用 `vars` 对象中的 `get` 方法引用变量：

![jmeter-use-variable-in-script](/images/posts/13_how_to_use_jmeter/use_variable_in_script.png)

**Note**: 当你需要一个唯一值的变量时，可以在变量值中使用 JMeter 的函数，如获取当前时间值 `${__time(,curTime)}`

我们还可以抽取请求信息到一个公共组件上，避免重复定义的问题，右击 `Thread Group` 添加 `Config Element/HTTP Request Defaults`，填入默认的请求信息：

![jmeter-http-request-defaults](/images/posts/13_how_to_use_jmeter/http_request_defaults.png)

之后我们便可以将 `HTTP Request` 上已经定义好的默认信息内容移除啦。

### 获取 star 仓库列表

前面我们获取到了用户 star 仓库的 url 并将其赋值到 `starred_url` 变量中。现在我们再添加多个 `HTTP Request`，设置请求方法为 `GET`, 请求路径填入 `${starred_url}`：

![jmeter-2-http-request](/images/posts/13_how_to_use_jmeter/2_http_request.png)

获取 star 仓库请求跟之前的有点不同，如果请求头没有 `User-Agent` 的话，获取 star 仓库请求将会返回空的 response body。右击 `Thread Group` 添加 `Config Element/HTTP Header Manager`，增加 `User-Agent` 项，值为一个合法的 `User-Agent`, 如 `Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/80.0.3987.122 Safari/537.36`。

![jmeter-http-header-manager](/images/posts/13_how_to_use_jmeter/http_header_manager.png)

现在我们为刚添加的 `HTTP Request` 增加一个断言组件，右击添加 `Assertions/JSON Assertion`。观察请求返回的 body，在 `JSON Assertion` 的 `Assert JSON Path exists` 上输入需要断言的值，如列表第一个仓库的 url 为 `$.[0].url`，那要获取特定名称的仓库 url 呢？

那我们只需要筛选出指定条件的仓库即可，`JSON Assertion` 组件筛选数组表达式，如 `$.[?(@.full_name=="jinhucheung/letscertbot")].url` 获取名称为 `jinhucheung/letscertbot` 的仓库。当然我们通常会定义变量去做筛选，将表达式中的具体仓库名替换为 `${starred_repo_full_name}`, 如 `$.[?(@.full_name=="${starred_repo_full_name}")].url`，然后在 `User Defined Variables` 组件上添加 `starred_repo_full_name` 变量，并赋予具体的仓库名。

等等，断言还没编写完成哦～ 勾选上 `Additionally assert value` 和 `Match as regular expression`，然后在 `Expected Value` 输入 `.+`，表示断言存在特定仓库的 url。

![jmeter-2-http-request-assertion](/images/posts/13_how_to_use_jmeter/2_http_request_assertion.png)

最后提取特定仓库的 URL 待往下使用，添加 `Post Processors/JSON Extractor`，输入前面断言的 JSON 表达式和变量名 `starred_repo_full_url`，如下：

![jmeter-2-http-request-postprocesser](/images/posts/13_how_to_use_jmeter/2_http_request_postprocesser.png)

### 获取特定仓库信息

到这一步，获取特定仓库信息已经十分简单了，添加 `HTTP Request` 请求 `starred_repo_full_url`，并断言请求返回的仓库 `full_name` 与我们定义的 `starred_repo_full_name` 一致。

![jmeter-3-http-request](/images/posts/13_how_to_use_jmeter/3_http_request.png)

![jmeter-3-http-request-assertion](/images/posts/13_how_to_use_jmeter/3_http_request_assertion.png)

### 录制请求

你是否觉得编写请求很麻烦呢？需要填入请求路径及一堆参数。其实 JMeter 支持录制请求，它可以将我们操作的请求一个个拷贝下来，生成相应的 `HTTP Request` 组件。

要开启 JMeter 录制请求，首先需要添加 `HTTP(S) Test Script Recorder` 组件，右击 `Test Plan` 在 `Non-Test Elements` 菜单中添加。

选择 `Target Controller` 为前面添加的 `Thread Group`，点击 `Start`。Jmeter 将提示生成了一个临时的 SSL 证书，用于录制 HTTPS 请求。点击 `确定` 后，`HTTP(S) Test Script Recorder` 组件将启动在默认的 `8888` 端口上。

![jmeter-http-test-script-recorder](/images/posts/13_how_to_use_jmeter/http_test_script_recorder.png)

然后我们打开操作系统的网络设置（可以设置浏览器的网络代理），添加 HTTP / HTTPS 代理至本地的 `8888` 端口，如下：

![jmeter-network-proxy](/images/posts/13_how_to_use_jmeter/network_proxy.png)

如果是要录制 HTTP 请求，到这里就可以愉快地工作啦。而要录制 HTTPS 请求，我们仍需要折腾一下。

将启动 `HTTP(S) Test Script Recorder` 时生成的临时 SSL 证书 `ApacheJMeterTemporaryRootCA.crt` 添加到浏览器可信任证书中。

官方文档<sup>[[3]](#jmeter_proxy_step_by_step)</sup>说 `ApacheJMeterTemporaryRootCA.crt` 在 `JMETER_HOME/bin`，而我通过 `sudo find / -name "ApacheJMeterTemporaryRootCA.*"` 发现其在 `/tmp` 目录下，具体情况可能跟你当前的操作系统有关。

然后我们将 `ApacheJMeterTemporaryRootCA.crt` 证书上传至浏览器。以 Chrome 为例，打开 `Settings`，进入 `Manage certificates`，然后打开 `Authorities` Tab，点击 `Import`并上传 `ApacheJMeterTemporaryRootCA.crt`，最后勾选上所有 `Trust Settings`。

![jmeter-brower-certificates-settings](/images/posts/13_how_to_use_jmeter/brower_certificates_settings.png)

### 集成 GitHub Actions

通过上面操作，你已经可以将 JMeter 运用于日常工作上啦。如果你还想了解 JMeter 与 CI 集成，那 Let's Go~

GitHub Actions 是 GitHub 官方推出的工作流特性，支持 CI/CD。你可以在[官方文档](https://help.github.com/en/actions)上进行详细了解。

现在让我们创建一个 GitHub 仓库，上传前面的测试计划文件，名称为 `test_plan.jmx`

然后添加 `.github/workflows/test.yml` 文件，输入以下内容：

```yml
name: JMeter Test

on: push

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v2
      - name: Mkdir test result directory
        run: mkdir testresults
      - name: Run test plan
        uses: docker://justb4/jmeter:latest
        with:
          args: -n -t ./test_plan.jmx -l ./testresults.log -e -o ./testresults
      - name: Upload test result directory
        uses: actions/upload-artifact@v1
        with:
          name: test_results_directory
          path: testresults
      - name: Upload test result log
        uses: actions/upload-artifact@v1
        with:
          name: test_results_log
          path: testresults.log
```

上面定义的工作流首先获取了 GitHub 仓库，然后拉取 JMeter 容器执行测试计划，最后上传测试结果文件。

## 总结

使用 JMeter 进行接口测试是一项十分容易的工作，基本不用编写代码，让你可以关注于业务接口本身。最后一个小建议：因为接口间调用是有次序的，最好在命名请求组件时带上相应的编号，好定位请求。而如果处理请求参数繁重，十分推荐编写脚本进行处理，如 [jmeter_batch_replace_args](https://github.com/jinhucheung/shell-script-sample/tree/master/jmeter_batch_replace_args)。

## Q&A

1. 接口看似返回 JSON 格式，但 JMeter JSON 断言解析报错: `(code 65279 / 0xfeff)` ?
  这个问题是接口返回了 [BOM](https://en.wikipedia.org/wiki/Byte_order_mark), 我们可以过滤掉该字符，参考处理 [Getting ï»¿ character in JMeter Response](https://stackoverflow.com/questions/47512202/getting-%C3%AF-character-in-jmeter-response)

2. 在全局 `HTTP Header Manager` 定义了一个 `Content-Type` 常量(如 `application/json`), 在需要提交 format data 的请求中 `Content-Type` 被覆盖成 json?
  我们可以在具体请求定义一个 PreProcessor，移除默认的 `Content-Type`, 如 `sampler.getHeaderManager().removeHeaderNamed("Content-Type")`

## 参考

1. [JMeter入门教程](https://www.jianshu.com/p/0e4daecc8122)
2. <a name='jmeter_basic_authentication'></a> [JMETER BASIC AUTHENTICATION EXPLAINED](https://octoperf.com/blog/2018/04/24/jmeter-basic-authentication/)
3. <a name='jmeter_proxy_step_by_step'></a> [Apache JMeter HTTP(S) Test Script Recorder](https://jmeter.apache.org/usermanual/jmeter_proxy_step_by_step.html)
4. [Jmeter之https录制](https://cloud.tencent.com/developer/news/234697)
5. [How to fix ‘Certificate Import Error: The Private Key for this Client Certificate is missing or invalid' error in the import certificate file](https://stackoverflow.com/questions/55947983/how-to-fix-certificate-import-error-the-private-key-for-this-client-certificat)
6. [Gitlab CI with jmeter](https://gitlab.com/knative-examples/functions/blob/master/.gitlab-ci.yml#L22)