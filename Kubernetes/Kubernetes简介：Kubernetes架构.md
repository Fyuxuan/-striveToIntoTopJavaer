#Kubernetes架构

Kubernetes最初源于谷歌内部的Borg，提供了面向应用的容器集群部署和管理系统。Kubernetes的目标旨在消除编排物理/虚拟计算，网络和存储基础设施的负担，并使应用程序运营商和开发人员完全将重点放在以容器为中心的原语上进行自助运营。Kubernetes 也提供稳定、兼容的基础（平台），用于构建定制化的workflows 和更高级的自动化任务。  
Kubernetes 具备完善的集群管理能力，包括多层次的安全防护和准入机制、多租户应用支撑能力、透明的服务注册和服务发现机制、内建负载均衡器、故障发现和自我修复能力、服务滚动升级和在线扩容、可扩展的资源自动调度机制、多粒度的资源配额管理能力。
Kubernetes 还提供完善的管理工具，涵盖开发、部署测试、运维监控等各个环节。

## Borg简介

Borg是谷歌内部的大规模集群管理系统，负责对谷歌内部很多核心服务的调度和管理。Borg的目的是让用户能够不必操心资源管理的问题，让他们专注于自己的核心业务，并且做到跨多个数据中心的资源利用率最大化。

Borg主要由BorgMaster、Borglet、borgcfg和Scheduler组成，如下图所示

  ![-w572](file/15662939314245.jpg)

* BorgMaster 是整个集群的大脑，负责维护整个集群的状态，并将数据持久化到 Paxos 存储中；
* Scheduer 负责任务的调度，根据应用的特点将其调度到具体的机器上去；
* Borglet 负责真正运行任务（在容器中）；
* borgcfg 是 Borg 的命令行工具，用于跟 Borg 系统交互，一般通过一个配置文件来提交任务。

## Kubernetes 特性
Kubernetes 是一种用于在一组主机上运行和协同容器化应用程序的系统，旨在提供可预测性、可扩展性与高可用性的方法来完全管理容器化应用程序和服务的生命周期的平台。用户可以定义应用程序的运行方式，以及与其他应用程序或外部世界交互的途径，并能实现服务的扩容和缩容，执行平滑滚动更新，以及在不同版本的应用程序之间调度流量以测试功能或回滚有问题的部署。 Kubernetes 提供了接口和可组合的平台原语，使得用户能够以高度的灵活性和可靠性定义及管理应用程序。简单总结起来，它具有以下几个重要特性 。

### 1. 自动装箱
建构于容器之上，基于资源依赖及其他约束自动完成容器部署且不影响其可用性，并 通过调度机制混合关键型应用和非关键型应用的工作负载于同一节点以提升资源利用率。
### 2. 自我修复（自愈）
支持容器故障后自动重启、节点故障后重新调度容器，以及其他可用节点、健康状态 检查失败后关闭容器并重新创建等自我修复机制。
### 3. 水平扩展
支持通过简单命令或 UI 手动水平扩展，以及基于 CPU 等资源负载率的自动水平扩展机制 。
### 4. 服务发现和负载均衡 
Kubernetes 通过其附加组件之一的 KubeDNS （或 CoreDNS ）为系统内置了服务发现功能，它会为每个 Service 配置 DNS 名称，并允许集群内的客户端直接使用此名称发出访问请求，而 Service 则通过 iptables 或 ipvs 内建了负载均衡机制 。
### 5. 自动发布和回滚
Kubernetes 支持“灰度”更新应用程序或其配置信息，它会监控更新过程中应用程序的健康状态，以确保它不会在同一时刻杀掉所有实例，而此过程中一旦有故障发生，就会立即自动执行回滚操作 。
### 6. 密钥和配置管理
Kubernetes 的 ConfigMap 实现了配置数据与 Docker 镜像解稠，需要时，仅对配置做出变更而无须重新构建 Docker 镜像，这为应用开发部署带来了很大的灵活性。此外，对于应用所依赖的一些敏感数据，如用户名和密码、令牌、密钥等信息， Kubernetes 专门提供了 Secret 对象为其解耦，既便利了应用的快速开发和交付，又提供了一定程度上的安全保障。
### 7. 存储编排
Kubernetes 支持 Pod 对象按需自动挂载不同类型的存储系统 ，这包括节点本地存储、公有云服务商的云存储（如 AWS 和 GCP 等），以及网络存储系统（例如， NFS、iSCSI、GlusterFS、Ceph、Cinder 和 Flocker 等）。
### 8. 批量处理执行
除了服务型应用，Kubernetes 还支持批处理作业及 CI（持续集成），如果需要， 一样可以实现容器故障后恢复。

## Kubernetes 架构

Kubernetes 借鉴了 Borg 的设计理念，比如 Pod、Service、Labels 和单 Pod 单 IP 等。Kubernetes 的整体架构跟 Borg 非常像，如下图所示

![](file/15662946967695.jpg)


Kubernetes 主要由以下几个核心组件组成：

* etcd 保存了整个集群的状态；
* kube-apiserver 提供了资源操作的唯一入口，并提供认证、授权、访问控制、API 注册和发现等机制；
* kube-controller-manager 负责维护集群的状态，比如故障检测、自动扩展、滚动更新等；
* kube-scheduler 负责资源的调度，按照预定的调度策略将 Pod 调度到相应的机器上；
* kubelet 负责维持容器的生命周期，同时也负责 Volume（CVI）和网络（CNI）的管理；
* Container runtime 负责镜像管理以及 Pod 和容器的真正运行（CRI），默认的容器运行时为 Docker；
* kube-proxy 负责为 Service 提供 cluster 内部的服务发现和负载均衡；

除了核心组件，还有一些推荐的 Add-ons：

* CoreDNS 负责为整个集群提供 DNS 服务
* Ingress Controller 为服务提供外网入口
* Heapster 提供资源监控
* Dashboard 提供 GUI
* Federation 提供跨可用区的集群
* Fluentd-elasticsearch 提供集群日志采集、存储与查询

## Kubernetes架构示意图

### 整体架构

下图清晰表明了Kubernetes的架构设计以及组件之间的通信协议。

![Kuberentes架构（图片来自于网络）](file/15663927970976.jpg)


下面是更抽象的一个视图：

![kubernetes整体架构示意图](file/15663928276452.jpg)

### Master架构

![Kubernetes master架构示意图](file/15663928485300.jpg)


### Node架构

![kubernetes node架构示意图](file/15663928706968.jpg)


### 分层架构

Kubernetes设计理念和功能其实就是一个类似Linux的分层架构，如下图所示。

![Kubernetes分层架构示意图](file/15663928960235.jpg)


* 核心层：Kubernetes最核心的功能，对外提供API构建高层的应用，对内提供插件式应用执行环境
* 应用层：部署（无状态应用、有状态应用、批处理任务、集群应用等）和路由（服务发现、DNS解析等）、Service Mesh（部分位于应用层）
* 管理层：系统度量（如基础设施、容器和网络的度量），自动化（如自动扩展、动态Provision等）以及策略管理（RBAC、Quota、PSP、NetworkPolicy等）、Service Mesh（部分位于管理层）
* 接口层：kubectl命令行工具、客户端SDK以及集群联邦
* 生态系统：在接口层之上的庞大容器集群管理调度的生态系统，可以划分为两个范畴
* Kubernetes外部：日志、监控、配置管理、CI/CD、Workflow、FaaS、OTS应用、ChatOps、GitOps、SecOps等
* Kubernetes内部：[CRI](cri.md)、[CNI](cni.md)、[CSI](csi.md)、镜像仓库、Cloud Provider、集群自身的配置和管理等