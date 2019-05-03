# 在Mac上安装Minikube 1.0


最近这几年，软件开发翻天覆地的变化，当个软件行业从业者，不学习新东西肯定是不行的。从微服务到容器化服务开发，其实并没有多长时间，随着Docker和Kubenetes的大热，还必须要跟上脚步才行，做一个程序员真不容易，做一个老程序员更难。

> Docker和Kubenetes是什么，我这里就不介绍了，比我了解这玩意的的人，大有人在，如果你想了解，请自我学习，本文结尾也推荐了《IBM微讲堂 Kubenetes》系列，一共十个视频，看一遍基本也就都懂了。

如果你开发微服务想控制和容器编排工具有互动，比如调用K8s的API之类，或者你想本Mac地装一个单机版本的K8s，我也走了一些弯路，没少浪费时间。K8s安装是一个不小的工程，但是我并不是一个很好的运维，也仅仅在开发层面了解K8s，所以装一个单机版本非常有必要，这里推荐你去安装`Minikube`，这就是一个单机版本的单节点K8s。其实本来就想一个安装记录，但是估计也有不少人会遇到安装的问题，那我就稍微写详细一点，帮助大家稍微少走一点弯路。

安装`Minikube`第一步，你先要越过`GFW`，至于什么是GFW怎么越过去这里就不详细说了。
`Minikube`需要你本地装有`VirtualBox`，推荐你下载一个比较新的版本，因为要在上面运行一个虚拟机来跑K8s的节点。

## 修改网络配置

我用的软件是`ShadowsocksX-NG`这个软件，早期版本并不支持代理，后期版本使用了privoxy实现了HTTP和Socks，但是本文只需要使用HTTP代理就好了。

首先修改进入`Performance`中，把`Advance`中的`Socks5`地址和`HTTP`中的监听地址都改为0.0,0,0就Ok了，这样你就能接收非本地的请求代理访问ss了。

## 安装

Mac下安装太容易了，别告诉我你没有`brew`。
```
brew cask install minikube
```
大功告成。

下载安装kubectl ，需要手工该权限，这是K8s的命令行控制工具
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.14.0/bin/darwin/amd64/kubectl
sudo mv kubectl /usr/local/bin/
chmod +x /usr/local/bin/kubectl
```
安装完Minikube 先测试下网络环境，下面命令正常说明代理服务没问题
```
curl -x 192.168.99.1:1087 http://baidu.com
```

## 启动
用下面命令，让K8s节点的docker，使用下面代理访问，insecure-registry详解见本文末尾扩展阅读
```
minikube start --docker-env HTTP_PROXY=http://192.168.99.1:1087 --docker-env HTTPS_PROXY=http://192.168.99.1:1087 --docker-env NO_PROXY=127.0.0.1/24 --insecure-registry=192.168.99.1:5000
```
见到下面log为启动成功。
```
😄  minikube v1.0.0 on darwin (amd64)
🤹  Downloading Kubernetes v1.14.0 images in the background ...
🔥  Creating virtualbox VM (CPUs=2, Memory=2048MB, Disk=20000MB) ...
📶  "minikube" IP address is 192.168.99.105
🐳  Configuring Docker as the container runtime ...
    ▪ env HTTP_PROXY=http://192.168.99.1:1087
    ▪ env HTTPS_PROXY=http://192.168.99.1:1087
    ▪ env NO_PROXY=127.0.0.1/24
