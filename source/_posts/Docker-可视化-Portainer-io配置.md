---
title: Docker 可视化 Portainer.io配置
date: 2018-09-13 10:38:57
tags: [Docker,容器]
categories: [Docker,容器技术]
---
### 1.下载镜像
```shell
docker search portainer

docker pull portainer/portainer
```
### 2.单机运行Portainer.io 
```shell
docker volume create portainer_data

docker run -d -p 9000:9000 --name portainer --restart always -v /etc/localtime:/etc/localtime -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
```
<!-- more -->
### 3.Swarm集群运行
```shell
docker volume create portainer_data

docker service create --name portainer --publish 9000:9000 --replicas=1 --constraint 'node.role == manager' --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock --mount type=volume,src=portainer_data,dst=/data portainer/portainer -H unix:///var/run/docker.sock
```

### 4.常见问题
在可视化界面添加docker端点的时候ping不通。
```shell
# 1.在需要被添加的机器上修改docker.service（centos 7）
vim /usr/lib/systemd/system/docker.service
# 2.在 `ExecStart`属性后面追加如下内容：
-H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock
```
### 5.参考文档
>1.https://portainer.io/install.html 

>2.https://blog.csdn.net/u011781521/article/details/80469804
