---
title: Centos 7 部署Kubernetes v1.11.1
date: 2018-09-13 10:42:47
tags: [kubernetes,k8s,Docker,容器]
categories: [kubernetes,k8s,Docker,容器技术]
---
### 1. 环境说明
系统 | IP | 名称
---|---|---
centos7 | 10.0.0.147 | k8s-master
centos7 | 10.0.0.146 | k8s-slave1
centos7 | 10.0.0.148 | k8s-slave2

### 2. 准备工作
#### 2.1 安装Docker
```shell 
#1.安装依赖包
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
<!-- more -->
#2.设置阿里云镜像源
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo

#3.重建 Yum 缓存
yum -y install epel-release
yum clean all
yum makecache

#4.安装Docker-ce
sudo yum install docker-ce

#5.启动 Docker-ce
sudo systemctl enable docker
sudo systemctl start docker

#6.Docker建立用户组
sudo groupadd docker
sudo usermod -aG docker $USER

```
配置[阿里云加速器](https://account.aliyun.com/login/login.htm?oauth_callback=https%3A%2F%2Fcr.console.aliyun.com%2F#/accelerator)
```shell
# 重新加载配置
systemctl daemon-reload
systemctl restart docker
```
#### 2.2 关闭防火墙和selinux
```shell
#1.关闭防火墙
systemctl  stop   firewalld  &&   systemctl  disable   firewalld

#2.关闭selinux
setenforce 0
#修改"SELINUX=disabled"为"SELINUX=disabled"
vim /etc/selinux/config

#3.关闭Swap
sudo swapoff -a
#要永久禁掉swap分区，打开如下文件注释掉swap那一行 
sudo vi /etc/fstab

#4.配置转发参数
cat <<EOF > /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
vm.swappiness=0
EOF
sysctl --system

#5.修改hostname
vim /etc/hosts
```
#### 2.3 安装kudeadm、kubectl、kubelet、kubernetes-cni
```shell
#1. 配置阿里云软件源
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
EOF

#2.重建Yum缓存
yum -y install epel-release
yum clean all
yum makecache

#3.安装 kudeadm
yum -y install kubelet kubeadm kubectl kubernetes-cni

#4.启用kubeadm服务，并配置开机启动
systemctl enable kubelet && systemctl start kubelet

#5.编写脚本下载需要的镜像（原镜像被墙）
#!/bin/bash
images=(kube-proxy-amd64:v1.11.0 kube-scheduler-amd64:v1.11.0 kube-controller-manager-amd64:v1.11.0 kube-apiserver-amd64:v1.11.0
etcd-amd64:3.2.18 coredns:1.1.3 pause-amd64:3.1 kubernetes-dashboard-amd64:v1.8.3 k8s-dns-sidecar-amd64:1.14.9 k8s-dns-kube-dns-amd64:1.14.9
k8s-dns-dnsmasq-nanny-amd64:1.14.9 )
for imageName in ${images[@]} ; do
docker pull keveon/$imageName
docker tag keveon/$imageName k8s.gcr.io/$imageName
docker rmi keveon/$imageName
done
# 个人新加的一句，V 1.11.0 必加
docker tag da86e6ba6ca1 k8s.gcr.io/pause:3.1

#6.修改脚本权限
chmod -R 777 ./xxx.sh
```
**注：这里我就遇到过一个坑，原作者是根据 1.10 来的，然后在 kubeadm init 执行的时候一直报错，说找不到镜像。之后镜像版本是下载对了，但还是在 `[init] this might take a minute or longer if the control plane images have to be pulled` 这一句卡住，在国外的 VPS 测试之后，发现多了一个 k8s.gcr.io/pause:3.1 镜像，他的 ID 其实与 pause-amd64:3.1 一样，然后加了一个新的 TAG 之后，正常部署**。

### 3. Master 安装kubernetes
```shell
#1.初始化 Master
kubeadm init --kubernetes-version=v1.11.0 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=0.0.0.0 --apiserver-cert-extra-sans={Master Ip},127.0.0.1,k8s-master
```
**--kubernetes-version -> 版本号；--pod-network-cidr -> POD 网络的 IP 段；--apiserver-advertise-address ->通过该 ip 地址向集群其他节点公布 api server 的信息，必须能够被其他节点访问； --apiserver-cert-extra-sans -> 指定自己内网地址和映射；**
```shell
#2.配置 kubectl 认证信息

export KUBECONFIG=/etc/kubernetes/admin.conf
# 如果你想持久化的话，直接执行以下命令【推荐】
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

#3. 查看初始化状态
kubectl  get  cs
#4.查看 Nodes 状态
kubectl  get nodes
#5. 记录 Token 和 SSh
kubeadm join --token <TOKEN> --discovery-token-ca-cert-hash sha256:<SSH>
```
### 4. 安装 Flannel 网络
```shell
#依次执行以下命令
mkdir -p /etc/cni/net.d/

cat <<EOF> /etc/cni/net.d/10-flannel.conf
{
“name”: “cbr0”,
“type”: “flannel”,
“delegate”: {
“isDefaultGateway”: true
}
}
EOF

mkdir /usr/share/oci-umount/oci-umount.d -p

mkdir /run/flannel/

cat <<EOF> /run/flannel/subnet.env
FLANNEL_NETWORK=10.244.0.0/16
FLANNEL_SUBNET=10.244.1.0/24
FLANNEL_MTU=1450
FLANNEL_IPMASQ=true
EOF
# 创建 flannel.yml:
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
rules:
  - apiGroups:
      - ""
    resources:
      - pods
    verbs:
      - get
  - apiGroups:
      - ""
    resources:
      - nodes
    verbs:
      - list
      - watch
  - apiGroups:
      - ""
    resources:
      - nodes/status
    verbs:
      - patch
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: flannel
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: flannel
subjects:
- kind: ServiceAccount
  name: flannel
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: flannel
  namespace: kube-system
---
kind: ConfigMap
apiVersion: v1
metadata:
  name: kube-flannel-cfg
  namespace: kube-system
  labels:
    tier: node
    app: flannel
data:
  cni-conf.json: |
    {
      "name": "cbr0",
      "type": "flannel",
      "delegate": {
        "isDefaultGateway": true
      }
    }
  net-conf.json: |
    {
      "Network": "10.244.0.0/16",
      "Backend": {
        "Type": "vxlan"
      }
    }
---
apiVersion: extensions/v1beta1
kind: DaemonSet
metadata:
  name: kube-flannel-ds
  namespace: kube-system
  labels:
    tier: node
    app: flannel
spec:
  template:
    metadata:
      labels:
        tier: node
        app: flannel
    spec:
      hostNetwork: true
      nodeSelector:
        beta.kubernetes.io/arch: amd64
      tolerations:
      - key: node-role.kubernetes.io/master
        operator: Exists
        effect: NoSchedule
      serviceAccountName: flannel
      initContainers:
      - name: install-cni
        image: quay.io/coreos/flannel:v0.9.1-amd64
        command:
        - cp
        args:
        - -f
        - /etc/kube-flannel/cni-conf.json
        - /etc/cni/net.d/10-flannel.conf
        volumeMounts:
        - name: cni
          mountPath: /etc/cni/net.d
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.9.1-amd64
        command: [ "/opt/bin/flanneld", "--ip-masq", "--kube-subnet-mgr" ]
        securityContext:
          privileged: true
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        volumeMounts:
        - name: run
          mountPath: /run
        - name: flannel-cfg
          mountPath: /etc/kube-flannel/
      volumes:
        - name: run
          hostPath:
            path: /run
        - name: cni
          hostPath:
            path: /etc/cni/net.d
        - name: flannel-cfg
          configMap:
            name: kube-flannel-cfg



#执行：
kubectl create -f ./flannel.yml

#查看节点信息
kubectl get nodes
```
### 5.Dashboard 配置
#### 5.1 安装 GIT
```shell
yum -y install git
```
#### 5.2 下载配置文件
Kuberentes 配置 DashBoard 也不简单，当然你可以使用官方的 dashboard 的 yaml 文件进行部署，也可以使用 Mr.Devin 这位博主所提供的修改版，避免踩坑。

地址在：https://github.com/gh-Devin/kubernetes-dashboard，将这些 Yaml 文件下载下来，在其目录下（注意在 Yaml 文件所在目录），执行以下命令：
```shell
kubectl  -n kube-system create -f .
```
启动 Dashboard 所需要的所有容器。

访问 MASTER 主机的 IP:30090，可以看到如下界面：
![image](https://images2018.cnblogs.com/blog/1203160/201807/1203160-20180712131120935-2084879289.png)
#### 5.3 配置权限
创建`dashboard-admin.yaml`文件
```shell
vim dashboard-admin.yaml
```
然后填充如下内容：
```shell
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
```
行如下命令启动容器:
```shell
kubectl -f ./dashboard-admin.yaml create
```
刷新页面
![image](https://images2018.cnblogs.com/blog/1203160/201807/1203160-20180712131136391-271894407.png)

### 6.配置 Slave 节点

#### 6.1 Slave 节点准备工作

> ### **重复第2步 准备工作**

#### 6.2 将子节点加入 Master

将初始化 Master 节点时保存的Token信息在各个Node 节点上执行
```shell
kubeadm join 10.0.0.147:6443 --token l9phbw.q2atway8la8o8t09 --discovery-token-ca-cert-hash sha256:8b3e15ff633a5394767f64885ac4525a25f608e46ba90dd1218e0676ff0c76bd
```
配置kubectl 命令
```shell
sudo cp /etc/kubernetes/kubelet.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/kubelet.conf
export KUBECONFIG=$HOME/kubelet.conf
```
==主意：==**必须配置kubectl命令，不然会出现以下错误:`The connection to the server localhost:8080 was refused - did you specify the right host or port?`(单独开终端也会出现）。**
#### 6.3 查看 Nodes
```shell
#在Master 节点上
kubectl  get  nodes
```
将会看见所有节点的状态为Ready。

### 7. 常见问题

1. 重新初始化Master 节点或者 各个子节点
```shell
kubeadm reset
```
Master需要重新执行**第3步及以下步骤**，子节点需要重新执行**第6步**。
2. 子节点加入Master时 Token 过期

默认token的有效期为24小时，当过期之后，该token就不可用了。解决方法如下：
```shell
#1. 重新生成Token
kubeadm token create

#2.获取ca证书sha256编码hash值
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

#3. 子节点加入集群
kubeadm join 10.0.0.147:6443 --token <New Token> --discovery-token-ca-cert-hash sha256:<New SSH>
```
3. The connection to the server localhost:8080 was refused - did you specify the right host or port?

在该节点机器上执行如下命令:
```shelll
sudo cp /etc/kubernetes/kubelet.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/kubelet.conf
export KUBECONFIG=$HOME/kubelet.conf
```
4.子节点加入不到Master，或者子节点访问不了 Api Server
> 初始化Master的时候需要指定 Api Server 能够被访问的IP 0.0.0.0 是任何IP等能反问。

### 8. 常用命令
kubectl get cs //查看集群信息

kubectl get componentstatuses //查看node节点组件状态

kubectl get svc -n kube-system //查看应用

kubectl cluster-info //查看集群信息

kubectl cluster-info dump //更详细的集群信息

kubectl describe --namespace kube-system service kubernetes-dashboard //详细服务信息

kubectl apply -f kube-apiserver.yaml   //更新kube-apiserver容器

kubectl delete -f /root/k8s/k8s_images/kubernetes-dashboard.yaml //删除应用

kubectl  delete service example-server //删除服务

systemctl  start kube-apiserver.service //启动服务。

kubectl get deployment --all-namespaces //启动的应用

kubectl get pod  -o wide  --all-namespaces //查看pod上跑哪些服务

kubectl get pod -o wide -n kube-system //查看应用在哪个node上

kubectl describe pod --namespace=kube-system //查看pod上活动信息

kubectl describe depoly kubernetes-dashboard -n kube-system

kubectl get depoly kubernetes-dashboard -n kube-system -o yaml

kubectl get service kubernetes-dashboard -n kube-system //查看应用

kubectl delete -f kubernetes-dashboard.yaml //删除应用

kubectl get events //查看事件

kubectl get rc/kubectl get svc

kubectl get namespace //获取namespace信息

kubectl delete node 节点名 //删除节点

kubectl create namespace {名称} //创建namespace

kubectl config set-context $(kubectl config current-context) --namespace={名称} //设置当前namespace

kubectl get service,deployment,pod

hostnamectl --static set-hostname  host名称

### 9.参考资料
1. https://www.cnblogs.com/myzony/p/9298783.html
2. https://www.cnblogs.com/yue-hong/p/8894033.html
3. https://www.cnblogs.com/menkeyi/p/7128809.html
