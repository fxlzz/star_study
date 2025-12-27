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
+ ClusterRole： 定义集群级别的一组角色权限
+ ClusterRoleBinding： 将角色绑定在某个集群上

**命名空间**
**Pod**： 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元（所有的命令，都会被封装成Pod 去执行）。是一组（一个或多个） 容器
	Pod 中的内容总是一同调度，在共享的上下文中运行，文件系统、网络等都是共享的。 Pod 所建模的是特定于应用的“逻辑主机”，其中包含一个或多个应用容器， 这些容器相对紧密地耦合在一起。 
	--> 最好是 “one-container-per-Pod” 一个 Pod 一个容器的模式

**副本集**（replicas）： 根据 PodTemplate 来创建几个Pod

**控制器**： 可以理解为更为抽象的 Pod ，管理者 Pod 的创建
	- 适用于**无状态服务**
		- ~~RC: ReplicationController(已废除)~~
		- RS: ReplicaSet --> 可用 Selector 指定用哪个 **Pod** 的模板进行 自动扩容和缩容
		- **Deployment** --> 可以认为这是对 RS 更上一层的封装
			- 创建 ReplicaSet / Pod（自动创建 RS）
			- 滚动升级 / 回滚  --> 类似于静默升级吧，用户是无感知的
			- 平滑扩容 / 缩容
			- 暂停与恢复 deployment
	- 适用于**有状态服务**（持久化数据、顺序访问）
		- StatefulSet --> 每个 Pod 的DNS格式为 `statefulSetName-(0...N-1).serviceName.namespace.svc.cluster.local`
			- Headless Service: 来做 dns 服务【用 Pod 的服务名 当作 域名】、顺序访问【数据库架构相关，例如，mysql 主从架构，需要有一个主节点、其他从节点，这就需要有顺序 -> 顺序通过 数字 来保证】
			- volumeClaimTemplate： 在 k8s 集群中申请一块空间，用来做持久化存储
	- 守护进程
		- DaemonSet： 类似于横向抽离，在符合标签的节点中，创建 Pod。
	- 任务 / 定时任务
		- Job
		- CronJob

![](assets/k8s/file-20251221174359994.png)

**服务发现**:
	- Service： 横向流量(东西流量)
	- Ingress： 纵向流量(南北流量)
![500](assets/k8s/file-20251221220330828.png)


节点之间：Service 服务，进行通信
外部： Ingress 服务，提供接口通信
![](assets/k8s/file-20251221222935926.png)

注意： 同一个 Node 之下的 Pod 是可以互相通信的，Pod 之下的容器之间，所有的资源（网络、文件系统都是共享的）

不同 Node 之前，通信需要端口映射 --> Service 服务，可以将 Pod 容器中的服务映射到另一个端口，其他 Node 中的服务可以通过这个外部映射端口，访问到Node，从而访问到 Pod 中的服务。

就像上面的图示：
order 服务要访问到 product 服务，需要访问 product 的service服务中暴露的端口和服务名
	-> product-svc:2088

# k8s 基本操作
## 操作资源

**资源类型与别名**
+ pods --> po 
+ deployment --> deploy
+ services  --> svc
+ namespace --> ns
+ nodes --> no

**格式化输出**
可指定以什么格式对某个资源进行输出
+ 输出 json 格式 --> -o json
+ 仅打印资源名称 --> -o name
+ 以纯文本的格式输出所有信息 --> -o wide
+ 输出 yaml 格式 --> -o yaml
例如，
我需要知道 deployment  资源的详细信息
`kubectl get deploy -o wide`
也可以获取某个资源下的信息，以 yaml 格式输出
`kubectl get deploy nginx -o yaml`

**获取资源信息**
`kubectl get xxx --> xxx 指的有 pod、deployment、namespace等` 详细信息看上一节资源和对象

要记住，k8s的世界中，一切都是资源

**删除资源**
`kubectl delete 哪类资源 资源中的具体名称`
例如，
删除由 deployment 创建出来的 nginx 信息
`kubectl delete deploy nginx` 
+ 这里 deploy 是 deployment 的缩写
+ nginx 是 deployment 中的名字

**创建资源**
`kubectl create -f xxx.yaml` -> 通过 yaml 配置文件的形式进行资源的创建

**查看某个资源的详细信息**
`kubctl describe 哪类资源 资源中的名字`
例如，
查看 pod 中的 nginx-demo（pod 的名称） 的信息
`kubectl describe po nginx-demo`


