## 认识 k8s
Kubernetes 是 Google 一个开源的容器编排引擎，用来对容器化应用进行自动化部署、扩缩和管理

为什么需要 k8s 技术呢？
在 docker 等容器化部署技术愈发成熟，一台服务器上部署多个容器都是没问题的。那么这里就有一个问题，一台服务器上部署多个 docker 容器，还能用 docker compose 来管理。

在企业中，多台服务器上部署的多个容器呢？ 上压力了，就需要用到 K8s 来统一管理分布在不同服务器上的不同容器。

比如呈现下面的架构：

![500](assets/k8s/file-20251221101015104.png)


## k8s 的架构
采用 主-从 的形式，主节点用来管理外部的请求和从节点的调度

![](assets/k8s/file-20251221105831394.png)

### Master
[kube-apiserver](https://kubernetes.io/zh-cn/docs/concepts/architecture/#kube-apiserver)
公开 Kubernetes HTTP API 的核心组件服务器。

[etcd](https://kubernetes.io/zh-cn/docs/concepts/architecture/#etcd)
具备一致性和高可用性的键值存储，用于所有 API 服务器的数据存储。

[kube-scheduler](https://kubernetes.io/zh-cn/docs/concepts/architecture/#kube-scheduler)
查找尚未绑定到节点的 Pod，并将每个 Pod 分配给合适的节点。

[kube-controller-manager](https://kubernetes.io/zh-cn/docs/concepts/architecture/#kube-controller-manager)
运行[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)来实现 Kubernetes API 行为。

[cloud-controller-manager](https://kubernetes.io/zh-cn/docs/concepts/architecture/#cloud-controller-manager) (optional)
与底层云驱动集成
### Node
[kubelet](https://kubernetes.io/zh-cn/docs/concepts/architecture/#kubelet)
确保 Pod 及其容器正常运行。

[kube-proxy](https://kubernetes.io/zh-cn/docs/concepts/architecture/#kube-proxy)（可选）
维护节点上的网络规则以实现 Service 的功能。

[容器运行时（Container runtime）](https://kubernetes.io/zh-cn/docs/concepts/architecture/#container-runtime)
负责运行容器的软件，阅读[容器运行时](https://kubernetes.io/zh-cn/docs/setup/production-environment/container-runtimes/)以了解更多信息。

注意：
+ api-server 是执行操作的核心，k8s 的所有操作都会先请求 api-server，然后等响应
+ 通过 kubectl --> 请求接口，发送请求 --> apiserver --> 最终作用于 Pod
+ Node 是一台服务器 -> 下面可运行多个 Pod -> Pod 可运行多个容器

### 其他组件
+ kube-dns： dns解析
+ ingress Controller： 外网访问
+ Prometheus： 资源监控
+ Dashboard： 图形化界面
+ Federation： 集群间的调度
+ Fluentd-elasticsearch： 日志采集
