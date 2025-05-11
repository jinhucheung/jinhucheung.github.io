---
title: AWS 介绍
date: 2022-11-18 15:00:00
tags: [AWS, Cloud Computing]
categories: [Web Backend]
---

# Cloud Computing

## 什么是云计算？

云计算通过互联网按需提供 IT 资源，采用按使用量付费的定价方式。云供应商提供技术服务，例如计算能力、存储、网络，而无需购买、拥有和维护物理数据中心及服务器。

简单说，它包含了物理数据中心的一些资源：计算、存储、网络、数据库、水电煤资源等等。

## 云计算的好处

1. 节省成本：大型规模经济中获益，减少运维投入资金和精力，将资本投入变成可变投入
2. 增加速度和灵活性：各种 IT 资源在云计算中进行统一管理，轻松使用
3. 动态调整资源：根据实际需求扩展或缩减资源量
4. 快速扩展业务到全球：借助云，业务能扩展新的地理区域，快速全局部署

## 云计算的分类

### 按服务类型分类

1. IaaS 基础架构即服务：提供网络、计算能力以及存储等功能，如 AWS、Azure、Aliyun
2. PaaS 平台即服务：提供程序部署运行环境，不需关注底层的基础架构资源，如 Heroku、Fly.io
3. SaaS 软件即服务：提供了一种完善的产品，如 Airhost、Gmail

![cloud services](https://user-images.githubusercontent.com/19590194/206862194-4b31d359-410c-42fe-90c9-d971049fb76d.png)

### 按部署类型分类

1. 公有云：第三方供应商为用户提供的云服务
2. 私有云：企业内部搭建的云服务
3. 混合云：公有云+私有云

# AWS

## 什么是 AWS ?

[Amazon Web Services](https://aws.amazon.com/cn/what-is-cloud-computing/?nc2=h_ql_le_int_cc) (AWS),  亚马逊在 2006 年发布的云计算服务，目前是全球最全面、应用最广泛的云平台，提供超过 200 项功能齐全的服务。

AWS 连续 12 年在云基础设施和平台服务 (CIPS) 魔力象限列领导者位置。

![CIPS](https://user-images.githubusercontent.com/19590194/206862232-00c5dac2-03f2-4b83-8816-51802e469e66.png)

## AWS 基础设施

目前 AWS 在全球各地拥有 27 个数据中心，分为亚太、北美、南美、欧洲/中东/非洲，每个数据中心至少两个可用区，共 87 个可用区，超过 410 个边缘站点

每个数据中心、可用区和 AWS 区域都通过专门打造的、高可用和低延迟的私有全球网络基础设施进行互连

![az](https://user-images.githubusercontent.com/19590194/206862251-0dde650c-b89c-4126-9ed0-962db0d7ad75.png)

## AWS 提供的服务

- 基础服务
    - 计算：EC2 弹性虚拟机、ECS 容器服务、Lambda 自动运行代码等
    - 存储：S3 对象存储、EFS 文件存储等
    - 数据库：RDS 关系数据库、ElastiCache 内存缓存等
    - 网络：VPC 虚拟网络、ELB 负载均衡器等
- 应用服务
    - 内容分发 CloudFront
    - 电子邮件 SES
    - 队列 SQS
    - …
- 工具
    - 部署：CodeCommit、CodeBuild、CodeDeploy、CodeArtifact、CodePipeline
    - 容器：ECR
    - 监控 CloudWatch
    - …