🐳  Version of container runtime is 18.06.2-ce
⌛  Waiting for image downloads to complete ...
E0503 17:26:45.170139    5798 start.go:209] Error caching images:  Caching images for kubeadm: caching images: caching image /Users/freewolf/.minikube/cache/images/k8s.gcr.io/k8s-dns-dnsmasq-nanny-amd64_1.14.13: fetching remote image: Get https://k8s.gcr.io/v2/: dial tcp 74.125.203.82:443: i/o timeout
✨  Preparing Kubernetes environment ...
❌  Unable to load cached images: loading cached images: loading image /Users/freewolf/.minikube/cache/images/gcr.io/k8s-minikube/storage-provisioner_v1.8.1: stat /Users/freewolf/.minikube/cache/images/gcr.io/k8s-minikube/storage-provisioner_v1.8.1: no such file or directory
🚜  Pulling images required by Kubernetes v1.14.0 ...
🚀  Launching Kubernetes v1.14.0 using kubeadm ... 
⌛  Waiting for pods: apiserver proxy etcd scheduler controller dns
🔑  Configuring cluster permissions ...
🤔  Verifying component health .....
💗  kubectl is now configured to use "minikube"
🏄  Done! Thank you for using minikube!
```
中间的错误信息说明本地的cache中没有所需文件都要连接互联网拉取。中间时间很长需要20分钟，因为要下载1g的内容。

如果你想知道都在下载哪些内容，你可以再开一个命令行行，用下面三个命令查看：
1 进入K8s节点
2 查看当前镜像，你会看到镜像不断增多
3 查看进程 你可以看到当前在pull什么镜像
4 查看启动容器
```
mimikube ssh
docker images
ps -aux
docker ps
```

## 插件配置

我又装了一个监控，一个反向代理两个插件
```
minikube addons  enable ingress
✅  ingress was successfully enabled

minikube addons  enable ingress
✅  ingress was successfully enabled
```
查看插件详情
```
minikube addons list
- addon-manager: enabled
- dashboard: enabled
- default-storageclass: enabled
- efk: disabled
- freshpod: disabled
- gvisor: disabled
- heapster: disabled
- ingress: enabled
- logviewer: disabled
- metrics-server: enabled
- nvidia-driver-installer: disabled
- nvidia-gpu-device-plugin: disabled
- registry: disabled
- registry-creds: disabled
- storage-provisioner: enabled
- storage-provisioner-gluster: disabled
```

## 启动Dashboard

输入`minikube dashboard`，当所需镜像下载完毕后会自动打开网页
```
minikube dashboard
🔌  Enabling dashboard ...
🤔  Verifying dashboard health ...
🚀  Launching proxy ...
🤔  Verifying proxy health ...
🎉  Opening http://127.0.0.1:60708/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```

## 查看镜像情况
进入容器`minikube ssh`查看镜像安装情况，可以看到装的东西不少。
```
docker images
REPOSITORY                                                       TAG                 IMAGE ID            CREATED             SIZE
k8s.gcr.io/kube-proxy                                            v1.14.0             5cd54e388aba        5 weeks ago         82.1MB
k8s.gcr.io/kube-controller-manager                               v1.14.0             b95b1efa0436        5 weeks ago         158MB
k8s.gcr.io/kube-apiserver                                        v1.14.0             ecf910f40d6e        5 weeks ago         210MB
k8s.gcr.io/kube-scheduler                                        v1.14.0             00638a24688b        5 weeks ago         81.6MB
quay.io/kubernetes-ingress-controller/nginx-ingress-controller   0.23.0              42d47fe0c78f        2 months ago        591MB
k8s.gcr.io/kube-addon-manager                                    v9.0                119701e77cbc        3 months ago        83.1MB
k8s.gcr.io/coredns                                               1.3.1               eb516548c180        3 months ago        40.3MB
k8s.gcr.io/kubernetes-dashboard-amd64                            v1.10.1             f9aed6605b81        4 months ago        122MB
k8s.gcr.io/etcd                                                  3.3.10              2c4adeb21b4f        5 months ago        258MB
k8s.gcr.io/pause                                                 3.1                 da86e6ba6ca1        16 months ago       742kB
k8s.gcr.io/metrics-server-amd64                                  v0.2.1              9801395070f3        16 months ago       42.5MB
gcr.io/k8s-minikube/storage-provisioner                          v1.8.1              4689081edb10        18 months ago       80.8MB
gcr.io/google_containers/defaultbackend                          1.4                 846921f0fe0e        18 months ago       4.84MB
```

## 最后
啰啰嗦嗦说一堆，我自己都看不下去了，其实只要保证K8s的安装节点网络没问题就好了，网络没问题，如果有错误就通过`minikube delete`，删除当前虚拟机，然后再运行带参数的`start`命令就好了，镜像下载最好一次性完成，镜像列表本文也有，希望能对你有所帮助。


## 资源推荐
- https://github.com/kubernetes/minikube
- https://blog.hasura.io/sharing-a-local-registry-for-minikube-37c7240d0615/
- https://v.youku.com/v_show/id_XMzA5NzkxNzA1Ng==.html?spm=a2h0k.11417342.soresults.dtitle
