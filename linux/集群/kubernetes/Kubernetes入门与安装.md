## Kubernetes入门与安装

kubernetes是简单来说就是一个集群，把多个主机组织成一个主机使用，并自动大规模部署运行容器，其他详细的概念基础见kubernets官方文件，这里介绍kubernetes的安装及使用，由于ubuntu的内核版本较新，所以这里使用Ubuntu16.04-server作为服务器

### 安装docker及k8s核心组件

- 安装docker，使用阿里云或者清华大学镜像站下载docker-ce

```shell
# step 1: 安装必要的一些系统工具
root@master:~# sudo apt-get update
root@master:~# sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
# step 2: 安装GPG证书
root@master:~# curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
# Step 3: 写入软件源信息
root@master:~# sudo add-apt-repository "deb [arch=amd64] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
# Step 4: 更新并安装Docker-CE
root@master:~# sudo apt-get -y update
root@master:~# sudo apt-get -y install docker-ce
## 输入docker info 如能看到信息则表示安装成功
```

- 安装k8s核心组件

```shell
root@master:~# apt-get update && apt-get install -y apt-transport-https
root@master:~# curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | apt-key add - 
root@master:~# cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
> deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
> EOF  
root@master:~# apt-get update
root@master:~# apt-get install -y kubernetes-cni=0.7.5-00 kubelet=1.14.0-00 kubeadm-1.14.0-00 kubectl-1.14.0-00
#这里选择指定版本是由于网络问题，我们无法访问k8s镜像节点从中获取kube-proxy等docker镜像及flannel网络相关组件
```

- 提前下载镜像

```shell
root@master:~# docker pull mirrorgooglecontainers/kube-apiserver:v1.14.0
root@master:~# docker pull mirrorgooglecontainers/kube-proxy:v1.14.0
root@master:~# docker pull mirrorgooglecontainers/kube-controller-manager:v1.14.0
root@master:~# docker pull mirrorgooglecontainers/kube-scheduler:v1.14.0
root@master:~# docker pull mirrorgooglecontainers/etcd:3.3.10
root@master:~# docker pull mirrorgooglecontainers/pause:3.1
root@master:~# docker pull coredns/coredns:1.3.1

# 并用docker tag将其打包为k8s.gcr.io
root@master:~# docker tag docker.io/mirrorgooglecontainers/kube-apiserver:v1.14.0 k8s.gcr.io/kube-apiserver:v1.14.0
root@master:~# docker tag docker.io/mirrorgooglecontainers/kube-controller-manager:v1.14.0 k8s.gcr.io/kube-controller-manager:v1.14.0
root@master:~# docker tag docker.io/mirrorgooglecontainers/kube-scheduler:v1.14.0 k8s.gcr.io/kube-scheduler:v1.14.0
root@master:~# docker tag docker.io/mirrorgooglecontainers/kube-proxy:v1.14.0 k8s.gcr.io/kube-proxy:v1.14.0
root@master:~# docker tag docker.io/mirrorgooglecontainers/pause:3.1 k8s.gcr.io/pause:3.1
root@master:~# docker tag docker.io/mirrorgooglecontainers/etcd:3.3.10 k8s.gcr.io/etcd:3.3.10
root@master:~# docker tag docker.io/coredns/coredns:1.3.1 k8s.gcr.io/coredns:1.3.1
```

- 配置启动项及Swap配置

  - 开机自启动

    ```shell
    root@master:~# systemctl enable docker
    root@master:~# systemctl enable kubelet
    ```

  - [关闭Swap分区或配置忽略Swap错误](swapoff)

    ```sh
    #关闭分区
    root@master:~#  swapoff -a
    #配置忽略swap错误，ubuntu默认kubelet配置文件与centos不同，默认位于/etc/default/kubelet，通常需要自行创建
    root@master:~# vim /etc/default/kubelet
    KUBELET_EXTRA_ARGS="--fail-swap-on=false" #忽略
    ```

- 初始化master

  ```shell
  root@master:~# kubeadm init --kubernetes-version=1.14.0 --pod-network-cidr=10.244.0.0/16 --service-cidr=10.96.0.0/12 --ignore-preflight-errors=Swap
  ```

  看到如下信息表示初始化成功,保存最后一行ku beadm join 命令用于后续node节点加入集群

  ```
  Your Kubernetes control-plane has initialized successfully!
  
  To start using your cluster, you need to run the following as a regular user:
  
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
  
  You should now deploy a pod network to the cluster.
  Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
    https://kubernetes.io/docs/concepts/cluster-administration/addons/
  
  Then you can join any number of worker nodes by running the following on each as root:
  
  kubeadm join 192.168.158.130:6443 --token 4y5rm4.gb7e2gt23jdzzkn5 \
      --discovery-token-ca-cert-hash sha256:e36bf429c8577a872fa033a5c3f7f6a12b388d8022960a0d6f92951ee1a87e7d
  ```

  根据提示生成配置目录及复制配置文件

  ```shell
  root@master:~# mkdir -p $HOME/.kube
  root@master:~# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  root@master:~# chown $(id -u):$(id -g) $HOME/.kube/config
  ```

- 最后部署flannel网络支持

```shell
root@master:~# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
#PS raw.githubusercontent.com域名解析经常收到污染，需手动解析hosts,我用的IP地址为：199.232.68.133
#使用kubectl get pods -n kube-system,当看到flannel相关组件STATUS=Running时表示部署成功
NAME                             READY   STATUS    RESTARTS   AGE
coredns-fb8b8dccf-k55kk          1/1     Running   0          170m
coredns-fb8b8dccf-npwjl          1/1     Running   0          170m
etcd-master                      1/1     Running   0          169m
kube-apiserver-master            1/1     Running   0          169m
kube-controller-manager-master   1/1     Running   0          169m
kube-flannel-ds-amd64-4dbk8      1/1     Running   0          168m
kube-flannel-ds-amd64-pwgf9      1/1     Running   0          169m
kube-proxy-hmknj                 1/1     Running   0          170m
kube-proxy-vbg4r                 1/1     Running   0          168m
kube-scheduler-master            1/1     Running   0          169m
```

- flannel部署成功后查看节点

```shell
root@master:~# kubectl get nodes 
#看到如下信息时候表示主节点部署成功
root@master:~# kubectl get nodes
NAME     STATUS   ROLES    AGE    VERSION
master   Ready    master   172m   v1.14.0
```

- Node节点加入集群

  同样在node节点上安装，docker-ce，k8s组件，设置docker,kubelet开启自启动，且关闭Swap交换分区，后通过kubeadm来加入主节点

  ```shell
  root@master:~# kubeadm join 192.168.158.130:6443 --token 4y5rm4.gb7e2gt23jdzzkn5 \
      --discovery-token-ca-cert-hash sha256:e36bf429c8577a872fa033a5c3f7f6a12b388d8022960a0d6f92951ee1a87e7d
  ```

  

### 错误说明

- [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". 应该将docker的cgroupfs改为systemd

  ```shell
  root@master:~# vim /etc/docker/daemon.json
  #加入如下一行，注意JSON格式
  "exec-opts": ["native.cgroupdriver=systemd"]
  #重载配置
  root@master:~# systemctl daemon-reload
  root@master:~# systemctl restart docker
  ```

- [WARNING SystemVerification]: this Docker version is not on the list of validated versions: 19.03.12. Latest validated version: 18.09，警告信息，意思为当前docker版本太新，官方认证过的版本仅支持至18.09，可以忽略不计

- [WARNING Swap]: running with swap on is not supported. Please disable swap：<a href="swapoff">未关闭系统Swap分区</a>

  

  

