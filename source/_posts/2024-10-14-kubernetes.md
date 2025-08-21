---
title: kubernetes详解
date: 2024-10-14 10:31:30
tags: [后台开发]
categories: [Program, Web]
typora-root-url: 2024-10-14-kubernetes
---

# 一、前言和知识性说明

Kubernetes 是一个可移植、可扩展的开源平台，用于管理容器化的工作负载和服务，方便进行声明式配置和自动化。一般来讲，取中间8个字母转成8，kubernetes又叫k8s。

## 1. 和docker的关系

docker其实就是启动、运行、维护容器的一个工具，是基于单个容器的。k8s是类似于一个更高级别的管理容器的工具，可以动态检测、启动、调度多个容器，提供高可用、稳定的业务服务

## 2. k8s各个组件说明

<img src="2024-11-29-01.png" />

### 2.1. pod

Pod是Kubernetes中最小的部署单元。一个Pod可以包含一个或多个容器，这些容器共享存储、网络和配置。Pod通常用于运行一个单一的应用实例。多个容器在同一个Pod中运行时，它们可以通过`localhost`进行通信。pod是真实运行的一个个容器，里面会有以下环境变量

```
=> env
TEST_SERVICE_PORT_80_TCP_ADDR=10.99.198.195
KUBERNETES_SERVICE_PORT_HTTPS=443
KUBERNETES_SERVICE_PORT=443
HOSTNAME=test-dep-6cb67f4fbb-n8d78
TEST_SERVICE_PORT_80_TCP_PORT=80
TEST_SERVICE_PORT_80_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP=tcp://10.96.0.1:443
TEST_SERVICE_PORT=tcp://10.99.198.195:80
KUBERNETES_PORT_443_TCP_PROTO=tcp
KUBERNETES_PORT_443_TCP_ADDR=10.96.0.1
TEST_SERVICE_SERVICE_HOST=10.99.198.195
TEST_SERVICE_PORT_80_TCP=tcp://10.99.198.195:80
KUBERNETES_SERVICE_HOST=10.96.0.1
KUBERNETES_PORT=tcp://10.96.0.1:443
KUBERNETES_PORT_443_TCP_PORT=443
TEST_SERVICE_SERVICE_PORT=80
```

### 2.2. service 将pod对外暴露端口

Service是Kubernetes中用于定义一组Pod的访问策略的抽象。它提供了一个稳定的IP地址和DNS名称，供外部或内部客户端访问。Service通过标签选择器将流量路由到一组Pod上，即使这些Pod的IP地址发生变化。它可以实现负载均衡和服务发现。Service有多种类型，如ClusterIP（默认，集群内部访问）、NodePort（通过节点IP和端口访问）、LoadBalancer（使用云提供商的负载均衡器）等。

### 2.3. 管理pod的组件

#### 1) Deployment

Deployment是Kubernetes中用于管理Pod的控制器。它定义了Pod的期望状态，比如副本数量、更新策略等。通过Deployment，用户可以声明应用的期望状态，Kubernetes会自动创建和管理Pod以达到这个状态。它支持滚动更新、回滚等功能。Deployment会创建ReplicaSet来管理Pod的副本数量，确保系统中始终有指定数量的Pod在运行。

#### 2) StatefulSet

statefulset是用于管理有状态应用程序的副本。适合需要持久化存储和稳定网络标识的场景。提供稳定的、唯一的网络标识符和持久化存储。和deployment对比如下

| 特性           | Deployment                     | StatefulSet                        |
| -------------- | ------------------------------ | ---------------------------------- |
| **Pod 的命名** | 随机生成（如：`nginx-abc123`） | 有序且稳定（如：`web-0`, `web-1`） |
| **存储**       | 通常使用临时存储               | 支持持久化存储（PVC）              |
| **扩缩容**     | 可以随时扩缩容                 | 需要按顺序扩缩容                   |
| **网络标识**   | 不稳定，Pod 重启后 IP 可能变化 | 稳定，Pod 的 DNS 名称和 IP 不变    |
| **启动顺序**   | 无特定顺序                     | 按顺序启动和终止                   |

#### 3) DaemonSet

DaemonSet 是一种控制器，用于确保在集群中的每个节点上运行一个特定的 Pod 实例。DaemonSet 的主要用途是为每个节点提供服务或功能，例如日志收集、监控、网络代理等。

- 当你创建一个 DaemonSet 时，Kubernetes 会在每个节点上启动一个 Pod 实例。
- 如果有新节点加入集群，DaemonSet 会自动在这些新节点上启动 Pod。
- 如果节点被删除，DaemonSet 会自动清理该节点上的 Pod。

DaemonSet 可以通过节点选择器（Node Selector）、节点亲和性（Node Affinity）和污点/容忍（Taints and Tolerations）来控制在哪些节点上运行 Pod。更新 DaemonSet 时，Kubernetes 会逐个更新每个节点上的 Pod。删除 DaemonSet 会删除所有相关的 Pod。

#### 4) Job

Job 是一种用于管理一次性任务的 Kubernetes 资源。它确保指定数量的 Pods 成功终止。Job 会创建一个或多个 Pods 来执行任务，直到任务完成。Job 主要用于执行一次性任务，例如数据处理、备份等。如果 Pod 执行失败，Job 会自动重启 Pod 以确保任务完成。可以设置并发策略，控制同时运行的 Pod 数量。

#### 5) CronJob

CronJob 是基于时间调度的 Job。它允许用户按照指定的时间表定期执行 Job，类似于 Linux 系统中的 cron 任务。CronJob 可以按照指定的时间间隔（如每分钟、每天等）自动创建 Job。使用 Cron 表达式来定义调度时间。可以设置保留的成功和失败的 Job 数量。

CronJob是定时创建Job对象，Job才是创建pod使用的

### 2.4. pod配置相关

#### 1) PersistentVolume

持久化存储卷，由管理员预先配置的，代表了一个具体的存储设备（如NFS、iSCSI、云存储等）。PV的生命周期独立于使用它的Pod。即使Pod被删除，PV仍然存在，直到被显式释放或删除。PV包含了存储的详细信息，如存储类型、容量、访问模式等。单独使用PV是无法直接给Pod提供存储的。Pod需要通过PVC来请求存储，Kubernetes会根据PVC的要求找到合适的PV并进行绑定。只有在PVC和PV绑定后，Pod才能使用该存储。

#### 2) PersistentVolumeClaim

PVC是用户对存储资源的请求。用户可以指定所需的存储大小和访问模式。PVC被创建时，Kubernetes会寻找一个符合PVC要求的PV，并将它们绑定在一起。绑定后，PVC可以被Pod使用。如果集群配置了动态供应，PVC可以自动创建PV。PVC是对PV的请求，PVC和PV之间通过绑定关系连接。一个PVC只能绑定一个PV，但一个PV可以被多个PVC（在不同的访问模式下）绑定。

#### 3) ConfigMap

ConfigMap 是一种用于存储非机密的配置信息的 Kubernetes 资源。它允许用户将配置信息分离出容器镜像，使得应用程序的配置可以在不重新构建镜像的情况下进行修改。ConfigMap 可以存储键值对、文件或目录等形式的配置信息。

#### 4) Secret

secret和configmap差不多，就是值是加密的

### 2.4. Ingress

管理外部访问集群服务的方式。它提供了HTTP和HTTPS路由功能，可以将外部请求路由到集群内部的服务。

- 路由：Ingress可以根据请求的URL路径或主机名将流量路由到不同的服务。例如，可以将/api的请求路由到api-service，而将/web的请求路由到web-service。
- TLS终止：Ingress可以配置TLS，以便在传输层加密流量，提供安全的HTTPS访问。
- 负载均衡：Ingress控制器通常会实现负载均衡功能，将流量分发到后端服务的多个实例。
- 基于规则的流量管理：可以根据请求的特征（如HTTP头、查询参数等）定义复杂的路由规则。

一般最常用的是ingress-nginx，使用nginx来作为路由功能

### 2.5. 网络相关

#### 1) Endpoints

表示服务的网络端点。每个Service对象都会自动创建一个Endpoints对象，Endpoints对象包含了与该Service相关的Pod的IP地址和端口信息。

- 服务发现：Endpoints提供了服务的实际网络地址，Kubernetes中的Service通过Endpoints来找到后端Pod。
- 动态更新：当Pod的状态发生变化（如启动、停止或崩溃）时，Endpoints会自动更新，以反映当前可用的Pod。
- 多种类型的Endpoints：Endpoints可以用于ClusterIP、NodePort、LoadBalancer等多种类型的Service。

也可以在service里面不指定pod选择器，自己定义endpoint，让service帮助暴露到外部

#### 2) HostEndpoint

主要用于在集群中定义和管理主机的网络端点。它通常与网络策略和网络插件（如Cilium、Calico等）一起使用，以实现更细粒度的网络控制和安全性。HostEndpoint 是一个描述主机网络端点的对象，通常包括主机的IP地址、端口和其他相关信息。它可以用于标识集群外部的服务或主机。主要用于

- 网络策略：通过定义 HostEndpoint，可以在Kubernetes中实现对外部主机的网络访问控制。
- 服务发现：可以将外部服务的IP和端口信息纳入Kubernetes的服务发现机制中。
- 安全性：通过限制对特定主机的访问，可以增强集群的安全性。

#### 3) GlobalNetworkSet

主要用于定义一组全局可用的 IP 地址或 CIDR 范围。这一概念通常与网络插件（如 Calico）一起使用，以便在集群中实现更灵活和强大的网络策略。

- 全局可用性：GlobalNetworkSet 定义的 IP 地址或 CIDR 范围在整个 Kubernetes 集群中都是可用的。这意味着它们可以被任何命名空间中的网络策略引用。
- 简化网络策略管理：通过使用 GlobalNetworkSet，用户可以集中管理一组 IP 地址或 CIDR 范围，而不必在每个网络策略中重复定义。这有助于减少配置的复杂性和潜在的错误。
- 与网络策略结合使用：GlobalNetworkSet 可以与 Kubernetes 的网络策略（NetworkPolicy）结合使用，以便更精确地控制流量。例如，可以创建一个网络策略，允许来自某个 GlobalNetworkSet 中定义的 IP 地址的流量。

## 3. kubectl/kubeadm/kubelete

- kubectl是一个命令行工具，用于和k8s的api server进行交互使用，类似于docker命令。二进制里面把鉴权，参数按照k8s的标准整理好，发送给api server拿到返回展示。
- kubeadm也是一个命令行工具，用于简化 Kubernetes 集群的安装和管理。提供快速、简单的方式来初始化 Kubernetes 控制平面。支持集群的升级和配置。生成必要的证书和配置文件，帮助用户快速搭建集群。
- kubelete是一个后台服务，用于监控容器的状态，确保它们按照预期运行。与 Kubernetes API 服务器通信，报告节点和容器的状态。处理 Pod 的创建、更新和删除等操作。

## 4. k8s下的dns说明

k8s下有一个核心组件是coredns

```shell
=> kubectl get all -n kube-system -o wide
NAME                         READY   STATUS    RESTARTS        AGE     IP                NODE         NOMINATED NODE   READINESS GATES
...
pod/coredns-86966648-crrt5   1/1     Running   9 (4d12h ago)   4d18h   192.168.235.195   k8s-master   <none>           <none>
pod/coredns-86966648-sg9n8   1/1     Running   0               4d18h   192.168.235.194   k8s-master   <none>           <none>

NAME               TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE     SELECTOR
...
service/kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   4d18h   k8s-app=kube-dns

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES                                      SELECTOR
...
deployment.apps/coredns  2/2     2            2           4d18h   coredns      xxxxxxx/google_containers/coredns:v1.10.1   k8s-app=kube-dns

NAME                                DESIRED   CURRENT   READY   AGE     CONTAINERS  IMAGES
replicaset.apps/coredns-86966648    2         2         2       4d18h   coredns     xxxxxx/google_containers/coredns:v1.10.1   k8s-app=kube-dns,pod-template-hash=86966648
```

可以看到coredns会启动一个service在集群内部可以访问，使用clusterip访问进行dns解析，随便新建一个pod可以查看一下底层的`/etc/resolv.conf`

```shell
=> kubectl exec test-7d94ddcc6f-6xc48 -- cat /etc/resolv.conf
nameserver 10.96.0.10
search default.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
```

k8s会对每一个pod上面都添加dns服务到coredns里面，也就是所有pod里面解析dns都会交给coredns进行处理。下面是coredns对于不同类型组件的域名的处理

| 组件             | 域名                                         | 响应结果                 |
| ---------------- | -------------------------------------------- | ------------------------ |
| pod              | <pod_name>.<namespace>.pod.cluster.local     | pod自己的ip              |
| service          | <service_name>.<namespace>.svc.cluster.local | service的clusterip       |
| Headless Service | <service_name>.<namespace>.svc.cluster.local | service对应的pod列表和ip |

### 4.1. Headless Service

请求域名输出如下，对应的是pod的ip列表

```
=> nslookup test.default.svc.cluster.local 10.96.0.10
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   test.default.svc.cluster.local
Address: 192.168.140.74
Name:   test.default.svc.cluster.local
Address: 192.168.235.242
Name:   test.default.svc.cluster.local
Address: 192.168.109.71
Name:   test.default.svc.cluster.local
Address: 192.168.109.70
```

## 5. istio的原理

<img src="2025-01-02-01.svg"/>

# 二、安装

## 1. minikube安装k8s虚拟机单节点集群

minikube是一个轻量级的k8s单节点集群，部署在设备的虚拟化环境，可以在本地环境搭建起来进行k8s的开发和测试

### 1.1. 选项解释

- `-d, --driver=''`: virtualbox, kvm2, qemu2, qemu, vmware, none, docker, podman, ssh (defaults to auto-detect)
- `--image-mirror-country=''`: 指定镜像的国家代码，中国就设置成cn，会使用`registry.cn-hangzhou.aliyuncs.com/google_containers`作为镜像下载地址
- `--base-image='xxx'`: 给`docker/podman`驱动使用的基础镜像，默认是`'gcr.io/k8s-minikube/kicbase:v0.0.45@sha256:81df288595202a317b1a4dc2506ca2e4ed5f22373c19a441b88cfbf4b9867c85'`，可以自己修改
- `--binary-mirror=''`: 下载kubectl, kubelet, kubeadm等二进制的地址

