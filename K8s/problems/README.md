# k8s踩坑

#### kubeadm init 时出错

[WARNING Firewalld]: firewalld is active, please ensure ports [6443 10250] are open or your cluster may not function correctly

**解决方法：**

 开放这两个端口

```bash
sudo firewall-cmd --zone=public --add-port=6443/tcp --permanent
sudo firewall-cmd --zone=public --add-port=10250/tcp --permanent
sudo firewall-cmd --reload
```

然后重置kubeadm之前执行过的操作，在进行初始化

```bash
sudo kubeadm reset 
sudo systemctl start kubelet 
sudo kubeadm init --image-repository registry.aliyuncs.com/google_containers --kubernetes-version=v1.19.2 --pod-network-cidr=10.244.0.1/16 > init.log
```



#### kubeadm join 时连接超时出错

```bash
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
```



原因/etc/systemd/system/kubelet.service.d/10-kubeadm.conf 没有创建

解决方法：我从其他结点机器上将该文件复制过来



网上的踩坑日志：https://blog.csdn.net/u012570862/article/details/80150988



#### couldn't validate the identity of the API Server（8和32号机错误)

error execution phase preflight: couldn't validate the identity of the API Server: Get "https://192.168.1.4:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

也是join时产生的错误



error execution phase preflight: couldn't validate the identity of the API Server: Get "https://192.168.1.4:6443/api/v1/namespaces/kube-public/configmaps/cluster-info?timeout=10s": net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)

**解决方案**https://github.com/AlPasser/K8sDeploy关于网络问题

使用代理服务器的时候他将其他节点的信息收集到代理服务器，再由代理服务器发送到主节点，依靠iptables设置





####  不同 Node 上的 Pod 间无法通信

![image-20210511201039051](C:\Users\ChenWuyang\AppData\Roaming\Typora\typora-user-images\image-20210511201039051.png)

**解决方法**

在每个节点上执行（[参考链接](https://my.oschina.net/u/4275236/blog/3354231)）：

```
  sudo bash -c "iptables -P OUTPUT ACCEPT && iptables -P FORWARD ACCEPT && iptables -F && iptables -L -n"
```



```bash
(base) cwy@iec32:~$ sudo bash -c "iptables -P OUTPUT ACCEPT && iptables -P FORWARD ACCEPT && iptables -F && iptables -L -n"
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain DOCKER (0 references)
target     prot opt source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (0 references)
target     prot opt source               destination         

Chain DOCKER-ISOLATION-STAGE-2 (0 references)
target     prot opt source               destination         

Chain DOCKER-USER (0 references)
target     prot opt source               destination         

Chain KUBE-EXTERNAL-SERVICES (0 references)
target     prot opt source               destination         

Chain KUBE-FIREWALL (0 references)
target     prot opt source               destination         

Chain KUBE-FORWARD (0 references)
target     prot opt source               destination         

Chain KUBE-KUBELET-CANARY (0 references)
target     prot opt source               destination         

Chain KUBE-PROXY-CANARY (0 references)
target     prot opt source               destination         

Chain KUBE-SERVICES (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-DEFAULT (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-EGRESS (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-EGRESS-ACCEPT (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-EGRESS-CUSTOM (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-EGRESS-DEFAULT (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-INGRESS (0 references)
target     prot opt source               destination     
```



```bash
ws@IECv8:~$ sudo bash -c "iptables -P OUTPUT ACCEPT && iptables -P FORWARD ACCEPT && iptables -F && iptables -L -n"
Chain INPUT (policy ACCEPT)
target     prot opt source               destination         

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination         

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination         

Chain DOCKER (0 references)
target     prot opt source               destination         

Chain DOCKER-ISOLATION-STAGE-1 (0 references)
target     prot opt source               destination         

Chain DOCKER-ISOLATION-STAGE-2 (0 references)
target     prot opt source               destination         

Chain DOCKER-USER (0 references)
target     prot opt source               destination         

Chain KUBE-EXTERNAL-SERVICES (0 references)
target     prot opt source               destination         

Chain KUBE-FIREWALL (0 references)
target     prot opt source               destination         

Chain KUBE-FORWARD (0 references)
target     prot opt source               destination         

Chain KUBE-KUBELET-CANARY (0 references)
target     prot opt source               destination         

Chain KUBE-PROXY-CANARY (0 references)
target     prot opt source               destination         

Chain KUBE-SERVICES (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-DEFAULT (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-EGRESS (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-EGRESS-ACCEPT (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-EGRESS-CUSTOM (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-EGRESS-DEFAULT (0 references)
target     prot opt source               destination         

Chain WEAVE-NPC-INGRESS (0 references)
target     prot opt source               destination 
```





#### flannel一直CrashLoopBackOff

* 32号机问题

  I0512 02:28:31.694750       1 main.go:213] Could not find valid interface matching eno1: error looking up interface eno1: route ip+net: no such network interface
  E0512 02:28:31.694850       1 main.go:237] Failed to find interface to use that matches the interfaces and/or regexes provided

flanne设置的网卡名字为eno1，但32号机对应的网卡名为enp6s0，应该修改网卡





* 8号机问题

  Error from server: Get "https://222.20.95.240:10250/containerLogs/kube-system/kube-proxy-cvmhx/kube-proxy": dial tcp 222.20.95.240:10250: connect: no route to host

8号机的eno1网卡对应ip为222.20.95.240，但这个ip所有机器都ping不通，应该改成222.20.72.24