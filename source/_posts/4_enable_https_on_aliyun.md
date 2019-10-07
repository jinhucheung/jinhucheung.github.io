---
title: 记录一次在阿里云 ECS 上开启 HTTPS 的经历
date: 2019-09-25 21:29:26
tags: [SSL, HTTPS, 阿里云, 运维, 工作]
categories: [运维]
---

今天工作中遇到了为阿里云 ECS 配置子域名站点 HTTPS 证书的需求，期间遇到一些坑，将配置过程记录下来。

<!--more-->

## 申请 SSL 证书

在阿里云 [SSL 证书（应用安全）] 中为域名购买免费的 DV SSL 证书，注意每个帐号最多持有 20 个免费证书。

然后选择 NGINX SSL 证书下载至本地，并上传至相应服务器。

## 配置 NGINX

首先查看 NGINX 是否配置了 SSL 模块，执行以下命令:

```
$ nginx -V
```

查看是否含有 `http_ssl_module`，如果没有，服务器需要安装 `openssl` 依赖库，且为 NGINX 重新编译 `http_ssl_module` 模块。

将前面的证书文件添加到 NGINX 配置路径（如 `/etc/nginx/conf/`）的 ssl 目录下。在 NGINX 配置目录增加 `conf.d` 文件并新建目标域名的配置文件，如 `example.com.conf`。

写入 `example.com` 的配置，与下面类似:

```
server {
  listen 443 ssl;
  server_name example.com; # 你要部署的域名

  ssl_certificate /etc/nginx/conf/ssl/example.com.pem; # 公钥
  ssl_certificate_key /etc/nginx/conf/ssl/example.com.key; # 秘钥

  ssl_session_timeout  5m;
  ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;

  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;

  ssl_prefer_server_ciphers  on;

  location / {
    proxy_pass http://localhost:3000; # 应用
    proxy_set_header X-Forwarded-Proto https;
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}

server {
  listen      80;
  server_name example.com;
  return 301 https://$server_name$request_uri;
}

```

执行下面命令查看 NGINX 配置是否含有错误:

```
$ nginx -t
```

NGINX 重新加载配置文件:

```
$ nginx -s reload
```

## 设置阿里云 ECS 安全组

在阿里云 [云服务器 ECS] 上查看部署实例机器的 [安全组配置]，查看其 [入网方向规则] 中是否含有 `443` 端口。如果没有，则新建含有 `443` 入网规则的安全组，并将实例机器加入该安全组中。

## 服务器开放 443 端口

首先查看服务器是否监听 443 端口:

```
$ netstat -ntlp
```

如果没有 443 端口，请确保 NGINX 正常启动。

查看防火墙是否开放 443 端口:

```
$ firewall-cmd --list-ports
```

如果不含 443 端口，须要将防火墙 443 端口开放:

```
$ firewall-cmd --zone=public --add-port=443/tcp --permanent
$ firewall-cmd --reload
```

## Q&A

1. `telnet $server_ip 443` 返回 `No route to host` ?

须确保服务器开放 443 端口，且 443 端口在安全组入网规则中。

2. webSocket connection to failed: Error during WebSocket handshake: Unexpected response code: 404?

当客户端使用 ws 或者 wss 协议请求服务端进行 WebSocket 通信时，如果服务端返回连接拒绝或失败，很可能是 NGINX 的配置问题。WebSocket 工作在 HTTP 的 80 和 443 端口。

NGINX 作为反向代理服务，在接收到客户端的 WebSocket 连接请求时，须请求后端服务升级协议为 WebSocket，进行如下配置:

```
map $http_upgrade $connection_upgrade {
  default upgrade;
  ''      close;
}

server {
  ...

  location / {
    proxy_pass http://backend;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection $connection_upgrade;
  }
}
```

## 参考

1. [快速实现 HTTPS 和 HTTP 同时都可以访问 Ruby on Rails 应用](https://ruby-china.org/topics/35009)
2. [Nginx编译安装添加http_ssl_module模块](https://hunfan.top/2018/11/16/Nginx%E9%87%8D%E6%96%B0%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85%E6%B7%BB%E5%8A%A0http-ssl-module%E6%A8%A1%E5%9D%97/)
3. [Centos7 下防火墙开启 80 443 端口](https://my.oschina.net/macleo/blog/1816346)
4. [配置 Nginx 反向代理 WebSocket](https://www.hi-linux.com/posts/42176.html)
5. [Nginx map 使用详解](https://blog.51cto.com/tchuairen/2175525)