### 1.2. 标准安装（需要翻墙）

```shell
minikube start --driver='docker'
```

### 1.3. 国内镜像安装

```shell
minikube start --image-mirror-country='cn' --base-image='registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.45'
```

指定`--image-mirror-country='cn'`会自动选择镜像`registry.cn-hangzhou.aliyuncs.com/google_containers`，但是kicbase会下载`registry.cn-hangzhou.aliyuncs.com/google_containers/k8s-minikube/kicbase:v0.0.45`，这个下不下来，所以需要指定一下`--base-image='registry.cn-hangzhou.aliyuncs.com/google_containers/kicbase:v0.0.45'`

### 1.4. 内网镜像安装

- 需要代理`registry.cn-hangzhou.aliyuncs.com`到`aaa/bbb`
- 代理`https://dl.k8s.io/`到`ccc/ddd`

```shell
minikube start --image-repository aaa/bbb/google_containers --base-image='aaa/bbb/google_containers/kicbase:v0.0.45' --binary-mirror='http://ccc/ddd'
```

下不下来还可能是默认的镜像地址不太对，虽然指定了`--image-repository aaa/bbb/google_containers`但是下面几个镜像下载存在问题

| minikube默认拉取                                              | 实际能拉下来                                     |
| ------------------------------------------------------------- | ------------------------------------------------ |
| aaa/bbb/google_containers/k8s-minikube/storage-provisioner:v5 | aaa/bbb/google_containers/storage-provisioner:v5 |
| aaa/bbb/google_containers/coredns/coredns:v1.11.1             | aaa/bbb/google_containers/coredns:v1.11.1        |

针对这些能看到minikube报错拉取失败，这个时候按照下面方式进行处理

```shell
# 用docker拉取能拉下来的镜像
=> docker pull aaa/bbb/google_containers/storage-provisioner:v5
# 打tag修改镜像路径
=> docker tag aaa/bbb/google_containers/storage-provisioner:v5 aaa/bbb/google_containers/k8s-minikube/storage-provisioner:v5
# 让minikube加载到缓存里面
=> minikube image load aaa/bbb/google_containers/k8s-minikube/storage-provisioner:v5
```

特殊处理后就可以拉取了。对于`--binary-mirror`就是一些二进制需要拉取，但是默认是`https://dl.k8s.io`，要改一下。

### 1.5. minikube缓存路径

minikube 内置了将下载的资源缓存到 $MINIKUBE_HOME/cache 的支持。以下是重要的文件位置

- `~/.minikube/cache` - 顶级文件夹
- `~/.minikube/cache/iso/<arch>` - 虚拟机 ISO 映像。通常在每次 minikube 主要版本发布时更新一次。
- `~/.minikube/cache/kic/<arch>` - Docker 基础映像。通常在每次 minikube 主要版本发布时更新一次。也就是`kicbase:v0.0.45`的位置
- `~/.minikube/cache/images/<arch>` - Kubernetes 使用的映像，仅在预加载不存在时存在。就是上面几个coredns等
- `~/.minikube/cache/<os>/<arch>/<version>` - Kubernetes 二进制文件，例如 kubeadm 和 kubelet
- `~/.minikube/cache/preloaded-tarball` - 预加载映像的 tarball，以缩短启动时间

在一个联网设备上，将这些缓存下载好后，直接拷贝到另一台机器也可以运行

```shell
=> tree .minikube/cache
.minikube/cache
├── images
│   └── amd64
│       └── proxy-url
│           └── google_containers
│               ├── coredns
│               │   └── coredns_v1.11.1
│               ├── etcd_3.5.15-0
│               ├── k8s-minikube
│               │   └── storage-provisioner_v5
│               ├── kube-apiserver_v1.31.0
│               ├── kube-controller-manager_v1.31.0
│               ├── kube-proxy_v1.31.0
│               ├── kube-scheduler_v1.31.0
│               └── pause_3.10
├── kic
│   └── amd64
│       └── kicbase_v0.0.45.tar
└── linux
    └── amd64
        └── v1.31.0
            ├── kubeadm
            ├── kubectl
            └── kubelet

24 directories, 23 files
```

## 2. ingress

minikube安装默认带了ingress，不过是自己搭建k8s可能会没带，并且minikube安装ingress在国内环境下会失败。在这个场景下就可以自己安装，自定义配置镜像源。

先下载一个官方的yaml文件: https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.12.0-beta.0/deploy/static/provider/cloud/deploy.yaml

里面的版本`controller-v1.12.0-beta.0`就找 https://github.com/kubernetes/ingress-nginx 仓库的tag的最新稳定的版本，下载到本地后修改几处地方

1. 里面的image的地址改为镜像地址，防止下载镜像出现问题

```shell
=> grep 'image: ' ingress-nginx.yaml
        image: xxx/ingress-nginx/controller:v1.11.3@sha256:d56f135b6462cfc476447cfe564b83a45e8bb7da2774963b00d12161112270b7
        image: xxx/ingress-nginx/kube-webhook-certgen:v1.4.4@sha256:a9f03b34a3cbfbb26d103a14046ab2c5130a80c3d69d526ff8063d2b37b9fd3f
        image: xxx/ingress-nginx/kube-webhook-certgen:v1.4.4@sha256:a9f03b34a3cbfbb26d103a14046ab2c5130a80c3d69d526ff8063d2b37b9fd3f
```

2. 修改service的参数配置

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
    app.kubernetes.io/version: 1.11.3
  name: ingress-nginx-controller
  namespace: ingress-nginx
spec:
  # 修改为Cluster，否则会出现只有跑了ingress-controller的pod的node才能访问，Cluster代表不管在哪个节点访问都会转发到这个pod上
  externalTrafficPolicy: Cluster
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - appProtocol: http
    name: http
    port: 31080     # 集群内部端口监听
    nodePort: 31080 # 节点对外暴露端口
    protocol: TCP
    targetPort: http
  - appProtocol: https
    name: https
    port: 31443     # 集群内部端口监听
    nodePort: 31443 # 节点对外暴露端口
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/component: controller
    app.kubernetes.io/instance: ingress-nginx
    app.kubernetes.io/name: ingress-nginx
  type: LoadBalancer
```

修改完直接命令进行生效即可

```shell
kubectl apply -f deploy.yaml
```

然后查看安装状态，安装的默认namespace在`ingress-nginx`下

```shell
=> kubectl get all -n ingress-nginx
NAME                                            READY   STATUS      RESTARTS   AGE
pod/ingress-nginx-admission-create-v9d5g        0/1     Completed   0          47m
pod/ingress-nginx-admission-patch-l4vrw         0/1     Completed   0          47m
pod/ingress-nginx-controller-6467df6ff5-frtkm   1/1     Running     0          47m

NAME                                       TYPE         CLUSTER-IP     EXTERNAL-IP PORT(S)                          AGE
service/ingress-nginx-controller           LoadBalancer 10.103.119.108 <pending>   31080:31080/TCP,31443:31443/TCP  47m
service/ingress-nginx-controller-admission ClusterIP    10.109.229.55  <none>      443/TCP                          47m

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ingress-nginx-controller   1/1     1            1           47m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/ingress-nginx-controller-6467df6ff5   1         1         1       47m

NAME                                       STATUS     COMPLETIONS   DURATION   AGE
job.batch/ingress-nginx-admission-create   Complete   1/1           32s        47m
job.batch/ingress-nginx-admission-patch    Complete   1/1           32s        47m
```

## 3. 使用kubeadm命令部署k8s集群

版本信息

```
kubernetesVersion: 1.28.0
```

### 3.1. 前期环境准备

1. 先配置软件源可以安装kubeadm、kubelet、kubectl三个软件，不同的系统不同的安装方式，这里不列举了，自己找。
2. 然后要安装cri-dockerd，因为kubernetes从1.24版本不再支持docker的api，支持原生的containerd的api，可以使用cri-dockerd进行中转一下。或者使用containerd。不过containerd要将`/etc/containerd/config.toml`中的`disabled_plugins`中去掉cri才能用
   - cri-dockerd在github上可以下载安装： https://github.com/Mirantis/cri-dockerd/releases
3. 需要关闭swap，可以查看一下`/etc/fstab`里面swap相关配置，使用`sudo swapoff -a`，然后删除`/etc/fstab`里面关于swap相关的东西
4. 需要加载`br_netfilter`驱动`sudo modprobe br_netfilter`

### 3.2. kubeadm初始化master节点

设定一个master节点和其他节点的名字，写入到hosts里面，将对应ip写好

```
10.200.2.25  k8s-master
10.200.2.26  k8s-node-1
10.200.2.27  k8s-node-2
```

生成k8s的初始化配置

```shell
kubeadm config print init-defaults > init-defaults.yaml
```

里面的配置可以按需更改，pod和service网段建议修改为两个自己，用来给globalnetworkpolicy配置使用

```yaml
localAPIEndpoint:
  advertiseAddress: 10.200.2.25 # 这个ip是用来kubectl连接使用，apiserver使用的
...
nodeRegistration:
  criSocket: unix:///run/cri-dockerd.sock	# 使用containerd这里就不用改，使用docker则要改成cri-docker的sock地址
  imagePullPolicy: IfNotPresent
  name: k8s-master	# 当前节点的名字，上面host里面配置的
...
imageRepository: xxxx  # 配置一下镜像源，用于拉取镜像，可以直接使用docker的镜像源
...
networking:
  dnsDomain: cluster.local
  podSubnet: 2.1.0.0/16  # 修改为你想要的 Pod 网段
  serviceSubnet: 2.2.0.0/16 # 修改为你想要的 Service 网段
```

先拉取需要的镜像

```shell
kubeadm config images pull --config=init-defaults.yaml
```

启动前需要做个事情，这个不清楚为什么kubelet不使用我们定义的镜像，要自己去拉取。将pause两个镜像版本都tag一下，放置kubelete拉取镜像失败导致启动失败

```shell
docker tag xxxxxx/google_containers/pause:3.9 registry.k8s.io/pause:3.9
```

全部搞完之后的images列表

```shell
=> docker images
REPOSITORY                                                      TAG       IMAGE ID       CREATED         SIZE
xxxxxx/google_containers/kube-apiserver            v1.28.0   bb5e0dde9054   15 months ago   126MB
xxxxxx/google_containers/kube-controller-manager   v1.28.0   4be79c38a4ba   15 months ago   122MB
xxxxxx/google_containers/kube-scheduler            v1.28.0   f6f496300a2a   15 months ago   60.1MB
xxxxxx/google_containers/kube-proxy                v1.28.0   ea1030da44aa   15 months ago   73.1MB
xxxxxx/google_containers/etcd                      3.5.9-0   73deb9a3f702   18 months ago   294MB
xxxxxx/google_containers/coredns                   v1.10.1   ead0a4a53df8   22 months ago   53.6MB
xxxxxx/google_containers/pause                     3.9       e6f181688397   2 years ago     744kB
registry.k8s.io/pause                                           3.9       e6f181688397   2 years ago     744kB
```

然后就开始启动

```shell
kubeadm init --config=init-defaults.yaml
```

顺利的话几分钟就可以完成，不顺利有报错就需要各种百度、谷歌查询了，最终效果如下

```shell
=> kubeadm init --config=init-defaults.yaml
...
[kubelet-finalize] Updating "/etc/kubernetes/kubelet.conf" to point to a rotatable kubelet client certificate and key
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.200.2.25:6443 --token 34ejyu.nedlcler8abxceht \
    --discovery-token-ca-cert-hash sha256:1243115a6c423dc241d44b96eaaa04a3bbe6f6c5cccb7b10920ce19843b41c09
```

按照提示的步骤将kubectl的配置搞好

```shell
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

查看节点状态

```shell
kubectl get node
NAME         STATUS      ROLES           AGE   VERSION
k8s-master   NotReady    control-plane   86m   v1.28.2
```

为什么是NotReady呢，查看kubelet服务可以看到，是网络插件没有安装

### 3.3. 安装网络插件calico

还有很多其他的网络插件，这里使用一个常见的插件calico。先下载calico的模板文件

```shell
# calico主要组件
wget https://docs.projectcalico.org/manifests/calico.yaml
# calico的apiserver，不安装的话可以集群间转发，但是用不了globalnetworkpolicy
wget https://docs.projectcalico.org/manifests/apiserver.yaml
```

修改calico.yaml里面的内容，把ip配置一下

```yaml
			# The default IPv4 pool to create on startup if none exists. Pod IPs will be
            # chosen from this range. Changing this value after installation will have
            # no effect. This should fall within `--cluster-cidr`.
            # - name: CALICO_IPV4POOL_CIDR
            #   value: "192.168.0.0/16"
            # Disable file logging so `kubectl logs` works.
            # 上面注释拷贝下来，将ipv4的地址池配置一下，配置为init-defaults.yaml里面的地址池就好了
            - name: CALICO_IPV4POOL_CIDR
              value: "2.1.0.0/16"
```

修改一下image的镜像源，这里面的都是官方的，能拉取到就不改，基本是要改的

```shell
=> grep 'image:' calico.yaml
          image: docker.io/calico/cni:v3.25.0
          image: docker.io/calico/cni:v3.25.0
          image: docker.io/calico/node:v3.25.0
          image: docker.io/calico/node:v3.25.0
          image: docker.io/calico/kube-controllers:v3.25.0
```

然后就是直接启动

```shell
kubectl apply -f calico.yaml
kubectl apply -f apiserver.yaml
```

等一段时间后查看节点状态

```shell
=> kubectl get node
NAME         STATUS   ROLES           AGE   VERSION
k8s-master   Ready    control-plane   86m   v1.28.2
```

配置calico的apiserver的证书

