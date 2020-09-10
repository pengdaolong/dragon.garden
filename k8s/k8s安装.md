>   在 master 节点和 worker 节点都要执行
 #### 修改 hostname
`hostnamectl set-hostname your-new-host-name`
#### 查看修改结果
`hostnamectl status`
#### 设置 hostname 解析
`echo "127.0.0.1   $(hostname)" >> /etc/hosts`
### 卸载旧版本
`yum remove -y docker \
docker-client \
docker-client-latest \
docker-common \
docker-latest \
docker-latest-logrotate \
docker-logrotate \
docker-selinux \
docker-engine-selinux \
docker-engine`
### 设置 yum repository
`yum install -y yum-utils \
device-mapper-persistent-data \
lvm2`
`yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo`
### 安装并启动 docker
`yum install -y docker-ce-18.09.7 docker-ce-cli-18.09.7 containerd.io`
`systemctl enable docker`
`systemctl start docker`

### 关闭 防火墙
`systemctl stop firewalld`
`systemctl disable firewalld`

### 关闭 SeLinux
`setenforce 0`
`sed -i "s/SELINUX=enforcing/SELINUX=disabled/g" /etc/selinux/config`
### 关闭 swap
`swapoff -a`
`yes | cp /etc/fstab /etc/fstab_bak`
`cat /etc/fstab_bak |grep -v swap > /etc/fstab`

### 修改 /etc/sysctl.conf  
#### 如果有配置，则修改
`sed -i "s#^net.ipv4.ip_forward.*#net.ipv4.ip_forward=1#g"  /etc/sysctl.conf`
`sed -i "s#^net.bridge.bridge-nf-call-ip6tables.*#net.bridge.bridge-nf-call-ip6tables=1#g"  /etc/sysctl.conf`
`sed -i "s#^net.bridge.bridge-nf-call-iptables.*#net.bridge.bridge-nf-call-iptables=1#g"  /etc/sysctl.conf`
#### 可能没有，追加
`echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf`
`echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf`
`echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf`
#### 执行命令以应用
sysctl -p
### 配置K8S的yum源
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF 
```

### 卸载旧版本
yum remove -y kubelet kubeadm kubectl

### 安装kubelet、kubeadm、kubectl
yum install -y kubelet-1.16.0 kubeadm-1.16.0 kubectl-1.16.0

### 如果不修改，在添加 worker 节点时可能会碰到如下错误
###[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". 
### Please follow the guide at https://kubernetes.io/docs/setup/cri/
```
sed -i "s#^ExecStart=/usr/bin/dockerd.*#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --exec-opt native.cgroupdriver=systemd#g" /usr/lib/systemd/system/docker.service
```
## 设置 docker 镜像，提高 docker 镜像下载速度和稳定性
### 如果您访问 https://hub.docker.io 速度非常稳定，亦可以跳过这个步骤
`echo "{\"registry-mirrors\": [\"${http://f1361db2.m.daocloud.io}\"]}" | sudo tee /etc/docker/daemon.json`

## 重启 docker，并启动 kubelet
`systemctl daemon-reload`
`systemctl restart docker`
`systemctl enable kubelet && systemctl start kubelet`
##查看docker是否正常
`docker version`

> 只在 master 节点执行
# 查看完整配置选项 https://godoc.org/k8s.io/kubernetes/cmd/kubeadm/app/apis/kubeadm/v1beta2
`rm -f ./kubeadm-config.yaml`

> 关于初始化时用到的环境变量
> 
> APISERVER_NAME 不能是 master 的 hostname
> APISERVER_NAME 必须全为小写字母、数字、小数点，不能包含减号
> POD_SUBNET 所使用的网段不能与 master节点/worker节点 所在的网段重叠。该字段的取值为一个 CIDR 值，如果您对 CIDR 这个概念还不熟悉，请仍然执行 export POD_SUBNET=10.100.0.1/16 命令，不做修改

```
cat <<EOF > ./kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: v1.16.0
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers
controlPlaneEndpoint: "${APISERVER_NAME}:6443"
networking:
  serviceSubnet: "10.96.0.0/16"
  podSubnet: "${POD_SUBNET}"# 
  dnsDomain: "cluster.local"
EOF
```
# kubeadm init
# 根据您服务器网速的情况，您需要等候 3 - 10 分钟
kubeadm init --config=kubeadm-config.yaml --upload-certs

# 配置 kubectl
`rm -rf /root/.kube/`
`mkdir /root/.kube/`
`cp -i /etc/kubernetes/admin.conf /root/.kube/config`
# 配置网络（二选一即可）
## 安装fannel
`kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml `

## 安装 calico 网络插件
### 参考文档 https://docs.projectcalico.org/v3.8/getting-started/kubernetes/
`rm -f calico.yaml`
`wget https://docs.projectcalico.org/v3.8/manifests/calico.yaml`
`sed -i "s#192\.168\.0\.0/16#${POD_SUBNET}#" calico.yaml`
`kubectl apply -f calico.yaml`


