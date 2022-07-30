---
title: WithU Note
date: 2021-01-16 21:59:46
tags: [WithU, Web]
categories: [产品]
hidden: true
---

## 2021.01.16

稍微调浅了主题色(#6666FF) by 设计师 [你的小表贝]。 后面开始画个人设置页面原型，还需要了解 ActiveStorage 的使用。

## 2020.01.12

考虑了下 WithU 的帐号方案 —— accounts, users, spaces:
- 一个 account 下有多个 users: 一个 account 下可以添加多个 user, 这些被添加 user 可以不需要邮箱或手机验证，account 所有者可以管理这些添加的 users。当添加的 user 首次登录后可自己设置个人信息，不再被 account 管理
- 一个 user 可以创建或加入多个 spaces
- 注册时会生成 account 和 user

但第一版先不考虑这种，可能有些地方需要预留下。

## 2020.12.29 - 2021.01.11

avataaars 的头像挺讨喜的，发现在 Rails 中集成有些麻烦。有个 Gem 实现了相关功能，但它通过 JavaScript Bridge 去调用 avataaars React component, 个人不太喜欢这种做法。而且 avataaars 生成的头像无法区分性别，这可能会让男性用户的默认头像是女性头像。

在 GitHub 上搜索到 Pictogrify 这个 JavaScript 库，默认的头像还算丰富，也分男性、女性主题。这正是我想要的。但要将集成到 Rails 中还是有些麻烦。因为我需要在 Ruby Runtime 中调用它，我不想通过 JavaScript Bridge 这种方式。

Just do it! 于是自己实现了 Rpictogrify - Ruby Version 的 Pictogrify。

![pictogrify](https://camo.githubusercontent.com/9bb9f306dec702869b08a8410aa3f42d56a1d66d512804eb3b599872dd790422/68747470733a2f2f692e696d6775722e636f6d2f56375763726f582e706e67)

## 2020.12.28

完成了错误页面。WithU 默认头像需要偏社交一点的，Letter Avatar 有点 Geek 了。

## 2020.12.19 - 2020.12.27

引入了 Tailwind CSS 和 Alpine.js，Yep! 希望能愉快地写前端。实现了登录逻辑及顶部导航栏。

## 2020.12.18

启动项目 - 在 GitHub 上创建了 WithU 的仓库，提交了第一行代码。使用 Ruby on Rails 搭了项目骨架。

## 2020.12.17

在网上搜索了类似的平台，存在一些主打 family 的网站。但感觉这些网站是 10 年前的设计，自己没有使用的冲动。所以打算自己折腾一个，于是就有了开发 WithU 的想法。

然后，在墨刀上设计了登录和主页的原型。

![login](https://user-images.githubusercontent.com/19590194/104814686-49454500-584b-11eb-8697-bd16525d6caa.png)

hmmm..大概是这样的感觉。

## 2020.12.16

一个想法：记录与喜欢的人点点滴滴，发生过的事件、计划中的事情、游玩过的地方、彼此喜欢吃的东西。我需要有这样的一个东西能记录这些。