```shell
openssl req -x509 -nodes -newkey rsa:4096 -keyout apiserver.key -out apiserver.crt -days 365 -subj "/" -addext "subjectAltName = DNS:calico-api.calico-apiserver.svc"
kubectl create secret -n calico-apiserver generic calico-apiserver-certs --from-file=apiserver.key --from-file=apiserver.crt
kubectl patch apiservice v3.projectcalico.org -p \
    "{\"spec\": {\"caBundle\": \"$(kubectl get secret -n calico-apiserver calico-apiserver-certs -o go-template='{{ index .data "apiserver.crt" }}')\"}}"
```

查看是否可以使用calico的api了

```shell
=> kubectl api-resources | grep '\sprojectcalico.org'
bgpconfigurations                 bgpconfig,bgpconfigs                            projectcalico.org/v3                   false        BGPConfiguration
bgppeers                                                                          projectcalico.org/v3                   false        BGPPeer
blockaffinities                   blockaffinity,affinity,affinities               projectcalico.org/v3                   false        BlockAffinity
caliconodestatuses                caliconodestatus                                projectcalico.org/v3                   false        CalicoNodeStatus
clusterinformations               clusterinfo                                     projectcalico.org/v3                   false        ClusterInformation
felixconfigurations               felixconfig,felixconfigs                        projectcalico.org/v3                   false        FelixConfiguration
globalnetworkpolicies             gnp,cgnp,calicoglobalnetworkpolicies            projectcalico.org/v3                   false        GlobalNetworkPolicy
globalnetworksets                                                                 projectcalico.org/v3                   false        GlobalNetworkSet
hostendpoints                     hep,heps                                        projectcalico.org/v3                   false        HostEndpoint
ipamconfigurations                ipamconfig                                      projectcalico.org/v3                   false        IPAMConfiguration
ippools                                                                           projectcalico.org/v3                   false        IPPool
ipreservations                                                                    projectcalico.org/v3                   false        IPReservation
kubecontrollersconfigurations                                                     projectcalico.org/v3                   false        KubeControllersConfiguration
networkpolicies                   cnp,caliconetworkpolicy,caliconetworkpolicies   projectcalico.org/v3                   true         NetworkPolicy
networksets                       netsets                                         projectcalico.org/v3                   true         NetworkSet
profiles                                                                          projectcalico.org/v3                   false        Profile
```

完美搞定主节点，已经可以用这个单节点学习使用k8s了

### 3.4. 去除主节点的taints（污点）

使用kubeadm部署会默认给master节点带一个污点，主要是让pod不调度到这里。但是我们还是不想要浪费一个节点的资源，所以把这个污点去掉。

```shell
=> kubectl describe node k8s-master | grep -i 'taints'
Taints:             node-role.kubernetes.io/control-plane:NoSchedule
```

