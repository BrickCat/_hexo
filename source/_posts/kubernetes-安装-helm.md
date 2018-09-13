---
title: kubernetes 安装 helm
date: 2018-09-13 10:46:49
tags: [kubernetes,helm,容器,k8s]
categories: [容器技术,kubernetes,docker,k8s]
---
### 1. Helm 客户端安装
```shell
# 1.官网下载二进制安装包
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.9.1-linux-amd64.tar.gz

# 2.开始安装
tar -zxvf helm-2.9.0.tar.gz

# 3.移动到系统目录下
mv helm-2.9.0/helm /usr/local/bin/helm
```
### 2. Helm 服务端安装
```shell
# 1.下载国内tiller镜像
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.9.1

# 2.为每个节点安装 socat
yum install -y socat

# 3.使用Service Account安装
## 1.创建一个名为tiller的Service Account
kubectl create serviceaccount tiller --namespace kube-system

## 2.授予名为tiller的ServiceAccount集群管理员角色cluster-admin：
vim helm-rbac-config.yaml
### 加入如下内容：
apiVersion: rbac.authorization.k8s.io/v1beta1 
kind: ClusterRoleBinding 
metadata: 
  name: tiller 
roleRef: 
  apiGroup: rbac.authorization.k8s.io 
  kind: ClusterRole 
  name: cluster-admin 
subjects: 
- kind: ServiceAccount 
  name: tiller 
  namespace: kube-system
  
  ### 授予tiller集群管理员角色：
  kubectl create -f helm-rbac-config.yaml
 
 ## 3.安装服务端（tiller）
 ### 创建服务端
 helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.9.1  --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
 
 ### 创建TLS认证服务端，参考地址：https://github.com/gjmzj/kubeasz/blob/master/docs/guide/helm.md
 helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.9.1 --tiller-tls-cert /etc/kubernetes/ssl/tiller001.pem --tiller-tls-key /etc/kubernetes/ssl/tiller001-key.pem --tls-ca-cert /etc/kubernetes/ssl/ca.pem --tiller-namespace kube-system --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
```
### 4. 验证
```shell
 helm version
 ### 结果：
 Client: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
 Server: &version.Version{SemVer:"v2.9.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
```
### 5. Helm使用
#### 5.1. 更换仓库
```shell
# 1.移除原来的仓库
helm repo remove stable
# 2.添加新仓库
helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
# 3.更新仓库
helm repo update
```
#### 5.2. 查看仓库中charts
```shell
helm search
```
#### 5.3. 更新charts列表
```shell
helm repo update
```
#### 5.4.安装charts
Monocular是一个开源软件，用于管理kubernetes上以Helm Charts形式创建的服务，可以通过它的web页面来安装helm Charts
##### 5.4.1 安装Nginx Ingress controller，安装的k8s集群启用了RBAC，则一定要加rbac.create=true参数
```shell
helm install stable/nginx-ingress --set controller.hostNetwork=true，rbac.create=true
```
##### 5.4.2 安装Monocular
```shell
# 1.添加新的源
helm repo add monocular https://kubernetes-helm.github.io/monocular
# 2.安装
helm install monocular/monocular -f custom-repos.yaml

# custom-repos.yaml 内容
cat custom-repos.yaml

api:
  config:
    repos:
      - name: stable
        url: https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts
        source: https://github.com/kubernetes/charts/tree/master/stable
      - name: incubator
        url: https://aliacs-app-catalog.oss-cn-hangzhou.aliyuncs.com/charts-incubator
        source: https://github.com/kubernetes/charts/tree/master/incubator
      - name: monocular
        url: https://kubernetes-helm.github.io/monocular
        source: https://github.com/kubernetes-helm/monocular/tree/master/charts
```
##### 5.4.3 查看需要安装的values
```shell
helm inspect values {chart名}
```

#### 5.5 查看K8S中已安装的charts
```shell
helm list
```
#### 5.6 删除安装的charts
```shell
# 删除：helm delete xxx
```
### 6. 卸载Helm服务端
```shell
helm reset 或 helm reset --force
```
### 7. 参考地址：
> 1.https://github.com/gjmzj/kubeasz/blob/master/docs/guide/helm.md 
> 2.https://blog.csdn.net/wenwenxiong/article/details/79067054 
> 3.https://blog.csdn.net/luanpeng825485697/article/details/80873236
> 4.https://blog.csdn.net/qq_35959573/article/details/80885052


