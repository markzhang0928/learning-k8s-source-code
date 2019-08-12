# step 1: 安装必要的一些系统工具
```
sudo apt-get update
sudo apt-get -y install apt-transport-https ca-certificates curl software-properties-common
```

# step 2: 安装GPG证书
```
curl -fsSL http://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg | sudo apt-key add -
```

# Step 3: 写入软件源信息
```
sudo add-apt-repository "deb [arch=amd64] http://mirrors.aliyun.com/docker-ce/linux/ubuntu $(lsb_release -cs) stable"
```

# Step 4: 更新并安装 Docker-CE
```
sudo apt-get -y update
sudo apt-get -y install docker-ce
```

# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
```
# apt-cache madison docker-ce
# docker-ce | 17.03.1~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
# docker-ce | 17.03.0~ce-0~ubuntu-xenial | http://mirrors.aliyun.com/docker-ce/linux/ubuntu xenial/stable amd64 Packages
```

# Step 2: 安装指定版本的Docker-CE: (VERSION 例如上面的 17.03.1~ce-0~ubuntu-xenial)
```
# apt-get -y install docker-ce=17.09.1~ce-0~ubuntu
```

--------------------------------
# Step 3: 安装
```
add-apt-repository "deb https://mirrors.aliyun.com/kubernetes/apt kubernetes-xenial main"
apt-get update &&
apt-get install -y kubernetes-cni=0.6.0-00 cri-tools=1.11.1-00 kubelet=1.11.6-00 kubeadm=1.11.6-00 kubectl=1.11.6-00 --allow-unauthenticated
```

### Step 4 因为墙的原因，有以下镜像需要手动下载
```
registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/kube-proxy-amd64:v1.11.6
registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/kube-controller-manager-amd64:v1.11.6
registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/kube-apiserver-amd64:v1.11.6
registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/kube-scheduler-amd64:v1.11.6
registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/coredns:1.1.3
registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/etcd-amd64:3.2.18
registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/pause:3.1


registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/coreos_flannel:v0.9.1
registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/calico_cni:v2.0.7
registry.cn-hangzhou.aliyuncs.com/huya_k8s_gcr_io/calico_node:v3.0.9

```

## 以下脚本用来修改 dockerhub域名和 namespace
```
#!/bin/bash

repositories=`docker images|grep registry.cn-hangzhou.aliyuncs.com > images.txt`
## echo $repositories

while read images; do
echo ${images}
image_name=`echo $images|awk '{print $1}'`
image_id=`echo $images|awk '{print $3}'`
image_tag=`echo $images|awk '{print $2}'`

echo ${image_name} ${image_id} ${image_tag}

if [ ${image_name##*/} == "coreos_flannel" ]; then
  docker tag $image_id quay.io/coreos/flannel:${image_tag}
elif [ ${image_name##*/} == "calico_cni" ]; then
  docker tag $image_id quay.io/calico/cni:${image_tag}
elif [ ${image_name##*/} == "calico_node" ]; then
  docker tag $image_id quay.io/calico/node:${image_tag}
else
  docker tag $image_id k8s.gcr.io/${image_name##*/}:${image_tag}
  echo k8s.gcr.io/${image_name##*/}:${image_tag}
fi

done < images.txt

```

```
kubeadm init --kubernetes-version=v1.11.6 --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=10.68.38.157
```

## slave 节点加入集群

```
kubeadm join 10.68.6.30:6443 --token jet02i.wdufhtw12n2jb5j8 --discovery-token-ca-cert-hash sha256:0389e4f2a25420ffb0526a6750436db011f5bbd1918a630b9b1ee8fc33d2ec64

```

## 安装网络插件canal
```
# kubectl apply -f  https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/canal/rbac.yaml
# kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/canal/canal.yaml
```



## 参考文档
1. https://juejin.im/entry/5c00d149e51d4522143b9c78
2. http://docs.kubernetes.org.cn/829.html
3. https://www.kubernetes.org.cn/3895.html
4. https://yq.aliyun.com/articles/656601