要么需要将pod的spec中添加tolerations来容忍这个taints，要么直接去掉。容忍的配置如下

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
```

直接去掉master下的taints，后面的`-`代表去除

```shell
kubectl taint nodes k8s-master node-role.kubernetes.io/control-plane:NoSchedule-
```

这样pod就不需要添加tolerations就可以调度到这个节点

### 3.5. 使用kubeadm添加其他节点到k8s集群

其他节点只需要满足前文的前置环境要求即可一行命令添加到k8s集群内部。现在主节点上执行命令获取加入命令

```shell
=> kubeadm token create --print-join-command
kubeadm join 10.200.2.25:6443 --token uiobb1.6s0u4r5smlz6kc9t --discovery-token-ca-cert-hash sha256:4350f712cf668e7ab2099b4d9a60a3addf1e77742ce2d281e226b3d303d6a79c
```

然后在从节点输入这个命令，需要加两个参数

- 由于我们使用的是cri-dockerd做的中转，所以需要指定一下cri-dockerd的socket，`--cri-socket /run/cri-dockerd.sock`
- 定义一下加入时使用的节点名字，`--node-name=k8s-node-1`

网络通的情况下就可以很快看到加入成功。

```shell
=> sudo kubeadm join 10.200.2.25:6443 --token uiobb1.6s0u4r5smlz6kc9t --discovery-token-ca-cert-hash sha256:4350f712cf668e7ab2099b4d9a60a3addf1e77742ce2d281e226b3d303d6a79c --cri-socket /run/cri-dockerd.sock --node-name=k8s-node-1
W1209 15:55:45.449350 2432150 initconfiguration.go:120] Usage of CRI endpoints without URL scheme is deprecated and can cause kubelet errors in the future. Automatically prepending scheme "unix" to the "criSocket" with value "/run/cri-dockerd.sock". Please update your configuration!
[preflight] Running pre-flight checks
[preflight] Reading configuration from the cluster...
[preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -o yaml'
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Starting the kubelet
[kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
```

在主节点上查看一下情况，已经加入成功了

```shell
=> kubectl get node
NAME         STATUS   ROLES           AGE     VERSION
k8s-master   Ready    control-plane   3d23h   v1.28.2
k8s-node-1   Ready    <none>          20s     v1.28.2
```

## 4. 部署私有镜像库

由于k8s一般会有很多节点，我们使用的是docker进行部署。那么就存在一个问题，pod不管在哪个节点启动，都需要此节点上有一份镜像才能启动pod。有下面两个场景

1. 一般k8s部署在内网的场景，无法从docker hub拉取镜像
2. 很多镜像是自己开发的，需要在每个节点上加载到docker里面才能使用

为了解决这样的场景，我们在k8s集群内部部署一个私有镜像站就可以完美解决。所有node想要使用镜像就从这个私有镜像站拉取进行部署pod。

部署私有镜像站需要用到docker的registry，可以直接从 https://github.com/distribution/distribution/releases 下载符合自己平台的二进制，建议把这个镜像站部署在主节点，部署在其他节点也可以。

```shell
# 创建存放目录
mkdir /opt/registry
# 解压二进制到目录中
tar -xzvf registry_2.8.3_linux_amd64.tar.gz -C /opt/registry/
# 添加软链接可以调用二进制
ln -sf /opt/registry/registry /usr/local/bin/
```

使用https就需要生成一下证书文件，我们用openssl命令生成一下

```shell
mkdir /opt/registry/cert
cd /opt/registry/cert
# 生成ca的私钥
openssl genrsa -out cakey.pem 2048
# 生成匹配域名的证书，将后面的test.com改成自己需要的域名就好
openssl req -new -x509 -days 36500 -key cakey.pem -out cacert.pem -subj "/C=CN/O=harbor" -extensions v3_req -config <( printf "[v3_req]\nsubjectAltName=DNS:harbor-local.harbor")
# 配置证书被设备信任，不然docker会拉取镜像失败
cp /opt/registry/cert/cacert.pem /usr/local/share/ca-certificates/harbor.crt
update-ca-certificates
# 要重启一下docker，不然证书授信不生效
systemctl restart docker
```

先写一个配置文件，配置一下私有镜像站的相关属性，放到`/opt/registry/config.yml`

```yaml
version: 0.1    # 配置文件的版本号
log:            # 日志配置
  accesslog:            # 访问日志配置
    disabled: true      # 禁用访问日志
  level: error          # 日志级别设置为错误
  formatter: text       # 日志格式为文本
  fields:               # 日志字段
    service: registry   # 服务名称为 registry
http:           # HTTP 配置
  addr: :5000                           # 监听地址和端口为 5000
  headers:                              # HTTP 头部配置
    X-Content-Type-Options: [nosniff]   # 设置 X-Content-Type-Options 头，防止 MIME 类型嗅探
  http2:                                # HTTP/2 配置
    disabled: false                     # 启用 HTTP/2
  tls:                                  # TLS 配置
    certificate: /opt/registry/cert/cacert.pem   # TLS 证书文件路径
    key: /opt/registry/cert/cakey.pem           # TLS 密钥文件路径
storage:        # 存储配置
  delete:                       # 删除配置
    enabled: true               # 启用删除功能
  cache:                        # 缓存配置
    blobdescriptor: inmemory    # Blob 描述符缓存使用内存
  filesystem:                   # 文件系统配置
    rootdirectory: /data/harbor/registry/   # 存储根目录
  redirect:                     # 重定向配置
    disable: true               # 禁用重定向
  maintenance:                  # 维护配置
    uploadurging:               # 上传排队配置
      enabled: false            # 禁用上传排队
```

写一个私有镜像的服务`/usr/lib/systemd/system/harbor.service`

```
[Unit]
Description=Docker Registry
After=network.target

[Service]
ExecStart=/usr/local/bin/registry serve /opt/registry/config.yml
Restart=always

[Install]
WantedBy=multi-user.target
```

服务起来了，我们需要注册到k8s里面，让k8s可以拉取到镜像。这里使用k8s的service的方式，写一个service和endpoint

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: harbor
---
apiVersion: v1
kind: Service
metadata:
  name: harbor-local
  namespace: harbor
spec:
  type: ClusterIP
  ports:
    - port: 443           # Service 的端口
      targetPort: 5000    # 转发到的目标端口
  selector: {}            # 不使用 selector，直接绑定到 Endpoint
---
apiVersion: v1
kind: Endpoints
metadata:
  name: harbor-local
  namespace: harbor
subsets:
  - addresses:
      - ip: 199.200.2.25  # 替换为实际的 IP 地址
    ports:
      - port: 5000          # 目标端口
```

生效一下就好

```shell
kubectl apply -f harbor.yaml
```

查看一下效果

```shell
# 查看endpoint生效
=> kubectl get ep -n harbor -o wide
NAME           ENDPOINTS           AGE
harbor-local   199.200.2.25:5000   74s
# 查看service生效
=> kubectl get all -n harbor -o wide
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE   SELECTOR
service/harbor-local   ClusterIP   10.100.151.147   <none>        443/TCP   79s   <none>
# 查看service的信息
=> kubectl describe -n harbor svc
Name:              harbor-local
Namespace:         harbor
Labels:            <none>
Annotations:       <none>
Selector:          <none>
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.100.151.147
IPs:               10.100.151.147
Port:              <unset>  443/TCP
TargetPort:        5000/TCP
Endpoints:         199.200.2.25:5000
Session Affinity:  None
Events:            <none>
# 请求这个服务试一下
=> curl -X GET https://10.100.151.147/v2/_catalog -k
{"repositories":[]}
# 内部k8s可以使用这个域名进行拉取镜像，10.96.0.10是coredns的服务ip
=> nslookup harbor-local.harbor.svc.cluster.local 10.96.0.10
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   harbor-local.harbor.svc.cluster.local
Address: 10.100.151.147
```

集群ip和域名已经可以在整个k8s集群下生效了，但是这个域名和ip的对应关系是动态的，我们写个定时脚本来动态修改域名。这里使用域名为`harbor-local.harbor`

```shell
#!/bin/bash

# ！！！里面的sudo -u test需要改成自己的用户名！！！
ip_addr="$(sudo -u test kubectl get svc -n harbor | tail -n 1 | awk '{print $3}')"
if [ -z "$ip_addr" ]; then
    echo "get ip failed"
    exit 1
fi

if grep -q "^$ip_addr harbor-local.harbor$" /etc/hosts; then
    echo "up to date"
    exit 0
fi

if ! grep -q "harbor-local" /etc/hosts; then
    if [ -n "$(tail -n 1 /etc/hosts)" ]; then
        echo >> /etc/hosts
    fi
    echo "$ip_addr harbor-local.harbor" >> /etc/hosts
else
    sed -i "s/.*harbor-local.harbor.*/$ip_addr harbor-local.harbor/" /etc/hosts
fi
```

放到每个节点的`/etc/cron.min`下面，加上可执行权限，就可以自动把`harbor-local.harbor`解析到其服务地址了。如果没有`cron.min`，自己百度解决

下面就可以测试一下k8s拉取镜像的操作了

```shell
# 先使用docker拉取一个官网的镜像
docker pull golang:latest
# 修改镜像的地址为localhost
docker tag golang:latest localhost:5000/golang:latest
# 推送到私有镜像站上
docker push localhost:5000/golang:latest
```

写个pod的yaml文件

```shell
apiVersion: v1			# 版本信息
kind: Pod				# 类型是pod
metadata:
  name: test-pod		# pod取名为test-pod
  labels:				# 添加标签，主要为了service暴露端口使用
    app: test
spec:					# 定义spec期望
  containers:
  - name: test					# 一个test的镜像
    image: harbor-local.harbor/golang:latest		# 使用私有镜像站的地址拉取镜像
    imagePullPolicy: Always		# 有私有镜像，那么直接随时拉取最新的就好了
    tty: true         # golang的镜像需要设置tty，不然就退出了
```

生效一下

```shell
kubectl apply -f test-pod.yaml
```

看效果，我们在master节点上制作的镜像上传到私有仓库，在node-2上面也可以拉下来运行

```shell
=> kubectl get pod -o wide
NAME                   READY   STATUS    RESTARTS   AGE     IP                NODE         NOMINATED NODE   READINESS GATES
test-pod               1/1     Running   0          65s     192.168.140.100   k8s-node-2   <none>           <none>
```

## 5. istio安装

### 1) 安装

Istio 是一个强大的工具，适用于需要管理复杂微服务架构的场景。它提供了丰富的功能，帮助开发者实现流量管理、安全性和可观察性。一般生产环境用于做灰度部署使用，如将header带某些标签的客户转发到另一组微服务上。

第一步先下载istio相关命令和配置组件，访问这个链接 https://github.com/istio/istio/releases 下载对应版本，当前使用最新的1.24.2。下载下来解压，放到一个目录中

```shell
tar -xzvf istio-1.24.2-linux-amd64.tar.gz
mv istio-1.24.2 /opt/
# 将istioctl放到一个位置，用于配置
ln -sf /opt/istio-1.24.2/bin/istioctl /usr/local/bin/
```

istio依赖crd（CustomResourceDefinitions）的机制，我们需要下载istio需要定义的crd配置

```shell
# 克隆仓库切换版本
git clone github.com/kubernetes-sigs/gateway-api
cd gateway-api
git checkout v1.2.0

# 将crd添加到k8s中
cd config/crd
kubectl kustomize . | kubectl apply -f -
```

k8s一般有自带的ingress或service来对外暴露，我们就不需要安装istio-gateway，现在开始安装istiod

```shell
cd /opt/istio-1.24.2
istioctl install -f samples/bookinfo/demo-profile-no-gateways.yaml -y

# 需要修改镜像源就直接使用kubectl修改，改配置太麻烦了
kubectl edit -n istio-system deployment.apps/istiod
```

查看安装结果，istiod在running状态即可

```shell
=> kubectl get all -n istio-system
NAME                          READY   STATUS    RESTARTS   AGE
pod/istiod-5df459fbd5-7zkm8   1/1     Running   0          2m36s

NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)                                 AGE
service/istiod   ClusterIP   2.2.136.150   <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP   3m10s

NAME                     READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istiod   1/1     1            1           3m10s

NAME                                DESIRED   CURRENT   READY   AGE
replicaset.apps/istiod-5df459fbd5   1         1         1       2m36s
replicaset.apps/istiod-78959b574b   0         0         0       3m10s
```

想要配置一下sidecar的镜像源，需要先修改configmap

```shell
kubectl edit cm istio-sidecar-injector -n istio-system
```

将里面的`"hub": "docker.io/istio"`改掉即可，需要重启pod，可以直接删除pod，由deployment自动重启。到这里基本就安装完了，很简单

### 2) 部署测试一下灰度效果

我们先给要注入sidecar的命名空间添加默认注入，我使用的是default

```shell
kubectl label namespace default istio-injection=enabled
```

创建一个golang的微服务，具体test-con的代码和镜像参考后文的`八、实战 :: 1. 使用一个简单golang服务器体验k8s`，这里写一下deployment，生成两个带标签`version: v1`和`version: v2`的服务，模拟两个版本的服务

```yaml
# test-v1
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
    version: v1
  name: test-v1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: test
      version: v1
  template:
    metadata:
      labels:
        app: test
        version: v1
    spec:
      containers:
      - image: harbor-local.harbor/test-con:v2
        imagePullPolicy: Always
        ports:
        - containerPort: 7878
        name: test-v1
---
# test-v2
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: test
    version: v2
  name: test-v2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: test
      version: v2
  template:
    metadata:
      labels:
        app: test
        version: v2
    spec:
      containers:
      - image: harbor-local.harbor/test-con:v2
        imagePullPolicy: Always
        ports:
        - containerPort: 7878
        name: test-v2
```

两个服务都启起来后，我们写个service来暴露这个服务

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: test
    service: test
  name: test
  namespace: default
spec:
  ports:
  - name: http
    port: 31234
    targetPort: 7878
  selector:
    app: test
  type: ClusterIP
```

生效后就可以通过集群ip来访问这个服务，正常应该多次访问都是不同的pod返回的，达到了service的负载均衡效果。写一个ingress的配置来放通到外部

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
  name: test
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - backend:
          service:
            name: test
            port:
              number: 31234
        path: /
        pathType: ImplementationSpecific
  tls:
  - secretName: chart-test-tls
```

从外部访问ingress暴露的端口，也是负载均衡到不同的pod上。下面开始进行灰度部署，将流量转发到v1上，先配置DestinationRule定义两个集合

```yaml
apiVersion: networking.istio.io/v1
kind: DestinationRule
metadata:
  name: test-destination
spec:
  host: test # 服务名
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
```

配置服务来定义将请求仅转发给v1

```yaml
apiVersion: networking.istio.io/v1
kind: VirtualService
metadata:
  name: test-route
spec:
  hosts:
  - test # 服务名
  http:
  - route:
    - destination:
        host: test # 服务名
        subset: v1
```

生效后，使用ingress的端口访问，竟然没有生效。那是因为ingress的pod没有注入sidecar，需要修改ingress的deployment来注入sidecar

```shell
kubectl patch deployments -n ingress-nginx ingress-nginx-controller -p '{"spec":{"template":{"metadata":{"labels":{"sidecar.istio.io/inject":"true"}}}}}' --type merge
```

生效后发现还是不行，是因为ingress-nginx的差异，需要在ingress的配置中添加两个注解

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    # 禁用nginx自己的负载均衡，直接请求到service，默认会直接到pod，nginx会自己搞负载均衡
    nginx.ingress.kubernetes.io/service-upstream: "true"
    # 修改上游host，这个是为了让envoy可以识别到对应的host来定义服务
    nginx.ingress.kubernetes.io/upstream-vhost: "test.default.svc.cluster.local"
  name: test
  namespace: default
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - backend:
          service:
            name: test # 服务名
            port:
              number: 31234
        path: /
        pathType: ImplementationSpecific
  tls:
  - secretName: chart-test-tls
```

生效后，就可以通过ingress-nginx的端口来访问服务，达到了灰度的效果，只会请求到v1的pod上。修改成v2就只会请求到v2的pod上

### 3) istio的可观测性

启动istio的仪表盘

```shell
# 直接全部开启
cd /opt/istio-1.24.2
kubectl apply -f samples/addons
# 修改镜像源就直接进去改
kubectl edit -n istio-system deployment
```

### 4) 卸载istio

```shell
# 删除命名空间的默认注入的标签，注意最后的减号
kubectl label namespace default istio-injection-

# 删除istiod
istioctl uninstall -y --purge

# 清理添加进来的crd
cd /path/to/gateway-api/config/crd
kubectl kustomize . | kubectl delete -f -
```

# 三、命令详解

## 1. kubectl

### 通用选项

- `-n <namespace>`: 指定namespace，不输入则在default下查找
- `--all-namespaces`: 在所有namespace下查看
- `--force`: 强制创建

### 1.1. pods相关

```shell
########## 获取状态 ##########
# 获取简单状态
kubectl get pods
# 获取某个pods的详细状态
kubectl describe pods <pod-name>

########## pod操作 ##########
# 进入pod执行命令
kubectl exec -it <pod-name> -- bash

########## 删除pod ##########
kubectl delete pods <pod-name>
```

### 1.2. services相关

```shell
########## 获取状态 ##########
# 获取简单状态
kubectl get services
```

### 1.3. deployments

```shell
########## 获取状态 ##########
# 获取简单状态
kubectl get deployments

########## 操作 ##########
# 重启deployment
kubectl rollout restart deployment/<deployment-name> -n <namespace>
```

### 1.4. yml操作

```shell
# 使用yml配置启动
kubectl apply -f test.yml
```

### 1.5. 其他信息查看

```shell
# 查看api-version对应的kind有哪些，不加--api-group显示所有资源
kubectl api-resources --api-group=crd.projectcalico.org -o wide
# 解释某一种api-resources
kubectl explain <resource_name>
```

# 四、yaml配置文件详解

## 1. pod定义

```yaml
apiVersion: v1			# 版本信息
kind: Pod				# 类型是pod
metadata:
  name: test-pod		# pod取名为test-pod
  labels:				# 添加标签，主要为了service暴露端口使用
    app: test
spec:					# 定义spec期望
  containers:
  - name: test					# 一个test镜像
    image: test-con:latest		# 使用test-con:latest镜像
    imagePullPolicy: Never		# 不允许从官方搜索，只找本地
    # tty: true					# 和docker一样，没有定义command就需要指定这个保证容器不退出
    command: ["go"]				# 镜像启动后执行的命令
    args: ["run", "/root/test.go"]
```

### 1.1. spec.containers[*].imagePullPolicy

镜像拉取策略

- Always: 每次都拉取最新的镜像
- IfNotPresent: 本地不存在镜像彩绘从仓库拉取
- Never: 从不从镜像仓库拉取，只使用本地已有的镜像

当镜像标签是latest，默认是Always。当镜像标签不是，默认是IfNotPresent

## 2. service定义

```yaml
apiVersion: v1
kind: Service			  # 类型是service
metadata:
  name: test-service	# 起个名字，这个名字会映射一个dns地址到clusterIP
spec:
  clusterIP: xxx    # 不设置会自动给一个ip，设置为None代表Headless service，请求dns返回pod的ip
  externalTrafficPolicy: Cluster  # 默认是Cluster，集群内会进行转发，Local只会找本节点
  type: NodePort		# 具体看后面解释
  selector:				  # 选择器，此服务对应哪些pod
    app: test			  # app为test的pod
  ports:
  - protocol: TCP
    port: 80			# 集群内部监听端口
    targetPort: 7878	# pod里面的端口
    # nodePort: 30001   # 外部监听的端口，不设置就是随机一个，设置就要取30000以上的，可以和port定义成一样的
```

service的dns地址为

```
<service_name>.<namespace>.svc.cluster.local
```

### 2.1. spec.type各个类型说明

| 类型         | 说明                                      | 效果                                            |
| ------------ | ----------------------------------------- | ----------------------------------------------- |
| ClusterIP    | 将pod的端口通过集群内部ip暴露给集群访问   | targetPort=>port                                |
| NodePort     | 扩展ClusterIP，在节点上暴露端口给外部访问 | targetPort=>port=>nodePort                      |
| LoadBalancer | 扩展NodePort，多了externalip              | targetPort=>port=>nodePort=>externalip:nodePort |
| ExternalName | 将服务映射到一个外部的 DNS 名称。         | 待补充                                          |

### 2.2. spec.clusterIP

- 不设置的话k8s会自动分配一个ip
- 设置为None则不会分配ip，请求dns会返回pod列表和对应的ip，也就是对应Headless Service

### 2.3. spec.externalTrafficPolicy

- Cluster：这是默认值。选择 Cluster 时，Kubernetes 会将外部流量路由到集群中的所有 Pod，无论它们在哪个节点上。
  - 优点：
    - 负载均衡：流量会均匀分配到所有后端 Pod。
    - 简单易用：不需要额外的配置。
  - 缺点：
    - 外部流量的源 IP 地址会被替换为负载均衡器或节点的 IP 地址，这可能会影响某些应用程序的功能（例如，基于 IP 的访问控制）。
- Local：选择 Local 时，外部流量只会路由到那些在同一节点上运行的 Pod。
  - 优点：
    - 保留源 IP：外部流量的源 IP 地址不会被修改，这对于某些需要源 IP 的应用程序非常重要。
    - 更好的性能：减少了跨节点的流量，可能会提高性能。
  - 缺点：
    - 负载不均：如果某些节点上的 Pod 较少，可能会导致流量不均匀分配。
    - 需要确保每个节点都有足够的 Pod 来处理流量，否则可能会导致流量丢失。

## 3. deployment定义

```yaml
apiVersion: apps/v1		# 版本，虽然不知道为什么，但是要写apps/v1
kind: Deployment		# 选deployment
metadata:
  name: test-dep		# 给deployment定义名字，创建的pod会以此为前缀
spec:					# deployment的要求
  strategy:
    type: RollingUpdate	# 策略为滚动更新
    rollingUpdate:
      maxUnavailable: 1	# 最多只能停止1个pod
      maxSurge: 1		# 在更新过程中，最多可以多创建多少个 Pod，本身停止了1个，创建1个，这里设置1会停1个创建2个
  replicas: 3			# pods启动3个
  selector:				# 定义deployment管理的pod选择器
    matchLabels:		# 要跟下面的template中一样，不一样会报错
      app: test
  template:				# 定义deployment管理的pod
    metadata:
      labels:			# 定义标签，要和deployment中一样
        app: test
    spec:				# pod的定义
      volumes:  # 定义要求的存储卷
        - name: test-volume         # 起个名字给pod使用
          persistentVolumeClaim:    # 对应的pvc
            claimName: test-local-pvc
      containers:
        - name: test
          image: test-con:latest
          imagePullPolicy: Never  # 使用本地镜像，不从远端拉取
          command: ["go"]
          args: ["run", "/root/test.go"]
          volumeMounts:         # 挂载到当前容器
            - mountPath: /data  # 挂载目录
              name: test-volume # 使用的volumes，是上面定义的名字
          readinessProbe:			# 就绪探针，什么时候可以开始接收流量，不会重启pod
            httpGet:
              path: /
              port: 7878
            timeoutSeconds: 1		# 探测超时时间5s
            initialDelaySeconds: 5	# 第一次探测的等待时间
            periodSeconds: 5		# 探测周期
          livenessProbe:			# 存活探针，什么时候需要杀掉这个pod，如果启动一直失败就会杀掉pod重启一个
            httpGet:
              path: /
              port: 7878
            timeoutSeconds: 1		# 探测超时时间5s
            initialDelaySeconds: 20	# 第一次探测的等待时间
            periodSeconds: 5		# 探测周期
            failureThreshold: 10	# 错误阈值，超过这个阈值将会重启pod
```

## 4. PersistentVolume

```yaml
apiVersion: v1          # 版本信息
kind: PersistentVolume  # 这个是一个持久化存储卷
metadata:
  name: test-local-pv   # 元数据定义名称
spec:
  capacity:
    storage: 1Ki        # 定义容量1KB，定义不定义都一样，因为目录挂载的方式取决于原始目录的容量
  storageClassName: local                 # 定义这个为了和pvc匹配，否则会绑定不了
  persistentVolumeReclaimPolicy: Retain   # pvc被删除时，pv的回收策略，retain为保留pv，手动清理
  accessModes:
    - ReadWriteMany     # 表示该卷可以被多个节点以读写的方式挂载。这意味着多个 Pod 可以同时访问这个卷并进行读写操作。
  hostPath:
    path: /data
```

### 4.1. storageClass 存储类型

k8s内置了几个存储类型，里面定义了一些存储的策略，也可以自己定义。但是pvc的存储类型和pv的要一致，否则会无法绑定到特定的pv上

## 5. PersistentVolumeClaim

```yaml
apiVersion: v1                # 版本
kind: PersistentVolumeClaim   # pvc类型，请求持久化存储卷
metadata:
  name: test-local-pvc
spec:
  accessModes:
    - ReadWriteMany           # 这个模式表示多个节点可以同时读写这个存储。
  resources:
    requests:
      storage: 1Ki            # 要和pv匹配，不然会无法绑定
  volumeName: test-local-pv   # 指定要绑定的pv的名字
  storageClassName: local     # 同样要和pv一致
```

## 6. DaemonSet

```yaml
apiVersion: apps/v1  # 指定 API 版本，表示使用 apps/v1 版本的 API
kind: DaemonSet  # 资源类型为 DaemonSet，确保在每个节点上运行一个 Pod
metadata:
  name: test  # DaemonSet 的名称为 test
  namespace: default  # DaemonSet 所在的命名空间为 default
spec:
  revisionHistoryLimit: 10  # 保留的历史修订版本数量，最多为 10
  selector:  # 选择器，用于选择与此 DaemonSet 关联的 Pods
    matchLabels:
      app.kubernetes.io/instance: test  # 标签选择器，匹配 app.kubernetes.io/instance 为 test 的 Pods
      app.kubernetes.io/name: test  # 标签选择器，匹配 app.kubernetes.io/name 为 test 的 Pods
  template:  # Pod 模板，用于创建 Pods
    metadata:
      creationTimestamp: null  # 创建时间戳，通常由系统自动生成
      labels:  # Pod 的标签
        app.kubernetes.io/instance: test  # 标签，标识实例为 test
        app.kubernetes.io/name: test  # 标签，标识名称为 test
    spec:  # Pod 的规格
      containers:  # 容器列表
      - env:  # 环境变量配置
        - name: TEST  # 环境变量的名称为 TEST
          valueFrom:  # 从其他资源获取值
            configMapKeyRef:  # 从 ConfigMap 中获取值
              key: TEST  # ConfigMap 中的键为 TEST
              name: test-config  # ConfigMap 的名称为 test-config
        image: harbor-local.harbor/test-con:v2  # 容器镜像，指定使用的镜像及其版本
        imagePullPolicy: Always  # 镜像拉取策略，始终拉取最新镜像
        livenessProbe:  # 存活探针配置
          failureThreshold: 3  # 失败阈值，连续失败 3 次后认为容器不健康
          httpGet:  # HTTP GET 请求探测
            path: /  # 请求路径
            port: 7878  # 请求端口
            scheme: HTTP  # 使用的协议
          initialDelaySeconds: 20  # 启动后等待 20 秒后开始探测
          periodSeconds: 5  # 每 5 秒进行一次探测
          successThreshold: 1  # 成功阈值，至少成功 1 次认为健康
          timeoutSeconds: 3  # 请求超时时间为 3 秒
        name: test  # 容器名称为 test
        ports:  # 容器端口配置
        - containerPort: 31234  # 容器暴露的端口为 31234
          name: http  # 端口名称为 http
          protocol: TCP  # 使用的协议为 TCP
        readinessProbe:  # 就绪探针配置
          failureThreshold: 3  # 失败阈值，连续失败 3 次后认为容器不就绪
          httpGet:  # HTTP GET 请求探测
            path: /  # 请求路径
            port: 7878  # 请求端口
            scheme: HTTP  # 使用的协议
          initialDelaySeconds: 5  # 启动后等待 5 秒后开始探测
          periodSeconds: 5  # 每 5 秒进行一次探测
          successThreshold: 1  # 成功阈值，至少成功 1 次认为就绪
          timeoutSeconds: 3  # 请求超时时间为 3 秒
        volumeMounts:  # 卷挂载配置
        - mountPath: /data  # 挂载路径为 /data
          name: test-volume  # 卷的名称为 test-volume
      volumes:  # 卷配置
      - name: test-volume  # 卷的名称为 test-volume
        persistentVolumeClaim:  # 使用持久卷声明
          claimName: test-local-pvc  # 持久卷声明的名称为 test-local-pvc
  updateStrategy:  # 更新策略配置
    rollingUpdate:  # 滚动更新策略
      maxSurge: 0  # 在更新时，最多可以超出期望 Pod 数量的 0 个
      maxUnavailable: 1  # 在更新时，最多可以不可用的 Pod 数量为 1
    type: RollingUpdate  # 更新类型为滚动更新
```

## 7. Ingress

### 7.1. 几个特殊注解解释

#### 1) nginx.ingress.kubernetes.io/service-upstream

- 功能：该注解用于指示 NGINX Ingress Controller 直接将请求转发到后端服务，而不是通过 NGINX 的负载均衡机制。
- 用法：当你希望 Ingress Controller 直接将请求转发到 Kubernetes 服务，而不经过 NGINX 的负载均衡时，可以使用此注解。

#### 2) nginx.ingress.kubernetes.io/upstream-vhost

- 功能：该注解用于设置上游主机名（upstream host），可以在请求转发到后端服务时指定一个特定的主机名。
- 用法：当后端服务需要特定的主机名来处理请求时，可以使用此注解。它通常用于需要基于主机名进行路由的服务。

#### 3) nginx.ingress.kubernetes.io/ssl-redirect

- 功能：当设置为 true 时，所有通过 HTTP 访问的请求会被自动重定向到 HTTPS。这意味着，如果用户尝试通过不安全的 HTTP 协议访问你的应用，Ingress Controller 会返回一个 301 或 302 重定向响应，指向相同的 URL，但使用 HTTPS。
- 默认值：如果没有设置这个注解，默认情况下值为true，Ingress Controller 会自动重定向 HTTP 到 HTTPS。

# 五、helm使用

Helm 是 Kubernetes 的一个包管理工具，类似于 Linux 中的 apt 或 yum。它使得在 Kubernetes 上部署和管理应用程序变得更加简单和高效。

## 1. 安装

下载压缩包，解压就可以了

```shell
https://get.helm.sh/helm-v3.16.3-linux-amd64.tar.gz
```

具体版本号可以在github上看到最新的版本号，然后替换上面链接来获取。github: https://github.com/helm/helm/releases

## 2. 基础使用

```shell
########## repo操作 ##########
# 添加repo，下面是一些常用的repo
helm repo add charts                https://charts.helm.sh/stable/
helm repo add gitlab                https://charts.gitlab.io/
helm repo add kubernetes            https://burdenbear.github.io/kube-charts-mirror/
helm repo add kubernetes-charts     http://mirror.azure.cn/kubernetes/charts/
helm repo add kubeshark             https://helm.kubeshark.co/
helm repo add victoriametrics       https://victoriametrics.github.io/helm-charts/

# 更新repo，类似于apt update
helm repo update

# 查看repo列表
helm repo list

# 在repo中查找一个包
helm search repo nginx

########## chart操作 ##########
# 在当前目录创建项目，会新建文件夹
helm create <project_name>
# 部署项目
helm install <release_name> <project_dir>
# 删除项目
helm uninstall <release_name>
# 热升级项目
helm upgrade <release_name> <new_project_dir>
# 回滚项目，可以回滚到历史的版本
helm rollback <release_name> <revision_num>
# 查看项目的状态
helm status <release_name>
# 查看项目历史版本
helm history <release_name>
# 查看项目历史某个版本详细配置
helm get all <release_name> --revision <revision_num>
```

# 六、minikube使用

minikube是在宿主机上启动一个k8s虚拟机，用来运行k8s的一些基础服务。可选的虚拟机引擎有virtualbox, kvm2, qemu2, qemu, vmware, none, docker, podman等

## 1. 通用命令

```shell
########## 进入minikube虚拟化环境 #########
minikube ssh
```

## 2. image镜像处理

```shell
########## 查看 ##########
# 列举所有镜像
minikube image ls

########## 加载镜像 ##########
# 从docker加载镜像到minikube内部docker
minikube image load <docker_image_name:tag>

########## 删除 ##########
# 删除某个镜像
minikube image rm <image_name:tag>
```

## 3. mount挂载本地目录到虚拟机内

```shell
# 挂载本地目录到minikube虚拟机内
# 注：命令执行会阻塞，中断的话挂载也就失效了
minikube mount /path/to/host:/path/to/minikube
```

# 七、架构和原理

## 1. kubernetes的架构

<img src="2024-10-14-01.svg" />

### 1.1. Control Plane 控制平面组件

控制平面组件为集群做全局决策，如资源调度、检测和响应集群时间。也可以说是k8s集群的主节点，管理整个k8s集群内部的pod调度。

# 八、实战

下述实战均是在minikube下完成的，先参考安装章节的minikube安装，宿主机要有docker，引擎选择docker。

## 1. 使用一个简单golang服务器体验k8s

先在宿主机的docker下载一下golang的最新镜像

```shell
docker pull golang:test
```

写个简单的服务器，监听7878端口，请求都返回`hello, world`

```go
package main

import (
    "fmt"
    "net/http"
    "os"
)

func main() {
    fmt.Println("start main")
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        fmt.Fprintf(w, "hello, world")
    })
    err := http.ListenAndServe(":7878", nil)
    if err != nil {
        panic(err)
    }
    os.Exit(0)
}
```

制作docker镜像，写一个dockerfile

```dockerfile
# 使用镜像golang:latest
FROM golang:latest

