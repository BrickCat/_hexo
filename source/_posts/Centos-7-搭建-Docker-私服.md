---
title: Centos 7 搭建 Docker 私服
date: 2018-09-13 10:45:12
tags: [Docker,容器]
categories: [Docker,容器技术]
---
### 1.环境说明

系统 | Ip | Docker版本 | Registry版本
---|---|---|---
Centos 7 | 10.0.0.149 | 18.06.0-ce | 2.0

### 2. 安装 Docker-ce
```shell
#1.安装依赖包
sudo yum install -y yum-utils device-mapper-persistent-data lvm2

#2.设置阿里云镜像源
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#3.重建 Yum 缓存
yum -y install epel-release
yum clean all
yum makecache

#4.安装Docker-ce
sudo yum install docker-ce

# 5.安装指定版本
yum install docker-ce-17.09.0.ce -y
yum install -y --setopt=obsoletes=0 docker-ce-17.03.2.ce-1.el7.centos.x86_64 docker-ce-selinux-17.03.2.ce-1.el7.centos.noarch
#5.启动 Docker-ce
sudo systemctl enable docker
sudo systemctl start docker

#6.Docker建立用户组
sudo groupadd docker
sudo usermod -aG docker $USER
```
配置[阿里云加速器](https://account.aliyun.com/login/login.htm?oauth_callback=https%3A%2F%2Fcr.console.aliyun.com%2F#/accelerator)
```shell
# 7.重新加载配置
systemctl daemon-reload

```
### 3. 系统环境配置
```shell
#1.关闭防火墙
systemctl  stop   firewalld  &&   systemctl  disable   firewalld

#2.关闭selinux
setenforce 0
#修改"SELINUX=disabled"为"SELINUX=disabled"
vim /etc/selinux/config

#3.修改hostname
vim /etc/hosts
```
### 4.搭建私服
```shell
# 1.pull 官方registry镜像
docker pull registry

# 2. 添加 Volume
mkdir /docker/registry

# 3. 运行 registry
docker run -d -p 5000:5000 -v /docker/registry:/var/lib/registry --name docker-registry --restart always registry

# 4.查看运行中的容器
docker ps

# 5.添加Tag标记
# 拉取一个官方的Java镜像
docker pull java:8-jre
# 添加tag
docker tag java:8-jre docker-registry:5000/java:8-jre
# 查看镜像
docker images

# 6.推送镜像到私服
docker push docker-registry:5000/java:8-jre

# 7.查看是否推送成功
curl -XGET http://docker-registry:5000/v2/_catalog

# 8.获取某个镜像的标签列表
curl -XGET http://docker-registry:5000/v2/image_name/tags/list

```
### 5.常见问题
1.Get https://docker-registry:5000/v1/_ping: http: server gave HTTP response to HTTPS client
```shell
vim /etc/docker/daemon.json
# 加入如下内容：
{ "insecure-registries":["hub.k8s.com"] }
# 重启服务
systemctl restart docker
```
2.Error response from daemon: driver failed programming external connectivity on endpoint quirky_allen
```shell
# 重启服务
systemctl restart docker
```
