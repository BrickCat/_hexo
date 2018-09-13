---
title: Kuberbetes CI/CD 持续集成Spring Cloud
date: 2018-09-13 11:05:56
tags: [kubernetes,k8s,CI/CD,SpringCloud,Docker]
categories: [kubernetes,Docker,容器技术,CI/CD]
---
### 写在前面
在现在这个科技不断快速发展的时代，我们产品的快速迭代变得十分重要，每个公司都有自己的迭代计划，有的是一个礼拜一个版本迭代，有的是半个月，也有的是更加频繁的迭代，面对这些迭代我们如何快速的平滑的发布应用就产生了 CI/CD，即持续集成和持续部署。

本场 Chat 会从零开始教会大家如何将我们的应用持续的，分场景、环境的持续的部署到 Kubernetes 集群中，以及我们的应用如何在不同的场景下，平滑的升级，以及构建我们自己的 Docker 私有仓库、代码托管服务、Jenkins 构建服务。

本场 Chat 将学到如下内容：

1. Docker 私有仓库搭建，以及 Kubernetes 中使用私有仓库；
2. 使用 Docker 搭建代码托管服务（ GitLab）；
3. 搭建 Kubernetes 1.11.2版的的基础集群和可视化管理；
4. 搭建 Jenkins 构建服务，利用 Pipeline 对应用镜像编译发布；
5. 如何将以上服务串联起来组成一个完整的持续集成的流水线；
6. Jenkins 构建服务如何构建不同分支上的代码，并发布到不同的环境中；
7. 一些常见问题的解决办法。

我们先来看看实现这个持续集成的流水线：

![](https://images.gitbook.cn/af6c4350-ab88-11e8-84f3-15434a82342c)

1. 开发人员提交代码 Gitlab 远程触发 Jenkins 构建
2. Jenkins 构建镜像推送到私有仓库
3. Jenkins 将最新的镜像部署到 k8s 中
4. 将部署反馈发送给用户

由于GitChat版权限制，这篇实战教程不能免费发于个人博客中，这篇文章定价`5元`，总共77页。所以希望有需要的朋友可以去GitChat支持一下信客，信客在这里谢过各位大大了。

### 教程目录：

1. 实验环境概览</br>
2. 虚拟机和必要软件安装</br>
3. 搭建 Docker 私有仓库：Harbor</br>
4. 搭建代码托管服务：GitLab</br>
5. 搭建 Jenkins 服务</br>
6. 搭建 k8s 集群 </br>
7. 搭建完整的持续的流水线</br>
8. 常见问题和常用命令</br>
9. 效果预览

### GitChat地址

![](Kuberbetes-CI-CD-持续集成Spring-Cloud\微信图片_20180913111445.jpg)