# 添加变量指定code_dir
ARG CODE_DIR="/code"

# 设置 root 用户的密码，默认使用test，但是留下root可以登陆
RUN echo 'root:1' | chpasswd

# 创建目录
# 添加用户
# 将目录所属交给新用户
RUN mkdir -p ${CODE_DIR} && \
    useradd -m -s /bin/bash test && \
    chown -R test:test ${CODE_DIR}

# 将文件按照新用户所属复制到工作目录
COPY --chown=test:test ./test.go ${CODE_DIR}

# 指定后面的操作为新用户上下文
USER test:test

# 指定工作目录
WORKDIR ${CODE_DIR}

# 指定容器启动命令，不可被覆盖
ENTRYPOINT ["go", "run", "test.go"]
```

制作镜像

```shell
# -t 指定镜像名和版本
# -f 指定dockerfile
# --no-cache 不使用缓存
docker rmi test-con:latest && docker build -t test-con:latest -f Dockerfile --no-cache .
```

这个时候查看docker的镜像里面就存在了我们制作的要发布的镜像

```shell
=> docker images
REPOSITORY                                                               TAG        IMAGE ID       CREATED          SIZE
test-con                                                                 latest     f6ee44e250ad   32 minutes ago   911MB
golang                                                                   latest     0457bb691895   12 days ago      838MB
xxxxxx/google_containers/k8s-minikube/kicbase               v0.0.45    aeed0e1d4642   2 months ago     1.28GB
xxxxxx/google_containers/kicbase                            v0.0.45    aeed0e1d4642   2 months ago     1.28GB
xxxxxx/google_containers/coredns/coredns                    v1.11.1    cbb01a7bd410   15 months ago    59.8MB
xxxxxx/google_containers/dashboard                          v2.7.0     07655ddf2eeb   2 years ago      246MB
xxxxxx/google_containers/kubernetesui/dashboard             v2.7.0     07655ddf2eeb   2 years ago      246MB
xxxxxx/google_containers/kubernetesui/metrics-scraper       v1.0.8     115053965e86   2 years ago      43.8MB
xxxxxx/google_containers/metrics-scraper                    v1.0.8     115053965e86   2 years ago      43.8MB
xxxxxx/google_containers/k8s-minikube/storage-provisioner   v5         6e38f40d628d   3 years ago      31.5MB
xxxxxx/google_containers/storage-provisioner                v5         6e38f40d628d   3 years ago      31.5MB
```

注意，这里的镜像是宿主机的docker的，不是minikube里面的，minikube是个类似虚拟机的环境，里面还有一个docker。我们把这个镜像加载进去

```shell
minikube image load test-con:latest
```

然后就能看到这个这个镜像在minikube里面

```
=> minikube image ls
xxxxxx/google_containers/pause:3.10
xxxxxx/google_containers/kubernetesui/metrics-scraper:v1.0.8
xxxxxx/google_containers/kubernetesui/dashboard:v2.7.0
xxxxxx/google_containers/kube-scheduler:v1.31.0
xxxxxx/google_containers/kube-proxy:v1.31.0
xxxxxx/google_containers/kube-controller-manager:v1.31.0
xxxxxx/google_containers/kube-apiserver:v1.31.0
xxxxxx/google_containers/k8s-minikube/storage-provisioner:v5
xxxxxx/google_containers/etcd:3.5.15-0
xxxxxx/google_containers/coredns/coredns:v1.11.1
docker.io/library/test-con:latest
```

### 1) 使用pod启动镜像给外部访问

写一个yaml文件

```yaml
apiVersion: v1			# 版本信息
kind: Pod				# 类型是pod
metadata:
  name: test-pod		# pod取名为test-pod
  labels:				# 添加标签，主要为了service暴露端口使用
    app: test
spec:					# 定义spec期望
  containers:
  - name: test					# 一个test镜像
    image: test-con:latest		# 使用test-con:latest镜像
    imagePullPolicy: Never		# 不允许从官方搜索，只找本地
```

启动一下看看效果

```shell
kubectl apply -f test.yaml
```

看一下pod情况

```shell
=> kubectl get pods
NAME       READY   STATUS    RESTARTS   AGE
test-pod   1/1     Running   0          38m
```

STATUS在Running状态就对了，但是现在还是访问不了，因为只是在pod里面监听了7878端口，但是外部还没有监听。前面讲了service是用来将pod里面的端口对外暴露使用的，那么定义一个service

```yaml
apiVersion: v1
kind: Service			# 类型是service
metadata:
  name: test-service	# 起个名字
spec:
  type: NodePort		# 具体看后面解释
  selector:				# 选择器，此服务对应哪些pod
    app: test			# app为test的pod
  ports:
  - protocol: TCP
    port: 80			# 集群内部监听端口
    targetPort: 7878	# pod里面的端口
    # nodePort: 30001   # 外部监听的端口，不设置就是随机一个，设置就要取30000以上的
```

启动看看效果

```shell
kubectl apply -f test-service.yaml
```

查看service情况

```shell
=> kubectl get services
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP        37d
test-service   NodePort    10.99.198.195   <none>        80:30449/TCP   57m
```

可以看到集群节点内部监听了80，外部暴露了一个随机端口30449，想要访问这个端口需要使用minikube对外暴露的ip。

```shell
=> minikube ip
192.168.49.2
=> curl 192.168.49.2:30449
hello, world%
```

到这里，自己启动的pod给外部访问成功了

### 2) 使用deployment管理起来

直接创建deployment来创建pod，顺便测试一下故障场景，修改一下test.go来增加退出场景，需要重新制作镜像，参考上面即可。

```go
package main

import (
	"context"
	"fmt"
	"io"
	"net"
	"net/http"
	"net/http/httptrace"
	"os"
	"time"
)

