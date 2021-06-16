# K8s 部署

## 安装前准备

允许iptables检查桥接流量

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward=1
net.ipv4.tcp_tw_recycle=0
vm.swappiness=0
vm.overcommit_memory=1
fs.inotify.max_user_instances=8192
fs.inotify.max_user_watches=1048576
fs.file-max=52706963
fs.nr_open=52706963
net.ipv6.conf.all.disable_ipv6=1
net.netfilter.nf_conntrack_max=2310720
EOF
sudo sysctl --system
```

## 安装 kubeadm

可先用`apt-cache madison kubeadm`查看可安装版本

```bash
# 安装系统工具
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
#安装GPG证书
#由于gpg地址来自国外，没有翻墙就会失败
#curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
curl https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -

#网址访问不了
#cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
#deb https://apt.kubernetes.io/ kubernetes-xenial main
#EOF
# 写入软件源；注意：我们用系统代号为 bionic，但目前阿里云不支持，所以沿用 16.04 的 xenial
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
# 安装
sudo apt-get update  
sudo apt-get install -y kubelet=1.19.0-00 kubeadm=1.19.0-00 kubectl=1.19.0-00
sudo apt-mark hold kubelet kubeadm kubectl
# 设置 kubelet 自启动，并启动 kubelet
sudo systemctl start kubelet && sudo systemctl enable kubelet
```

## 关闭防火墙

```bash
sudo ufw disable
```

## 初始化

```bash
sudo kubeadm reset
rm -rf $HOME/.kube/config
```

## 主节点

**结点初始化**

```bash
sudo swapoff -a
sudo sed -i 's/.*swap.*/#&/' /etc/fstab
#此处不用阿里云镜像会因网络问题报错
sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=v1.19.0 --pod-network-cidr=10.244.0.1/16 > init.log
```

成功日志,里面有token和discovery-token-ca-cert-hash

```bash
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.247.128:6443 --token wyb66m.1xl6ky6691kylrtq \
    --discovery-token-ca-cert-hash sha256:3f31ccf903b9a52d9764b32b9494f0369e481f4f25e62a3d54dcc00fc070c18e 
```

**要开始使用集群我们先运行以下命令**

```bash
mkdir mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**查看kubernetes版本**

```bash
chen@k8s-master01:~$ kubectl version
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.2", GitCommit:"f5743093fd1c663cb0cbc89748f730662345d44d", GitTreeState:"clean", BuildDate:"2020-09-16T13:41:02Z", GoVersion:"go1.15", Compiler:"gc", Platform:"linux/amd64"}
```



## 网络

### flannel

```bash
# 旧版
kubectl -n kube-system apply -f ./network/kube-flannel.yml
    
# 新版（镜像拉取需翻墙）
kubectl -n kube-system apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```

### canal

```bash
kubectl -n kube-system apply -f ./network/canal.yaml
```

## 其它节点

节点是你的工作负载（容器和 Pod 等）运行的地方。要将新节点添加到集群，请对每台计算机执行以下操作：

- SSH 到机器
- 成为 root （例如 `sudo su -`）
- 运行 `kubeadm init` 输出的命令。例如:

```bash
kubeadm join --token <token> <control-plane-host>:<control-plane-port> --discovery-token-ca-cert-hash sha256:<hash>
```

### 关闭swap

将/etc/fstab文件中所有设置为swap的设备关闭

防止开机自动挂载 swap 分区永久关闭：把/etc/fstab中的swap注释掉

```bash
sudo swapoff -a
sudo sed -i 's/.*swap.*/#&/' /etc/fstab
```

### 1. 已知令牌

```bash
sudo kubeadm join 192.168.1.4:6443 --token jr7313.urbx5iln1vn7ozwz --discovery-token-ca-cert-hash sha256:7e59da3a22ba82ebbc062c509b739f9d03174b1dad1509244bc2d24776d4aec7 --image-repository registry.aliyuncs.com/google_containers
```

### 2. 未知令牌

#### 在主节点创建 token

```bash
sudo kubeadm token create

#输出类似以下
7gzzsx.qgpsskyw1dpj7vxe
```

#### 查看 token

```bash
sudo kubeadm token list
```

#### 获取 CA 公钥的哈希值

如果你没有 `--discovery-token-ca-cert-hash` 的值，则可以通过在主节点上执行以下命令链来获取它：

```bash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'

#输出类似以下
8cb2de97839780a412b93877f8507ad6c94f73add17d5d7058e91741c9d5ec78
```

### 一步获取加入集群命令

```bash
kubeadm token create --print-join-command
```



```bash
cwy@dell-PowerEdge-R720:~$ sudo kubeadm join 192.168.1.4:6443 --token jr7313.urbx5iln1vn7ozwz --discovery-token-ca-cert-hash sha256:7e59da3a22ba82ebbc062c509b739f9d03174b1dad1509244bc2d24776d4aec7 
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
	[WARNING SystemVerification]: this Docker version is not on the list of validated versions: 20.10.5. Latest validated version: 19.03
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...
[kubelet-check] Initial timeout of 40s passed.

Unfortunately, an error has occurred:
	timed out waiting for the condition

This error is likely caused by:
	- The kubelet is not running
	- The kubelet is unhealthy due to a misconfiguration of the node in some way (required cgroups disabled)

If you are on a systemd-powered system, you can try to troubleshoot the error with the following commands:
	- 'systemctl status kubelet'
	- 'journalctl -xeu kubelet'
error execution phase kubelet-start: timed out waiting for the condition
To see the stack trace of this error execute with --v=5 or higher

```

## 主节点有网关代理情况

### 主节点初始化

apiserver-cert-extra-sans 设置为网关IP

```
    sudo kubeadm init --apiserver-advertise-address=0.0.0.0 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=v1.19.0 --pod-network-cidr=10.244.0.1/16 --apiserver-cert-extra-sans=211.67.19.251 > init.log
```

### 主节点上修改 cluster-info ConfigMap

```
    kubectl edit cm cluster-info -oyaml -n kube-public
```

将 data.kubeconfig.clusters.cluster.server 改成 [https://网关socket](https://xn--socket-vo2jf441a/)





# k8s命令

## 显示与查看

```bash
#获取所有名字空间
kubectl get namespace 

#查看服务的集群IP以及端口
kubectl get svc -n namespace
#修改服务
kubectl edit svc your-deployment

#查看名字空间中的pod
kubectl get pod -n namespace
#查看名字空间中deployment信息
kubectl get deployment -n openfaas

#查看名字空间中的pod的详细信息
kubectl get pod -o wide -n openfaas

#pod副本扩容，deployment/后面跟想扩容的服务
kubectl scale --replicas = 3 deploymnet/gateway
#查看集群配置
kubectl -n kube-system get cm kubeadm-config -oyaml

#查看pod详细信息
kubectl describe pod pod-name -n namespace
#查看日志
kubectl log pod-name -c service
```



## 删除pod和deployment

```bash
kubectl delete deployment --all -n namespace
kubectl delte pod --all -n namespace
```

修改pod配置

```bash
kubectl edit pod pod-name -n namespace 
```



### Deployment

扩容

```bash
kubectl scale deployment name-deployment --replicas 10
```

更新镜像

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.9.1
```

回滚

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

查看回滚状态

```bash
kubectl rollout status deployment/nginx-deployment

echo $?
#输出为0
```

查看历史版本

```bash
kubectl rollout history deployment/nginx-deployment
```


