---
title: Java Web - 搭建开发环境
date: 2019-10-09 21:19:25
tags: [Java, Web, IDEA, Tomcat, Maven]
categories: [Web Backend]
---

出于工作原因，最近开始学习 Java Web，自己也有打算接触些 Ruby 以外的语言。

俗话说，欲善其事，先利其器～在开始 Java Web 学习前，我们当然要准备好趁手的 “兵器” 啦。

<!--more-->

## IDE

Java 社区流行的 IDE 有 [Eclipse](https://www.eclipse.org/)、[MyEclipse](https://www.genuitec.com/products/myeclipse/)、[IntelliJ IDEA](https://www.jetbrains.com/idea/)。考虑到对 Java Web 的集成和工具流行度，我选择了 IDEA。

IDEA 分为社区版和终极版，其中社区版免费，但比较起终极版，缺少很多实用功能，如 JavaEE 等，下面操作以终极版为例子。

### 安装 IDEA

访问 IDEA 官网地址(https://www.jetbrains.com/idea/download/) 进行下载安装， Archlinux 用户可执行下面命令直接安装:

```
$ yaourt -S intellij-idea-ultimate-edition
```

### 激活 IDEA

终极版 IDEA 需要激活码才能无限制使用，终极版一年的授权费可要 499 刀，表示买不起 TT 好在一些网友无私分享他们购买的激活码，让我们有机会使用，真的十分感谢～ 我使用了下面的激活码：

```
Y9MXSIF79G-eyJsaWNlbnNlSWQiOiJZOU1YU0lGNzlHIiwibGljZW5zZWVOYW1lIjoiSkJGYW1pbHkgQ2hpbmEiLCJhc3NpZ25lZU5hbWUiOiIiLCJhc3NpZ25lZUVtYWlsIjoiIiwibGljZW5zZVJlc3RyaWN0aW9uIjoiIiwiY2hlY2tDb25jdXJyZW50VXNlIjpmYWxzZSwicHJvZHVjdHMiOlt7ImNvZGUiOiJJSSIsImZhbGxiYWNrRGF0ZSI6IjIwMTktMDctMjYiLCJwYWlkVXBUbyI6IjIwMjAtMDctMjUifSx7ImNvZGUiOiJBQyIsImZhbGxiYWNrRGF0ZSI6IjIwMTktMDctMjYiLCJwYWlkVXBUbyI6IjIwMjAtMDctMjUifSx7ImNvZGUiOiJEUE4iLCJmYWxsYmFja0RhdGUiOiIyMDE5LTA3LTI2IiwicGFpZFVwVG8iOiIyMDIwLTA3LTI1In0seyJjb2RlIjoiUFMiLCJmYWxsYmFja0RhdGUiOiIyMDE5LTA3LTI2IiwicGFpZFVwVG8iOiIyMDIwLTA3LTI1In0seyJjb2RlIjoiR08iLCJmYWxsYmFja0RhdGUiOiIyMDE5LTA3LTI2IiwicGFpZFVwVG8iOiIyMDIwLTA3LTI1In0seyJjb2RlIjoiRE0iLCJmYWxsYmFja0RhdGUiOiIyMDE5LTA3LTI2IiwicGFpZFVwVG8iOiIyMDIwLTA3LTI1In0seyJjb2RlIjoiQ0wiLCJmYWxsYmFja0RhdGUiOiIyMDE5LTA3LTI2IiwicGFpZFVwVG8iOiIyMDIwLTA3LTI1In0seyJjb2RlIjoiUlMwIiwiZmFsbGJhY2tEYXRlIjoiMjAxOS0wNy0yNiIsInBhaWRVcFRvIjoiMjAyMC0wNy0yNSJ9LHsiY29kZSI6IlJDIiwiZmFsbGJhY2tEYXRlIjoiMjAxOS0wNy0yNiIsInBhaWRVcFRvIjoiMjAyMC0wNy0yNSJ9LHsiY29kZSI6IlJEIiwiZmFsbGJhY2tEYXRlIjoiMjAxOS0wNy0yNiIsInBhaWRVcFRvIjoiMjAyMC0wNy0yNSJ9LHsiY29kZSI6IlBDIiwiZmFsbGJhY2tEYXRlIjoiMjAxOS0wNy0yNiIsInBhaWRVcFRvIjoiMjAyMC0wNy0yNSJ9LHsiY29kZSI6IlJNIiwiZmFsbGJhY2tEYXRlIjoiMjAxOS0wNy0yNiIsInBhaWRVcFRvIjoiMjAyMC0wNy0yNSJ9LHsiY29kZSI6IldTIiwiZmFsbGJhY2tEYXRlIjoiMjAxOS0wNy0yNiIsInBhaWRVcFRvIjoiMjAyMC0wNy0yNSJ9LHsiY29kZSI6IkRCIiwiZmFsbGJhY2tEYXRlIjoiMjAxOS0wNy0yNiIsInBhaWRVcFRvIjoiMjAyMC0wNy0yNSJ9LHsiY29kZSI6IkRDIiwiZmFsbGJhY2tEYXRlIjoiMjAxOS0wNy0yNiIsInBhaWRVcFRvIjoiMjAyMC0wNy0yNSJ9LHsiY29kZSI6IlJTVSIsImZhbGxiYWNrRGF0ZSI6IjIwMTktMDctMjYiLCJwYWlkVXBUbyI6IjIwMjAtMDctMjUifV0sImhhc2giOiIxMzgzODYyOS8wIiwiZ3JhY2VQZXJpb2REYXlzIjo3LCJhdXRvUHJvbG9uZ2F0ZWQiOmZhbHNlLCJpc0F1dG9Qcm9sb25nYXRlZCI6ZmFsc2V9-rI4et6OSKLA4gvOzxtyp48SCWtjwsOSQBJittaw6BOVJOwVBz0p31wBWDFSdIogdRPKquk2BAou7N694entEn4/Db3Ol5uotDtUd2MHuo+BBu9QcwIoX3RTrnYLwJfTlEJfRH/3TF3WtkPGQZQQcw/23hsZzdC/WJY6tmvyTijIBScUsvIOxZ+8REbWbkTQx1KliliFyrMua7hit8LThzfffZloHciaHwUP9BjxEjU0qQi+yFacSXjxEZERJT25hZrMN+bqBxcn59/4UJBrITt8YpLIlydt0+6vMSWAMawMzKpeDEDInKy0XomauTIUfxS4sbw/dSyVdSrh+IuOc7g==-MIIElTCCAn2gAwIBAgIBCTANBgkqhkiG9w0BAQsFADAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBMB4XDTE4MTEwMTEyMjk0NloXDTIwMTEwMjEyMjk0NlowaDELMAkGA1UEBhMCQ1oxDjAMBgNVBAgMBU51c2xlMQ8wDQYDVQQHDAZQcmFndWUxGTAXBgNVBAoMEEpldEJyYWlucyBzLnIuby4xHTAbBgNVBAMMFHByb2QzeS1mcm9tLTIwMTgxMTAxMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxcQkq+zdxlR2mmRYBPzGbUNdMN6OaXiXzxIWtMEkrJMO/5oUfQJbLLuMSMK0QHFmaI37WShyxZcfRCidwXjot4zmNBKnlyHodDij/78TmVqFl8nOeD5+07B8VEaIu7c3E1N+e1doC6wht4I4+IEmtsPAdoaj5WCQVQbrI8KeT8M9VcBIWX7fD0fhexfg3ZRt0xqwMcXGNp3DdJHiO0rCdU+Itv7EmtnSVq9jBG1usMSFvMowR25mju2JcPFp1+I4ZI+FqgR8gyG8oiNDyNEoAbsR3lOpI7grUYSvkB/xVy/VoklPCK2h0f0GJxFjnye8NT1PAywoyl7RmiAVRE/EKwIDAQABo4GZMIGWMAkGA1UdEwQCMAAwHQYDVR0OBBYEFGEpG9oZGcfLMGNBkY7SgHiMGgTcMEgGA1UdIwRBMD+AFKOetkhnQhI2Qb1t4Lm0oFKLl/GzoRykGjAYMRYwFAYDVQQDDA1KZXRQcm9maWxlIENBggkA0myxg7KDeeEwEwYDVR0lBAwwCgYIKwYBBQUHAwEwCwYDVR0PBAQDAgWgMA0GCSqGSIb3DQEBCwUAA4ICAQAF8uc+YJOHHwOFcPzmbjcxNDuGoOUIP+2h1R75Lecswb7ru2LWWSUMtXVKQzChLNPn/72W0k+oI056tgiwuG7M49LXp4zQVlQnFmWU1wwGvVhq5R63Rpjx1zjGUhcXgayu7+9zMUW596Lbomsg8qVve6euqsrFicYkIIuUu4zYPndJwfe0YkS5nY72SHnNdbPhEnN8wcB2Kz+OIG0lih3yz5EqFhld03bGp222ZQCIghCTVL6QBNadGsiN/lWLl4JdR3lJkZzlpFdiHijoVRdWeSWqM4y0t23c92HXKrgppoSV18XMxrWVdoSM3nuMHwxGhFyde05OdDtLpCv+jlWf5REAHHA201pAU6bJSZINyHDUTB+Beo28rRXSwSh3OUIvYwKNVeoBY+KwOJ7WnuTCUq1meE6GkKc4D/cXmgpOyW/1SmBz3XjVIi/zprZ0zf3qH5mkphtg6ksjKgKjmx1cXfZAAX6wcDBNaCL+Ortep1Dh8xDUbqbBVNBL4jbiL3i3xsfNiyJgaZ5sX7i8tmStEpLbPwvHcByuf59qJhV/bZOl8KqJBETCDJcY6O2aqhTUy+9x93ThKs1GKrRPePrWPluud7ttlgtRveit/pcBrnQcXOl1rHq7ByB8CFAxNotRUYL9IF5n3wJOgkPojMy6jetQA5Ogc8Sm7RG6vg1yow==
```

## Web 容器

在 Java 中， Web 容器也称为 Servlet 容器，主要用于加载、管理 Servlet。下面我们以 Tomcat 为例，安装一个 Web 容器。

### 安装 Tomcat

下载并解压 Tomcat:

```
$ wget http://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-9/v9.0.26/bin/apache-tomcat-9.0.26.zip
$ unzip apache-tomcat-9.0.26.zip
```

将 Tomcat 移动到 ~/.local/shared 目录下:

```
$ mkdir -p ~/.local/shared
$ mv apache-tomcat-9.0.26.zip ~/.local/share
```

变更 `catalina.sh` 为可执行:

```
$  chmod a+x ~/.local/share/apache-tomcat-9.0.26/bin/catalina.sh
```

然后启动 Tomcat:

```
$ ~/.local/share/apache-tomcat-9.0.26/bin/catalina.sh start
```

最后访问 http://localhost:8080 查看 Tomcat 是否启动成功。

关闭 Tomcat:

```
$ ~/.local/share/apache-tomcat-9.0.26/bin/catalina.sh stop
```

当然我们也可以使用 linux 的包管理器安装 Tomcat ～

### IDEA 配置 Tomcat

首先打开 IDEA，在工具栏中添加运行配置，如下图所示:

![IDEA 添加运行配置](/images/posts/6_java_web_getting_started/idea_add_run_configurations.png)

然后在弹窗中添加 `Tomcat Server`，并配置前面安装的 Tomcat 路径，如下图所示:

![IDEA 添加 Tomcat Home](/images/posts/6_java_web_getting_started/idea_configure_tomcat_path.png)

如果是包管理器安装，可以通过 `systemctl status tomcat${tomcat_version}` 查看 Tomcat 所在。

配置完 Tomcat 后点击工具栏的绿色箭头运行，然后访问 http://localhost:8080 查看 Tomcat 是否正常。

**Note**: 如果运行 Tomcat 时出现权限问题，须确保运行 Tomcat 进程的用户拥有写入 IDEA 启动用户目录的权限。

## 项目构建工具

Java 项目构建工具有 [Ant](https://ant.apache.org/), [Maven](https://maven.apache.org/what-is-maven.html), [Gradle](https://gradle.org/)。 Ant 已经销声匿迹，Maven 至今在社区中还相当流行，Gradle 作为后起之秀，凭借其简约的配置文件，受到大多数用户的拥戴。

考虑到现在国内大部分项目仍使用 Maven 作为构建工具，我选择使用 Maven 构建项目。

### 安装 Maven

我直接使用 linux 包管理工具进行安装，以 Archlinux 为例:

```
$ pacman -S maven
```

当然也可以直接下载 Maven 官网提供的压缩包进行安装(https://maven.apache.org/install.html)。

查看 maven 是否安装成功:

```
$ mvn -v
```

### 配置阿里云镜像

Maven 的配置文件分为用户级与全局配置，它们分别是 `~/.m2/settings.xml` 和 `${MAVEN_HOME}/conf/settings.xml`。

在上面两个配置文件中一个加入以下配置，设置阿里云镜像:

```xml
  <mirrors>
    <mirror>
       <id>alimaven</id>
       <name>aliyun maven</name>
       <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
       <mirrorOf>central</mirrorOf>
    </mirror>
  </mirrors>
```

### IDEA 配置 Maven

在 IDEA 全局设置中搜索 Maven 设置，如果 IDEA 能找到 Maven Home，它会默认配置，没有的需要我们自己填入。Maven 设置如下图:

![IDEA Maven 配置](/images/posts/6_java_web_getting_started/idea_maven_configuration.png)

### IDEA 使用 Maven

在 IDEA 中我们可以选择使用 Maven 搭建新项目的骨架，也可以通过右键项目选择 `Add Frameworks Support` 来对已有的非 Maven 项目导入 Maven 骨架。

Maven 中依赖管理是一个很重要的模块，IDEA 在上给我们带来很大便利。只要我们打开依赖文件 `pom.xml`，按下 `ALT` + `Insert` 后，在弹窗中选择 `Dependency` 搜索要添加的依赖包，IDEA 会在添加后提示我们要下载依赖。

## 参考

1. [分享几个IntelliJ IDEA 2019激活码（破解码、注册码），亲测可用](https://www.jiweichengzhu.com/article/eb340e382d1d456c84a1d190db12755c)
2. [一文看懂web服务器、应用服务器、web容器、反向代理服务器区别与联系](https://www.cnblogs.com/vipyoumay/p/7455431.html)
3. [IntelliJ IDEA 普通java工程如何转为maven工程](https://blog.csdn.net/u010221213/article/details/51361641)