func main() {
	fmt.Println("start main")
	hostName := os.Getenv("HOSTNAME")
	test := os.Getenv("TEST")
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// 获取连接信息
		addr, ok := r.Context().Value(http.LocalAddrContextKey).(net.Addr)
		connStr := "empty"
		if ok {
			connStr = fmt.Sprintf("%s => %s", r.RemoteAddr, addr)
		}
		contentBytes, err := os.ReadFile("/data/test.txt")
		outStr := ""
		if err != nil {
			outStr = err.Error()
		} else {
			outStr = string(contentBytes)
		}
		fmt.Fprintf(w, "hello, world, %s, txt: '%s'\nversion: %s\nconnStr: %s\nenv: TEST %s",
			hostName, outStr, r.Header["Version"], connStr, test)
	})
	http.HandleFunc("/save", func(w http.ResponseWriter, r *http.Request) {
		_ = r.ParseForm()
		tmp := r.Form["content"]
		inputText := ""
		if len(tmp) > 0 {
			inputText = tmp[0]
		}
		_ = os.WriteFile("/data/test.txt", []byte(inputText), 0644)
		fmt.Fprintf(w, "%s", inputText)
	})
	http.HandleFunc("/exit", func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("recieve exit, bye!")
		os.Exit(0)
	})
	http.HandleFunc("/proxy", func(w http.ResponseWriter, r *http.Request) {
		_ = r.ParseForm()
		url := r.Form["url"][0]

		// 创建一个新的 HTTP 客户端
		client := &http.Client{
			Transport: &http.Transport{
				DisableKeepAlives: true, // 禁用连接保持
			},
			Timeout: 5 * time.Second, // 设置请求超时时间
		}

		connStr := "empty"
		trace := &httptrace.ClientTrace{
			GotConn: func(info httptrace.GotConnInfo) {
				connStr = fmt.Sprintf("%s => %s", info.Conn.LocalAddr(), info.Conn.RemoteAddr())
			},
		}

		req, err := http.NewRequestWithContext(httptrace.WithClientTrace(context.Background(), trace),
			http.MethodGet, url, nil)
		if err != nil {
			fmt.Fprintf(w, "%+v", err)
			return
		}

		resp, err := client.Do(req)
		if err != nil {
			fmt.Fprintf(w, "%+v", err)
			return
		}
		defer resp.Body.Close()

		// 读取响应体
		body, err := io.ReadAll(resp.Body)
		if err != nil {
			fmt.Fprintf(w, "%+v", err)
			return
		}
		fmt.Fprintf(w, "%s, %s\n%s", hostName, connStr, string(body))
	})
	err := http.ListenAndServe(":7878", nil)
	if err != nil {
		panic(err)
	}
	os.Exit(0)
}
```

重新制作镜像搞到minikube里面之后，我们开始定义deployment来管理pod

```yaml
apiVersion: apps/v1		# 版本，虽然不知道为什么，但是要写apps/v1
kind: Deployment		# 选deployment
metadata:
  name: test-dep		# 给deployment定义名字，创建的pod会以此为前缀
spec:					# deployment的要求
  replicas: 1			# pods启动1个，保持1个
  selector:				# 定义deployment管理的pod选择器
    matchLabels:		# 要跟下面的template中一样，不一样会报错
      app: test
  template:				# 定义deployment管理的pod
    metadata:
      labels:			# 定义标签，要和deployment中一样
        app: test
    spec:				# pod的定义
      containers:
        - name: test
          image: test-con:latest
          imagePullPolicy: Never
```

然后就是启动一下deployment

```shell
kubectl apply -f test.yaml
```

查看deployment和pod的状态

```shell
=> kubectl get deployments
NAME       READY   UP-TO-DATE   AVAILABLE   AGE
test-dep   1/1     1            1           4m6s
=> kubectl get pods
NAME                        READY   STATUS    RESTARTS   AGE
test-dep-769f4564f5-56zv6   1/1     Running   0          4m10s
```

在Running状态后，service那边不用动，同样用上面的方式访问，可以通。

```shell
=> minikube ip
192.168.49.2
=> curl 192.168.49.2:30449
hello, world%
```

然后测试一下搞挂这个服务端，自己会恢复

```shell
=> curl 192.168.49.2:30449/exit; date; while true; do curl 192.168.49.2:30449 &>/dev/null; if [ $? -eq 0 ]; then break; fi; sleep 1; done; date
curl: (52) Empty reply from server
2024年 11月 19日 星期二 20:06:46 CST
2024年 11月 19日 星期二 20:07:29 CST
```

pods的状态也是有一次重启

```shell
=> kubectl get pods
NAME                        READY   STATUS    RESTARTS       AGE
test-dep-769f4564f5-56zv6   1/1     Running   1 (119s ago)   42m
```

测试一下pods的容错，我们把deployments配置改一下，把replicas改成3个。等待启动完成再次测试。

```shell
=> kubectl apply -f test.yaml --force
# 搞挂第一个，第二个可以响应，很快返回
=> curl 192.168.49.2:30449/exit; date; while true; do curl 192.168.49.2:30449 &>/dev/null; if [ $? -eq 0 ]; then break; fi; sleep 1; done; date
curl: (52) Empty reply from server
2024年 11月 19日 星期二 20:17:15 CST
2024年 11月 19日 星期二 20:17:15 CST
# 搞挂第二个，第三个可以响应，很快返回
=> curl 192.168.49.2:30449/exit; date; while true; do curl 192.168.49.2:30449 &>/dev/null; if [ $? -eq 0 ]; then break; fi; sleep 1; done; date
curl: (52) Empty reply from server
2024年 11月 19日 星期二 20:17:16 CST
2024年 11月 19日 星期二 20:17:17 CST
# 搞挂第三个，第一个重启还没搞定，等第一个重启好就可以了
=> curl 192.168.49.2:30449/exit; date; while true; do curl 192.168.49.2:30449 &>/dev/null; if [ $? -eq 0 ]; then break; fi; sleep 1; done; date
curl: (52) Empty reply from server
2024年 11月 19日 星期二 20:17:19 CST
2024年 11月 19日 星期二 20:18:06 CST
```

由于我写的是`go run test.go` 容器启动之后需要现场编译，所以会慢一点，编译好的二进制启动会更快，应该就都是快速返回了。可以看到service里面暴露的端口会自动负载均衡到三个pods上面。

### 3) 热升级pod

负载均衡的容错基本可以了，现在考虑要升级一下这个容器，把其中的test.go修改一下，返回hello, world的同时打印一下HOSTNAME。

```go
package main

import (
    "fmt"
    "net/http"
    "os"
)

func main() {
    fmt.Println("start main")
    // 从环境变量取hostname，为pod的名称，如test-dep-6cb67f4fbb-n8d78
    hostName := os.Getenv("HOSTNAME")
    http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
        // 返回时加上这个HOSTNAME
        fmt.Fprintf(w, "hello, world, %s\n", hostName)
    })
    http.HandleFunc("/exit", func(w http.ResponseWriter, r *http.Request) {
        fmt.Println("recieve exit, bye!")
        os.Exit(0)
    })
    err := http.ListenAndServe(":7878", nil)
    if err != nil {
        panic(err)
    }
    os.Exit(0)
}
```

重新制作镜像，版本使用v2

```shell
# -t 指定镜像名和版本
# -f 指定dockerfile
# --no-cache 不使用缓存
docker rmi test-con:v2 && docker build -t test-con:v2 -f Dockerfile --no-cache .

# 加载到minikube里面去
minikube image load test-con:v2
```

先修改一下deployment来处理一下更新的策略

```yaml
apiVersion: apps/v1		# 版本，虽然不知道为什么，但是要写apps/v1
kind: Deployment		# 选deployment
metadata:
  name: test-dep		# 给deployment定义名字，创建的pod会以此为前缀
spec:					# deployment的要求
  strategy:
    type: RollingUpdate	# 策略为滚动更新
    rollingUpdate:
      maxUnavailable: 1	# 最多只能停止1个pod
      maxSurge: 1		# 在更新过程中，最多可以多创建多少个 Pod，本身停止了1个，创建1个，这里设置1会停1个创建2个
  replicas: 3			# pods启动3个
  selector:				# 定义deployment管理的pod选择器
    matchLabels:		# 要跟下面的template中一样，不一样会报错
      app: test
  template:				# 定义deployment管理的pod
    metadata:
      labels:			# 定义标签，要和deployment中一样
        app: test
    spec:				# pod的定义
      containers:
        - name: test
          image: test-con:latest
          imagePullPolicy: Never
          readinessProbe:			# 就绪探针，什么时候可以开始接收流量，不会重启pod
            httpGet:
              path: /
              port: 7878
            timeoutSeconds: 1		# 探测超时时间5s
            initialDelaySeconds: 5	# 第一次探测的等待时间
            periodSeconds: 5		# 探测周期
          livenessProbe:			# 存活探针，什么时候需要杀掉这个pod，如果启动一直失败就会杀掉pod重启一个
            httpGet:
              path: /
              port: 7878
            timeoutSeconds: 1		# 探测超时时间5s
            initialDelaySeconds: 20	# 第一次探测的等待时间
            periodSeconds: 5		# 探测周期
            failureThreshold: 10	# 错误阈值，超过这个阈值将会重启pod
```

里面的image先使用`test-con:latest`，使用老的然后测试更新为新的，开不同的窗口，执行下面的命令监控状态

```shell
# 窗口1：查看pod状态
watch -n 1 kubectl get pods
# 窗口2：查看服务对外状态
watch -n 1 curl 192.168.49.2:30449 -s
# 窗口3：查看更新状态
watch -n 1 kubectl rollout status deployment/test-dep
```

三个窗口搞定之后，开始进行热更新。两种方式，可以直接修改deployment里面的image，也可以使用下面的命令

```shell
kubectl set image deployment/test-dep test=test-con:v2
```

可以看到窗口1里面pod的更新状态，先停止1个，启动2个pod。2个pod就绪后停止第三个，启动1个pod。这个时候窗口2中的curl结果变成了`hello, world, test-dep-7668f6d499-x72cg`每秒变一下不同的pod返回，并且**<font color="red">整个过程中没有获取失败的情况</font>**。到此热升级完成。

但是作为运维人员，假设升级出现问题，需要回滚，执行下面命令就可以回滚。

```shell
kubectl rollout undo deployment/test-dep
```

### 4) 本地目录实现持久化存储

容器是无状态的，每次重启都是新的进程，但是我们需要将一些状态数据如配置、用户数据等存到本地来方便新的容器可以拿到历史状态。先创建一个目录来存放数据，并且挂载到minikube虚拟机内（不是pod里面）。注意要新开一个终端来调用，这个命令会阻塞，不能中断。

```shell
=> mkdir storage
=> minikube mount storage:/data
```

可以测试一下是否成功了

```shell
=> echo aaa > storage/test.txt
=> minikube ssh 'cat /data/test.txt'
aaa
```

然后创建一个持久化存储卷（PersistentVolume）的配置，挂载本机目录，注意是minikube虚拟机的目录，所以是`/data`

```yaml
apiVersion: v1          # 版本信息
kind: PersistentVolume  # 这个是一个持久化存储卷
metadata:
  name: test-local-pv   # 元数据定义名称
spec:
  capacity:
    storage: 1Ki        # 定义容量1KB，定义不定义都一样，因为目录挂载的方式取决于原始目录的容量
  storageClassName: local                 # 定义这个为了和pvc匹配，否则会绑定不了
  persistentVolumeReclaimPolicy: Retain   # pvc被删除时，pv的回收策略，retain为保留pv，手动清理
  accessModes:
    - ReadWriteMany     # 表示该卷可以被多个节点以读写的方式挂载。这意味着多个 Pod 可以同时访问这个卷并进行读写操作。
  hostPath:
    path: /data
```

pv是不能直接给pod访问和挂载使用的，需要使用pvc才能给到pod使用，所以写一个pvc的配置

```yaml
apiVersion: v1                # 版本
kind: PersistentVolumeClaim   # pvc类型，请求持久化存储卷
metadata:
  name: test-local-pvc
spec:
  accessModes:
    - ReadWriteMany           # 这个模式表示多个节点可以同时读写这个存储。
  resources:
    requests:
      storage: 1Ki            # 要和pv匹配，不然会无法绑定
  volumeName: test-local-pv   # 指定要绑定的pv的名字
  storageClassName: local     # 同样要和pv一致
```

启动pv和pvc，要看一下是否启动和绑定成功

```shell
kubectl apply -f test-pv.yaml
kubectl apply -f test-pvc.yaml
```

pv和pvc的绑定关系使用describe看起来会更清晰一点

```shell
=> kubectl describe pv test-local-pv
Name:            test-local-pv
Labels:          <none>
Annotations:     pv.kubernetes.io/bound-by-controller: yes
Finalizers:      [kubernetes.io/pv-protection]
StorageClass:    local
Status:          Bound
Claim:           default/test-local-pvc
Reclaim Policy:  Retain
Access Modes:    RWX
VolumeMode:      Filesystem
Capacity:        1Ki
Node Affinity:   <none>
Message:
Source:
    Type:          HostPath (bare host directory volume)
    Path:          /data
    HostPathType:
Events:            <none>

=> kubectl describe pvc test-local-pvc
Name:          test-local-pvc
Namespace:     default
StorageClass:  local
Status:        Bound
Volume:        test-local-pv
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Ki
Access Modes:  RWX
VolumeMode:    Filesystem
Used By:       <none>
Events:            <none>
```

可以看到pv和pvc绑定成功

下面要修改一下test.go，让我们的服务端可以读取和写入存储的一个文件

```go
package main

import (
	"fmt"
	"io/ioutil"
	"net/http"
	"os"
)

