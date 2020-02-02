---
title: 个人 Linux 桌面工具清单
date: 2020-02-01 21:45:00
tags: [Desktop, Linux, ArchLinux, Manjaro]
categories: [Workstation]
---

本文记录了自己平时使用的工作环境，便于新环境重新部署。

<!--more-->

## 安装 Arch Linux

1. 下载镜像
2. 做启动盘 (dd)
3. 分区安装系统
4. 安装 gnome 桌面 (Manjaro 跳过)
5. 安装一堆驱动 (Archlinux)

## 基本设置

1. 为工作用户设置 sudoer: 注意工作用户 sudo 配置是否被覆盖 (sudo -l -U username 查看)
2. 启用 sshd: 根据情况是否开启 ssh 登录 root
3. 为工作用户生成 ssh 密钥
4. 连接网络并设置自动时间
5. 为 pacman 增加中国源
6. 安装 yaourt
7. 安装 dash to dock 并配置
8. 安装搜狗输入法

## 科学上网

1. 下载 v2ray 并配置
2. 开启 v2ray 服务并设置开机启动
3. 下载 proxychains-ng 并配置 socket5
4. 下载 chromium
5. 下载 SwitchyOmega 并配置: 导入[配置文件](/attachs/OmegaOptions.bak), 下载[规则清单文件](https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt)
6. 同步 Google 帐号
7. 登录 lastpass

## 效率工具

1. 安装 deepin-wine
2. 安装微信 deepin-wine-wechat
3. 安装企业微信 deepin-wxwork
4. 安装 xmind
5. 安装 wps
6. 安装 peek: 录 gif
7. 安装网易云

## 开发环境

### 基本

1. 安装 base-devel 依赖
2. 安装 terminator 并配置: 参考[文档](https://github.com/jinhucheung/m-terminal)
3. 安装 Oh My Zsh 并配置
4. 为 terminator 添加全局快捷键

### Ruby

1. 安装 rvm 并添加 ruby-china 源
2. 安装 ruby 并设置默认版本
3. gem 添加 ruby-china 源
4. 安装 bundler 并设置镜像源

**Note**: add script to zsh

```sh
[[ -s "$HOME/.rvm/scripts/rvm" ]] && . "$HOME/.rvm/scripts/rvm"
```

### Node

1. 安装 nvm 并设置淘宝源
2. 安装 node
3. npm 安装 yarn

### Java

1. pacman 安装 OpenJDK
2. 通过 archlinux-java 切换 java 版本

### Python

1. pacman 安装 python, python2

### VSCode

1. 安装 vscode
2. 安装插件：vscode-icons, Visual Studio IntelliCode, GitLens, Git History, Git Blame, ruby 等
3. 配置 vscode，如下：

```json
{
    "window.zoomLevel": 0,
    "editor.fontSize": 14,
    "terminal.external.linuxExec": "terminator",
    "editor.tabSize": 2,
    "window.zoomLevel": 1,
    "files.trimTrailingWhitespace": true,
    "explorer.confirmDelete": false,
    "editor.suggestSelection": "first",
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",
    "ruby.intellisense": "rubyLocate",
    "gitlens.advanced.messages": {
        "suppressLineUncommittedWarning": true
    },
    "emmetdeng.includeLanguages": {
        "erb": "html"
    },
    "typescript.updateImportsOnFileMove.enabled": "always",
    "python.linting.enabled": false
}
```

### 其他

1. 安装 git
2. 安装 nginx： 带 ssl, gzip 模块最好，并设置开机启动
3. 安装 redis 并设置开机启动
4. 安装 mariadb 并设置开机启动
5. 安装 dbeaver
6. 安装 docker / docker-compose

## 远程环境

如当前部署的机器为远程环境机，进行如下配置：

1. 远程环境机安装 nomachine 并启动 nxserver.service
2. 客户端同样安装 nomachine
3. 关闭自动熄屏
4. 关闭合盖熄屏，如下：

```sh
$ sed -i "s~#HandleLidSwitch=suspend~HandleLidSwitch=ignore~g" /etc/systemd/logind.conf
$ reboot
```