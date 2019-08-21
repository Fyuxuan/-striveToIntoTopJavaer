# Kubernetes简介：核心资源组件&核心组件

Kubernetes 使用共享网络将多个物理机或虚拟机汇集到一个集群中，在各服务器之间进行通信，该集群是配置 Kubernetes 的所有组件、功能和工作负载的物理平台。 集群中一台服务器（或高可用部署中的一组服务器）用作 Master ，负责管理整个集群，余下的其他机器用作 Worker Node ，它们是使用本地和外部资源接收和运行工作负载的服务器。

## Master
Master 是集群的网关和中枢，负责为用户和客户端暴露 API ，跟踪其他服务器的健 康状态、以最优方式调度工作负载，以及编排其他组件之间的通信等任务，它是用户或客户端与集群之间的核心联络点，并负责 Kubernetes 系统的大多数集中式管控逻辑。 单个 Master 节点即可完成其所有的功能，但出于冗余及负载均衡等目的，生产环境中通常需要协同部署多个此类主机。

## Node
Node 是 Kubernetes 集群的工作节点，负责接收来自 Master 的工作指令并根据指令相应地创建或销毁 Pod 对象，以及调整网络规则以合理地路由和转发流量等。 理论上讲，Node 可以是任何形式的计算设备，不过 Master 会统一将其抽象为 Node 对象进行管理。

Kubernetes 将所有 Node 的资源集结于一处形成一台更加强大的“服务器”，在用户将应用部署于其上时，Master 会使用调度算法将其自动指派至某个特定的 Node 运行。在 Node 加入集群或从集群中移除时 Master 也会按需重新编排影响到的 Pod（容器）。于是，用户无须关心其应用究竟运行于何处。

从抽象的视角来讲， Kubernetes 还有着众多的组件来支撑其内部的业务逻辑，包括运行应用、应用编排、服务暴露、应用恢复等，它们在 Kubernetes 中被抽象为Pod、Label、Annotation、Ingress、Service、Controller等资源对象。

## 资源对象

### Pod
Kubernetes 并不直接运行容器，而是使用一个抽象的资源对象来封装一个或者多个容器，这个抽象为 Pod ，它也是 Kubernetes 的最小调度单元。同一 Pod 中的容器共享网络名称空间和存储资源，这些容器可经由本地回环接口 lineout 直接通信，但彼此之间又在 Mount、User 及 PID 等名称空间上保持了隔离。尽管 Pod 中可以包含多个容器，但是作为最小调度单元，它应该尽可能地保持“小”，即通常只应该包含一个主容器。

### Label 资源标签
标签（ Label ）是将资源进行分类的标识符，资源标签其实就是一个键值型（ key/values) 数据。标签旨在指定对象（如 Pod 等）辨识性的属性，这些属性仅对用户存在特定的意义，对 Kubernetes 集群来说并不直接表达核心系统语义。标签可以在对象创建时附加其上，并能够在创建后的任意时间进行添加和修改。一个对象可以拥有多个标签，一个标签也可以附加于多个对象（通常是同一类对象）之上