func main() {
	fmt.Println("start main")
	hostName := os.Getenv("HOSTNAME")
  // 正常请求根目录读取这个文件返回内容
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		contentBytes, err := ioutil.ReadFile("/data/test.txt")
		outStr := ""
		if err != nil {
			outStr = err.Error()
		} else {
			outStr = string(contentBytes)
		}
		fmt.Fprintf(w, "hello, world, %s, txt: '%s'\n", hostName, outStr)
	})
  // 保存文件，取xxx:xxx/save?content=xxx来存入
	http.HandleFunc("/save", func(w http.ResponseWriter, r *http.Request) {
		r.ParseForm()
		tmp := r.Form["content"]
		inputText := ""
		if len(tmp) > 0 {
			inputText = tmp[0]
		}
		ioutil.WriteFile("/data/test.txt", []byte(inputText), 0644)
		fmt.Fprintf(w, "%s", inputText)
	})
	http.HandleFunc("/exit", func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("recieve exit, bye!")
		os.Exit(0)
	})
	err := http.ListenAndServe(":7878", nil)
	if err != nil {
		panic(err)
	}
	os.Exit(0)
}
```

修改完重新做一个镜像加载到minikube中，然后要修改一下deployment的配置来让pod使用pvc作为存储

```yaml
apiVersion: apps/v1		# 版本，虽然不知道为什么，但是要写apps/v1
kind: Deployment		# 选deployment
metadata:
  name: test-dep		# 给deployment定义名字，创建的pod会以此为前缀
spec:					# deployment的要求
  strategy:
    type: RollingUpdate	# 策略为滚动更新
    rollingUpdate:
      maxUnavailable: 1	# 最多只能停止1个pod
      maxSurge: 1		# 在更新过程中，最多可以多创建多少个 Pod，本身停止了1个，创建1个，这里设置1会停1个创建2个
  replicas: 3			# pods启动3个
  selector:				# 定义deployment管理的pod选择器
    matchLabels:		# 要跟下面的template中一样，不一样会报错
      app: test
  template:				# 定义deployment管理的pod
    metadata:
      labels:			# 定义标签，要和deployment中一样
        app: test
    spec:				# pod的定义
      volumes:  # 定义要求的存储卷
        - name: test-volume         # 起个名字给pod使用
          persistentVolumeClaim:    # 对应的pvc
            claimName: test-local-pvc
      containers:
        - name: test
          image: test-con:latest
          imagePullPolicy: Never  # 使用本地镜像，不从远端拉取
          volumeMounts:         # 挂载到当前容器
            - mountPath: /data  # 挂载目录
              name: test-volume # 使用的volumes，是上面定义的名字
          readinessProbe:			# 就绪探针，什么时候可以开始接收流量，不会重启pod
            httpGet:
              path: /
              port: 7878
            timeoutSeconds: 1		# 探测超时时间5s
            initialDelaySeconds: 5	# 第一次探测的等待时间
            periodSeconds: 5		# 探测周期
          livenessProbe:			# 存活探针，什么时候需要杀掉这个pod，如果启动一直失败就会杀掉pod重启一个
            httpGet:
              path: /
              port: 7878
            timeoutSeconds: 1		# 探测超时时间5s
            initialDelaySeconds: 20	# 第一次探测的等待时间
            periodSeconds: 5		# 探测周期
            failureThreshold: 10	# 错误阈值，超过这个阈值将会重启pod
```

生效一下这个配置文件，查看pod的pvc使用情况

```shell
kubectl apply -f test-dep.yaml
```

查看pvc里面更能看出来挂载情况

```shell
=> kubectl describe pvc test-local-pvc
Name:          test-local-pvc
Namespace:     default
StorageClass:  local
Status:        Bound
Volume:        test-local-pv
Labels:        <none>
Annotations:   pv.kubernetes.io/bind-completed: yes
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      1Ki
Access Modes:  RWX
VolumeMode:    Filesystem
Used By:       test-dep-6dc4bdc5bd-7mf7w
               test-dep-6dc4bdc5bd-ftv24
               test-dep-6dc4bdc5bd-w8zxf
Events:            <none>
```

### 5) 使用statefulset管理带状态服务

deployment主要管理无状态的pod，就算需要磁盘也是期望做动态扩容的。而对于那种有状态需要存储数据的，如log、数据库、消息中间件需要持久化数据的，则建议使用StatefulSet，我们的pod现在扩充了pv和pvc，那就使用StatefulSet重新部署一下试试。我们把deployment的配置稍微改一下就可以了

```yaml
apiVersion: apps/v1		# 版本，虽然不知道为什么，但是要写apps/v1
kind: StatefulSet		# 选StatefulSet
metadata:
  name: test-st			# 给StatefulSet定义名字，创建的pod会以此为前缀
spec:					# StatefulSet的要求
  replicas: 3			# pods启动3个
  selector:				# 定义管理的pod选择器
    matchLabels:		# 要跟下面的template中一样，不一样会报错
      app: test
  template:				# 定义管理的pod
    metadata:
      labels:			# 定义标签，要和上面的一样
        app: test
    spec:				# pod的定义
      volumes:  # 定义要求的存储卷
        - name: test-volume         # 起个名字给pod使用
          persistentVolumeClaim:    # 对应的pvc
            claimName: test-local-pvc
      containers:
        - name: test
          image: test-con:latest
          imagePullPolicy: Never  # 使用本地镜像，不从远端拉取
          volumeMounts:         # 挂载到当前容器
            - mountPath: /data  # 挂载目录
              name: test-volume # 使用的volumes，是上面定义的名字
          readinessProbe:			# 就绪探针，什么时候可以开始接收流量，不会重启pod
            httpGet:
              path: /
              port: 7878
            timeoutSeconds: 1		# 探测超时时间5s
            initialDelaySeconds: 5	# 第一次探测的等待时间
            periodSeconds: 5		# 探测周期
          livenessProbe:			# 存活探针，什么时候需要杀掉这个pod，如果启动一直失败就会杀掉pod重启一个
            httpGet:
              path: /
              port: 7878
            timeoutSeconds: 1		# 探测超时时间5s
            initialDelaySeconds: 20	# 第一次探测的等待时间
            periodSeconds: 5		# 探测周期
            failureThreshold: 10	# 错误阈值，超过这个阈值将会重启pod
```

启动一个窗口监控pod的状态，看一下启动statefulset的启动效果

```shell
# 删除deployment
kubectl delete deployment test-dep
# 监听pod变化
watch -n 1 kubectl get pod
```

启动statefulset

```shell
kubectl apply -f test-st.yaml
```

会发现pod是一个一个启动的，一个启动成功才会启动另一个，而且名字是按照0，1，2开始的

```shell
=> kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
test-st-0   1/1     Running   0          10m
test-st-1   1/1     Running   0          11m
test-st-2   1/1     Running   0          11m
```

试一试升级到读取文件的版本，看升级过程

```shell
kubectl set image statefulset/test-st test=test-con:v2
```

默认的statefulset的升级策略是从最大的开始升级，一个一个升级到最小的编号

### 6) 使用helm重新生成服务

搞一个pod部署，需要写deployment，写service，写configmap等，太麻烦了。使用helm一个配置文件，生成所有的配置部署更方便。这里使用helm重新部署这个golang服务器。先使用helm生成默认配置，找个目录输入。

```shell
# 创建test项目chart
helm create test
```

会生成一个模板目录`test`里面结构如下

```shell
=> tree test
test
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

4 directories, 10 files
```

其中的template是用来生成对应的k8s配置文件，values.yaml是我们自定义的配置。先使用deployment管理pod，statefulset还需要自己写一个模板文件。修改values.yaml如下

```yaml
# Default values for test-chart.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

# This will set the replicaset count more information can be found here: https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/
replicaCount: 4				# 生成的默认滚动更新策略是25%，取3就没有效果，取4就会maxSurge和maxUnavailable都是1

# This sets the container image more information can be found here: https://kubernetes.io/docs/concepts/containers/images/
image:
  repository: test-con		# 镜像
  # This sets the pull policy for images.
  pullPolicy: Never			# 拉取策略不拉取
  # Overrides the image tag whose default is the chart appVersion.
  tag: "latest"				# 先使用latest，一会切换到v2看一下热更新

# This is for the secretes for pulling an image from a private repository more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
imagePullSecrets: []
# This is to override the chart name.
nameOverride: ""
fullnameOverride: ""

#This section builds out the service account more information can be found here: https://kubernetes.io/docs/concepts/security/service-accounts/
serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Automatically mount a ServiceAccount's API credentials?
  automount: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

# This is for setting Kubernetes Annotations to a Pod.
# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/
podAnnotations: {}
# This is for setting Kubernetes Labels to a Pod.
# For more information checkout: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/
podLabels: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

# This is for setting up a service more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/
service:
  # This sets the service type more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types
  type: NodePort		# service将端口暴露到外面，k8s会自动分配一个集群外部端口，不用配置nodePort
  # This sets the ports more information can be found here: https://kubernetes.io/docs/concepts/services-networking/service/#field-spec-ports
  port: 7878			# 集群内部监听的端口，要设置targetPort，需要改模板，不配置就会将这个端口用于targetPort

# This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/
ingress:
  enabled: false
  className: ""
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

# This is to setup the liveness and readiness probes more information can be found here: https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/
livenessProbe:
  httpGet:
    path: /
    port: 7878
  timeoutSeconds: 3		    # 探测超时时间5s
  initialDelaySeconds: 60	# 第一次探测的等待时间
  periodSeconds: 5		    # 探测周期
readinessProbe:
  httpGet:
    path: /
    port: 7878
  timeoutSeconds: 3		# 探测超时时间5s
  initialDelaySeconds: 20	# 第一次探测的等待时间
  periodSeconds: 5		# 探测周期
  failureThreshold: 15	# 错误阈值，超过这个阈值将会重启pod

#This section is for setting up autoscaling more information can be found here: https://kubernetes.io/docs/concepts/workloads/autoscaling/
autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

# Additional volumes on the output Deployment definition.
volumes:
  - name: test-volume         # 起个名字给pod使用
    persistentVolumeClaim:    # 对应的pvc
      claimName: test-local-pvc
# - name: foo
#   secret:
#     secretName: mysecret
#     optional: false

# Additional volumeMounts on the output Deployment definition.
volumeMounts:
  - mountPath: /data  # 挂载目录
    name: test-volume # 使用的volumes，是上面定义的名字
# - name: foo
#   mountPath: "/etc/foo"
#   readOnly: true

nodeSelector: {}

tolerations: []

affinity: {}
```

pv和pvc我们不动，不需要使用模板生成，还是手动处理更好。helm主要还是管理pod使用。然后就很简单的使用命令安装我们的test项目

```shell
# 先删除上面步骤创建的pod
kubectl delete service test-service
kubectl delete statefulset test-st
# 使用helm直接部署pod，取当前目录下的values.yaml和template目录，名字为test
helm install test ./
```

查看各个组件的情况

```shell
=> kubectl get pods
NAME                    READY   STATUS    RESTARTS   AGE
test-68d446659d-5ssvq   1/1     Running   0          76s
test-68d446659d-8w6s5   1/1     Running   0          86s
test-68d446659d-ddh9c   1/1     Running   0          86s
test-68d446659d-kvs5q   1/1     Running   0          76s
=> kubectl get deployments
NAME   READY   UP-TO-DATE   AVAILABLE   AGE
test   4/4     4            4           2m42s
=> kubectl get services
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          47d
test         NodePort    10.101.143.137   <none>        7878:31064/TCP   2m56s
```

给出了访问的命令，照着执行一下就可以拿到结果

```shell
export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services test)
export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
echo http://$NODE_IP:$NODE_PORT

curl http://$NODE_IP:$NODE_PORT
```

现在体验热更新，只修改values.yaml里面的tag为v2然后就可以开始升级，先启动两个终端查看情况

```shell
watch -n 1 curl http://$NODE_IP:$NODE_PORT
watch -n 1 kubectl get pods
```

升级命令很简单

```shell
helm upgrade test ./
```

等一会就滚动升级完成了，效果和上面的deployment一样，回滚命令如下

```shell
# 回滚test到版本1，helm会记录过程中的所有版本，可以随便回滚，不过需要镜像还在
helm rollback test 1
```

### 7) 使用configMap给pod加环境变量

默认helm生成的模板文件里面并没有configmap的模板，我们写一个`configmap.yaml`到template下面

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ include "test.fullname" . }}-config
  labels:
    {{- include "test.labels" . | nindent 4 }}
data:
  {{- toYaml .Values.configMap.data | nindent 2 }}
```

顺便还要修改一下deployment.yaml的模板文件，从configmap里面加载key到pod里面

```yaml
spec:
  ...
  template:
    ...
    spec:
    ...
      containters:
        - name: {{ .Chart.Name }}
          ...
          env:
            - name: TEST
              valueFrom:
                configMapKeyRef:
                  name: {{ include "test.fullname" . }}-config
                  key: TEST
```

然后就可以在values.yaml里面定义configmap中的TEST的值

```yaml
configMap:
  data:
    TEST: "aaa"
```

需要修改我们的镜像，支持读取这个环境变量并输出出来

```go
package main

import (
	"fmt"
	"net/http"
	"os"
)

func main() {
	fmt.Println("start main")
	hostName := os.Getenv("HOSTNAME")
	test := os.Getenv("TEST")
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		contentBytes, err := os.ReadFile("/data/test.txt")
		outStr := ""
		if err != nil {
			outStr = err.Error()
		} else {
			outStr = string(contentBytes)
		}
		fmt.Fprintf(w, "hello, world, %s, txt: '%s'\nenv: TEST %s", hostName, outStr, test)
	})
	http.HandleFunc("/save", func(w http.ResponseWriter, r *http.Request) {
		_ = r.ParseForm()
		tmp := r.Form["content"]
		inputText := ""
		if len(tmp) > 0 {
			inputText = tmp[0]
		}
		_ = os.WriteFile("/data/test.txt", []byte(inputText), 0644)
		fmt.Fprintf(w, "%s", inputText)
	})
	http.HandleFunc("/exit", func(w http.ResponseWriter, r *http.Request) {
		fmt.Println("recieve exit, bye!")
		os.Exit(0)
	})
	err := http.ListenAndServe(":7878", nil)
	if err != nil {
		panic(err)
	}
	os.Exit(0)
}
```

先使用helm回退到版本1，对`test-con:v2`不进行占用，制作镜像并提交到minikube里面，然后进行热更新到`test-con:v2`查看configmap是否生效

