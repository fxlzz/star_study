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

## 核心概念
### 服务分类
分成两种类别： 
+ 有状态
+ 无状态

区分他们两个，可以从函数式编程的视角来看
**有状态**： 不是纯函数，会对环境产生依赖。例如，redis 服务需要存储东西到服务器上，作持久化数据。如果进行扩容，需要将上一台服务器上的数据，都重新复核一遍到扩容服务器。如果没有复刻，那么该服务可能不能正常提供服务。

**无状态**： 是纯函数，不会因为外部的改动而改动。无论是扩容、缩容都对这个服务没有影响，提供稳定的服务。 例如，nginx等服务。

### 资源和对象
在 k8s 的世界中，一切都是被抽象为 “资源”，例如，Pod、Node等，会有一个东西来描述 这些资源，也就是“资源清单”。

“资源清单”：其实就是用 json /yaml 格式来描述资源 --> 定义“类”的模样

其实，本质上，资源是一个抽象的概念，可以类似于 “类”。 对象就是这个 “类” 创建出来的实例。

#### 对象的规约和状态
**规约**： 在配置 资源清单 时，会描述某个对象的 **理想** 情况。这个理想情况，就是规约
**状态**： 由 k8s 维护的状态，是 对象真实的正真的运行时状态。k8s 会尽可能的将目前状态向 规约状态靠近。

#### 资源的分类
+ 集群级： k8s 集群
+ 命名空间级： 在集群内部，在细分为多个命名空间
+ 元数据级： 主要作用于 Pod

![](assets/k8s/file-20251221154543815.png)

具体来说：

**元数据级**：
+ HPA：Horizontal Pod Autoscaler  --> Pod 自动扩容，可根据 CPU 使用率或自定义指标自动对 Pod 进行扩容 / 缩容，控制管理器每隔 30s 查询 metrics 的资源使用情况
+ PodTemplate： Pod 的描述 --> 类似于 Pod 的模板
+ LimitRange： 对集群内的 Request 和 Limits  的配置做一个全局的统一的限制 --> 批量设置了某一个命名空间的 Pod 资源使用限制

**集群级**：
+ Namespace： 命名空间
+ Node： 节点
+ ClusterRole： 角色权限控制
+ ClusterRoleBinding： 将角色绑定在某个集群上

**命名空间**
+ Pod 