### 标签选择器（ Selector ）
标签选择器（ Selector ）全称为 " Label Selector" 它是一种根据 Label 来过滤符合条件的资源对象的机制。 例如，将附有标签“ role:backend ”的所有 Pod 对象挑选出来归为一组就是标签选择器的一种应用，用户通常使用标签对资源对象进行分类，而后使用标签选择器挑选出它们，直接批量给挑选出来的资源对象做相应的操作。
### Pod 控制器
尽管 Pod 是 Kubernetes 的最小调度单元，但用户通常并不会直接部署及管理 Pod 对象，而是要借助于另一类抽象——控制器（Controller）对其进行管理。 用于工作负载的控制器是一种管理 Pod 生命周期的资源抽象，它们是 Kubernetes 上的一类对象，而非单个资源对象，包 括 ReplicationController、ReplicaSet、 Deployment、StatefulSet、Job 等 。 以 Deployment 控制器为例，它负责确保指定的 Pod 对象的副本数量精确符合定义，否则“多退少补”。使用控制器之后就不再需要手动管理 Pod 对象了，用户只需要声明应用的期望状态，控制器就会自动对其进行进程管理。
### 服务资源（ Service )
Service 是建立在一组 Pod 对象之上的资源抽象，它通过标签选择器选定一组 Pod 对象， 并为这组 Pod 对象定义一个统一的固定访问入口（通常是一个 IP 地址），若 Kubernetes 集群存在 DNS 附件，它就会在 Service 创建时为其自动配置一个 DNS 名称以便客户端进行服务发现。 到达 Service IP 的请求将被负载均衡至其后的端点——各个 Pod 对象之上，因此 Service 从本质上来讲是一个四层代理服务。 另外， Service 还可以将集群外部流量引入到集群中来 。
### 存储卷
存储卷（ Volume ）是独立于容器文件系统之外的存储空间，常用于扩展容器的存储空间并为它提供持久存储能力。Kubernetes 集群上的存储卷大体可分为临时卷 、本地卷和网络卷。临时卷和本地卷都位于 Node 本地，一旦 Pod 被调度至其他 Node，此种类型的存储卷将无法访问到，因此临时卷和本地卷通常用于数据缓存，持久化的数据则需要放置于持久卷 (persistent volume ）之上。
### Name 和 Namespace
名称（ Name ）是 Kubernetes 集群中资源对象的标识符，它们的作用域通常是名称空间( Namespace ）因此名称空间是名称的额外的限定机制。在同一个名称空间中，同一类型资源对象的名称必须具有唯一性。名称空间通常用于实现租户或项目的资源隔离，从而形成逻辑分组。创建资源对象未指定名称空间时 ，它们都属于默认的名称空间“ default ” 。
### Annotation
Annotation（注解）是另一种附加在对象之上的键值类型的数据，但它拥有更大的数据容量。Annotation常用于将各种非标识型元数据（ metadata ） 附加到对象上，但它不能用于标识和选择对象，通常也不会被 Kubernetes 直接使用，其主要目的是方便工具或用户的阅读及查找等
### Ingress
Kubernetes 将 Pod 对象和外部网络环境进行了隔离，Pod 和 Service 等对象间的通信都使用其内部专用地址进行，如若需要开放某些 Pod 对象提供给外部用户访问，则需要为其请求流量打开一个通往 Kubernetes 集群内部的通道，除了Service之外，Ingress 也是这类通道的实现方式之一。

## Kubernetes集群组件
一个典型的 Kubernetes 集群由多个工作节点和一个或多个Master节点，以及一个集群状态存储系统（ etcd ）组成。其中 Master 节点负责整个集群的管理工作， 为集群提供管理接口，并监控和编排集群中的各个工作节点。各个工作节点负责以 Pod 的形式运行容器，因此，各节点需要事先配置好容器运行依赖的所有服务和资源，如容器运行时环境（比如Docker）。

Master 节点主要是由 apiserver、 controller-manager 和 scheduler 三个组件，以及一个用于集群状态存储的 etcd 存储服务组成，而每个 Node 节点则主要包含 kubelet、kube-proxy 及容器引擎（ Docker是最为常用的实现）等组件。此外，完整的集群服务还依赖于一些附加组件，如 KubeDNS、Dashboard、Heapster、Ingress Controller 等。

### API Server
API Server 负责输出 RESTful 风格的 Kubernetes API ，它是发往集群的所有 REST 操作命令的接入点，并负责接收、校验并响应所有的 REST 请求，结果状态被持久存储于 etcd 中。因此，API Server 是整个集群的网关。
### 集群状态存储（etcd）
Kubernetes 集群的所有状态信息都需要持久存储于存储系统 etcd 中，不过， etcd 是由 CoreOS 基于 Raft 协议开发的分布式键值存储，可用于服务发现、共享配置以及一致性保障（如数据库主节点选择、分布式锁等）。因此，etcd 是独立的服务组件，并不隶属于 Kubernetes 集群自身。生产环境中应该以 etcd 集群的方式运行以确保其服务可用性。

etcd 不仅能够提供键值数据存储， 而且还为其提供了监昕（ watch ）机制，用于监听和推送变更。 Kubernetes 集群系统中，etcd 中的键值发生变化时会通知到 API Server ，并由其通过 watchAPI 向客户端输出。基于 watch 机制， Kubernetes 集群的各组件实现了高效协同 。
### 控制器管理器（ Controller Manager )
Kubernetes 中，集群级别的大多数功能都是由几个被称为控制器的进程执行实现的，这几个进程被集成于 kube-controller-manager 守护进程中。由控制器完成的功能主要包括生命周期功能和 API 业务逻辑，具体如下。
- 生命周期功能：包括 Namespace 创建和生命周期、 Event 垃圾回收、 Pod 终止相关的垃圾回收、级联垃圾回收及 Node 垃圾回收等 。
- API业务逻辑：例如，由 ReplicaSet 执行的 Pod 扩展等 。

### 调度器（ Scheduler )
Kubernetes 是用于部署和管理大规模容器应用的平台，根据集群规模的不同，其托管运行的容器很可能会数以千计甚至更多。 API Server 确认 Pod 对象的创建请求之后， 便需要由 Scheduler 根据集群内各节点的可用资源状态，以及要运行的容器的资源需求做出调度决策。另外，Kubernetes 还支持用户自定义调度器

### kubelet
kubelet 是运行于工作节点之上的守护进程，它从 API Server 接收关于 Pod 对象的配置信息并确保它们处于期望的状态。 kubelet 会在 API Server 上注册当前工作节点， 定期向 Master 汇报节点资源使用情况，并通过 cAdvisor 监控容器和节点的资源占用状况 。
### 容器运行时环境
每个 Node 都要提供一个容器运行时（ Container Runtime ）环境， 它负责下载镜像并运行容器。 kubelet 并未固定链接至某容器运行时环境，而是以插件的方式载入配置的容器环境。这种方式清晰地定义了各组件的边界。目前，Kubernetes 支持的容器运行环境至少包括Docker、RKT、cri-o 和 Fraki等。
### kube-proxy
每个工作节点都需要运行一个 kube-proxy 守护进程，它能够按需为 Service 资源对象生成 iptables 或 ipvs 规则，从而捕获访问当前 Service 的ClusterIP 的流量并将其转发至正确的后端 Pod 对象。
### KubeDNS 
KubeDNS 是在 Kubernetes 集群中调度运行提供 DNS 服务的 Pod，同一集群中的其他 Pod 可使用此 DNS 服务解决主机名。 Kubernetes 自 1.11 版本开始默认使用 CoreDNS 项目为集群提供服务注册和服务发现的动态名称解析服务，之前的版本中用到的是 kube-dns 项目，而 SkyDNS 则是更早一代的项目。
### Kubernetes Dashboard
Kubernetes 集群的全部功能都要基于 Web 的 UI，来管理集群中的应用甚至是集群自身。
### Heapster
容器和节点的性能监控与分析系统，它收集并解析多种指标数据，如资源利用率、生命周期事件等。新版本的 Kubernetes 中，其功能会逐渐由 Prometheus 结合其他组件所取代。
### Ingress Controller
Service 是一种工作于传统层的负载均衡器，而 Ingress 是在应用层实现的 HTTP (s）负载均衡机制。不过，Ingress 资源自身并不能进行“流量穿透”，它仅是一组路由规则的集合，这些规则需要通过 Ingress 控制器（ Ingress Controller) 发挥作用 。 目前，此类的可用项目有 Nginx、Traefik、Envoy 及 HAProxy 等。