```shell
helm rollback test 1
docker rmi test-con:v2 && docker build -t test-con:v2 -f Dockerfile --no-cache ./
minikube image load test-con:v2
helm upgrade test ./test/
```

查看configmap的状态

```shell
=> kubectl get cm
NAME               DATA   AGE
kube-root-ca.crt   1      48d
test-config        1      4m49s
=> kubectl describe cm test-config
Name:         test-config
Namespace:    default
Labels:       app.kubernetes.io/instance=test
              app.kubernetes.io/managed-by=Helm
              app.kubernetes.io/name=test
              app.kubernetes.io/version=1.16.0
              helm.sh/chart=test-0.1.0
Annotations:  meta.helm.sh/release-name: test
              meta.helm.sh/release-namespace: default

Data
====
TEST:
----
aaa

BinaryData
====

Events:  <none>
```

查看pod状态，很好，生效了

```shell
=> curl http://$NODE_IP:$NODE_PORT
hello, world, test-f9dd4db-d5s47, txt: 'abcde'
env: TEST aaa
```

### 8) 配置ingress将service中的端口暴露出去

service是管理pod的暴露的，但是每个service都暴露一个端口到外部并不是很合适。使用ingress将不同的service暴露的端口通过子url的方式做反向代理，使用7层代理方式暴露给外部。

配置ingress-nginx需要先按照前面讲解的安装方式安装一下，然后就可以修改helm的values.yaml配置文件

```yaml
# This block is for setting up the ingress for more information can be found here: https://kubernetes.io/docs/concepts/services-networking/ingress/
ingress:
  enabled: true
  className: "nginx"
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"  # 禁用重定向
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
    - secretName: chart-test-tls
```

然后做个更新就好了

```shell
helm upgrade test ./
```

可以查看ingress的配置状态

```shell
=> kubectl get ingress
NAME   CLASS   HOSTS   ADDRESS   PORTS     AGE
test   nginx   *                 80, 443   49m
```

获取一下ingress的对外暴露端口

```shell
=> kubectl get -n ingress-nginx svc
NAME                                 TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                           AGE
ingress-nginx-controller             LoadBalancer   10.109.244.245   <pending>     31080:31080/TCP,31443:31443/TCP   15h
ingress-nginx-controller-admission   ClusterIP      10.97.75.238     <none>        443/TCP                           15h
```

测试一下访问

```shell
=> curl https://10.200.2.25:31443 -s -k
hello, world, test-7c8d55fbf8-plcfv, txt: 'abdd'
env: TEST aaa%
=> curl http://10.200.2.25:31080 -s
hello, world, test-7c8d55fbf8-hwsmk, txt: 'aasdfed'
env: TEST aaa%
```

对集群下任意节点的ip访问都可以到达pod得到响应，到此ingress-nginx暴露golang服务端口成功

## 2. k8s下的calico网络策略配置

### 1) 安全考虑放通网络

目标是仅暴露有用的端口，其他端口不允许外部访问。部署k8s的时候把防火墙关闭了，使用calico完全掌控防火墙。而calico是基于endpoint和service来控制访问流量的，我们需要处理外部流量时就需要将节点对外的网口作为endpoint注册到k8s中才能控制其流量。先将对应网口注册到k8s的hostendpoint中，注意打标签用来后面使用。

```yaml
apiVersion: projectcalico.org/v3
kind: HostEndpoint
metadata:
  annotations:
  labels:
    host-endpoint: ens18
    interface-name: ens18
    node-name: k8s-master
    node-role: k8s-master
  name: k8s-master-ens18
spec:
  interfaceName: ens18
  node: k8s-master
---
apiVersion: projectcalico.org/v3
kind: HostEndpoint
metadata:
  annotations:
  labels:
    host-endpoint: lo
    interface-name: lo
    node-name: k8s-master
    node-role: k8s-master
  name: k8s-master-lo
spec:
  interfaceName: lo
  node: k8s-master
---
apiVersion: projectcalico.org/v3
kind: HostEndpoint
metadata:
  annotations:
  labels:
    host-endpoint: ens18
    interface-name: ens18
    node-name: k8s-node-1
    node-role: k8s-worker
  name: k8s-node-1-ens18
spec:
  interfaceName: ens18
  node: k8s-node-1
---
apiVersion: projectcalico.org/v3
kind: HostEndpoint
metadata:
  annotations:
  labels:
    host-endpoint: lo
    interface-name: lo
    node-name: k8s-node-1
    node-role: k8s-worker
  name: k8s-node-1-lo
spec:
  interfaceName: lo
  node: k8s-node-1
---
apiVersion: projectcalico.org/v3
kind: HostEndpoint
metadata:
  annotations:
  labels:
    host-endpoint: ens18
    interface-name: ens18
    node-name: k8s-node-2
    node-role: k8s-worker
  name: k8s-node-2-ens18
spec:
  interfaceName: ens18
  node: k8s-node-2
---
apiVersion: projectcalico.org/v3
kind: HostEndpoint
metadata:
  annotations:
  labels:
    host-endpoint: lo
    interface-name: lo
    node-name: k8s-node-2
    node-role: k8s-worker
  name: k8s-node-2-lo
spec:
  interfaceName: lo
  node: k8s-node-2
```

标签打好了，就可以开始限制网络访问了。我们按照下面的优先级进行处理网络策略。

| 优先级 | 策略说明                 | 方向    |
| ------ | ------------------------ | ------- |
| 0      | 节点对外访问放通         | egress  |
| 5      | service和pod网段相互放通 | ingress |
| 5      | 本机环回网卡放通         | ingress |
| 10     | 对外放通部分端口访问     | ingress |
| 20     | 默认禁止所有流量进入     | ingress |

生效网络策略前，我们先看看k8s对外暴露了哪些端口

```shell
=> sudo nmap -sS -p- -O 199.200.2.25
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-21 15:45 CST
Nmap scan report for 199.200.2.25
Host is up (0.0018s latency).
Not shown: 65524 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
179/tcp   open  bgp
2379/tcp  open  etcd-client
2380/tcp  open  etcd-server
5000/tcp  open  upnp
6443/tcp  open  sun-sr-https
10250/tcp open  unknown
10256/tcp open  unknown
31080/tcp open  unknown
31443/tcp open  unknown
38863/tcp open  unknown
MAC Address: FE:FC:FE:07:FE:6A (Unknown)
Device type: general purpose
Running: Linux 5.X
OS CPE: cpe:/o:linux:linux_kernel:5
OS details: Linux 5.0 - 5.4
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 10.99 seconds
```

开始写策略并生效

```yaml
# 允许本机向外发送流量
apiVersion:   projectcalico.org/v3
kind:         GlobalNetworkPolicy
metadata:
  name:       allow-egress
spec:
  egress:
  - action:  Allow
    destination: {}
    source: {}
  selector: has(host-endpoint)
  order: 0
  types:
  - Egress

---

# 允许k8s集群内部所有节点的pod和service之间转发流量
# 允许k8s集群内部节点ip相互转发流量
apiVersion:   projectcalico.org/v3
kind:         GlobalNetworkPolicy
metadata:
  name:       allow-inernal-ingress
spec:
  applyOnForward: true
  preDNAT: true
  ingress:
  - action:  Allow
    destination: {}
    source:
      nets:
      - 199.200.2.25/32  # k8s-master
      - 199.200.2.26/32  # k8s-node-1
      - 199.200.2.27/32  # k8s-node-2
  - action:  Allow
    destination: {}
    source:
      nets:
      - 2.1.0.0/16    # pod相关ip
      - 2.2.0.0/16    # service相关ip
  selector: has(host-endpoint)
  order: 5
  types:
  - Ingress

---

# 允许k8s集群内部所有节点的pod和service之间转发流量
apiVersion:   projectcalico.org/v3
kind:         GlobalNetworkPolicy
metadata:
  name:       allow-localhost-ingress
spec:
  applyOnForward: true
  preDNAT: true
  ingress:
  - action:  Allow
    destination: {}
    source: {}
  selector: host-endpoint == "lo"
  order: 5
  types:
  - Ingress

---

apiVersion:   projectcalico.org/v3
kind:         GlobalNetworkPolicy
metadata:
  name:       allow-external-ingress
spec:
  applyOnForward: true
  preDNAT: true
  ingress:
  - action:  Allow
    destination:
      ports:
      - 22    # ssh端口
      - 31443 # ingress的https端口
      - 31080 # ingress的http端口
    source: {}
    protocol: TCP
  order: 10
  selector: all()
  types:
  - Ingress

---

# 禁用所有外部访问端口
apiVersion: projectcalico.org/v3
kind: GlobalNetworkPolicy
metadata:
  name: deny-other
spec:
  selector: has(host-endpoint)
  order: 20
  types:
  - Ingress
```

生效一下，再次验证一下端口放通情况

```shell
=> sudo nmap -sS -p- -O 199.200.2.25
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-24 15:49 CST
Nmap scan report for 199.200.2.25
Host is up (0.0020s latency).
Not shown: 65525 filtered tcp ports (no-response)
PORT      STATE  SERVICE
22/tcp    open   ssh
179/tcp   open   bgp
2379/tcp  open   etcd-client
2380/tcp  open   etcd-server
5473/tcp  closed apsolab-tags
6443/tcp  open   sun-sr-https
6666/tcp  closed irc
6667/tcp  closed irc
31080/tcp open   unknown
31443/tcp open   unknown
MAC Address: FE:FC:FE:07:FE:6A (Unknown)
Device type: general purpose|WAP|storage-misc
Running (JUST GUESSING): Linux 5.X|4.X|2.6.X|3.X (95%), Ubiquiti AirOS 5.X (86%), HP embedded (85%)
OS CPE: cpe:/o:linux:linux_kernel:5 cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:2.6 cpe:/o:linux:linux_kernel:3 cpe:/o:ubnt:airos:5.2.6 cpe:/h:hp:p2000_g3
Aggressive OS guesses: Linux 5.0 - 5.4 (95%), Linux 4.15 - 5.8 (92%), Linux 5.0 - 5.5 (91%), Linux 5.1 (91%), Linux 2.6.32 - 3.13 (90%), Linux 2.6.39 (90%), Linux 5.0 (89%), Linux 2.6.22 -
2.6.36 (88%), Linux 3.10 (88%), Linux 3.10 - 4.11 (88%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 114.88 seconds
```

一下子少了好多，但是还有一些端口对外放通了。这是因为k8s有一个动态策略会默认放通下面的端口

参考: https://docs.tigera.io/calico/latest/reference/host-endpoints/failsafe

| Port | Protocol | Direction          | Purpose                         |
| ---- | -------- | ------------------ | ------------------------------- |
| 22   | TCP      | Inbound            | SSH access                      |
| 53   | UDP      | Outbound           | DNS queries                     |
| 67   | UDP      | Outbound           | DHCP access                     |
| 68   | UDP      | Inbound            | DHCP access                     |
| 179  | TCP      | Inbound & Outbound | BGP access (Calico networking)  |
| 2379 | TCP      | Inbound & Outbound | etcd access                     |
| 2380 | TCP      | Inbound & Outbound | etcd access                     |
| 5473 | TCP      | Inbound & Outbound | etcd access                     |
| 6443 | TCP      | Inbound & Outbound | Kubernetes API server access    |
| 6666 | TCP      | Inbound & Outbound | etcd self-hosted service access |
| 6667 | TCP      | Inbound & Outbound | etcd self-hosted service access |

我们需要改动一下动态策略配置，将所有默认端口放开的都删掉，使用我们自己管理来放通。

```shell
kubectl edit FelixConfiguration default
```

添加下面两个配置即可

```
spec:
  ...
  failsafeInboundHostPorts: []
  failsafeOutboundHostPorts: []
```

然后再次nmap看一眼放通的端口

```shell
sudo nmap -sS -p- -O 199.200.2.25
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-24 15:53 CST
Nmap scan report for 199.200.2.25
Host is up (0.0021s latency).
Not shown: 65531 filtered tcp ports (no-response)
PORT      STATE SERVICE
22/tcp    open  ssh
31080/tcp open  unknown
31443/tcp open  unknown
MAC Address: FE:FC:FE:07:FE:6A (Unknown)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose|storage-misc
Running (JUST GUESSING): Linux 4.X|5.X|2.6.X|3.X (93%), Synology DiskStation Manager 5.X (85%)
OS CPE: cpe:/o:linux:linux_kernel:4 cpe:/o:linux:linux_kernel:5 cpe:/o:linux:linux_kernel:2.6.32 cpe:/o:linux:linux_kernel:3.10 cpe:/a:synology:diskstation_manager:5.2
Aggressive OS guesses: Linux 4.15 - 5.8 (93%), Linux 5.0 - 5.4 (93%), Linux 5.0 - 5.5 (92%), Linux 2.6.32 (87%), Linux 3.10 (87%), Linux 3.10 - 4.11 (87%), Linux 3.2 - 4.9 (87%), Linux 3.4 - 3.10 (87%), Linux 5.1 (87%), Linux 2.6.32 - 3.10 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 1 hop

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 115.43 seconds
```

只有我们期望放通的端口被放开了，达成

### 2) 放通一个ip访问6443管理k8s集群

可能运维机器在本地，需要放通其ip来管理k8s集群，那么我们放通一个单独的ip。

```yaml
apiVersion:   projectcalico.org/v3
kind:         GlobalNetworkPolicy
metadata:
  name:       allow-manage-ingress
spec:
  applyOnForward: true
  preDNAT: true
  ingress:
  - action:  Allow
    destination:
      ports:
      - 6443 # kubectl访问的apiserver的端口
    source:
      nets:
      - 10.32.47.26/32   # 生效的ip，也就是运维的机器
    protocol: TCP
  order: 10
  selector: all()
  types:
  - Ingress
```

生效后，就只有这一个ip可以访问到6443的端口

# 小技巧和踩坑记

## 1. 在命名空间级别设置了sidecar如何针对单独的deployment设置去除

```yaml
apiVersion: apps/v1
kind: Deployment
spec:
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: "false"  # 关键：添加此注解禁用注入
```
