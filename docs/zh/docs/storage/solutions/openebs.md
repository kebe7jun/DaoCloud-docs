# OpenEBS 存储方案

DCE 5.0 支持众多第三方存储方案，我们针对 OpenEBS 进行了相关测试，并最终将其作为 Addon 集成了应用商店中。
以下是对 OpenEBS 的调研和测评。

有关应用商店 Addon 的图形化界面安装、部署、卸载等操作说明，将于稍后提供。

## 什么是 CAS

[容器附加存储 (CAS) ](../terms/cas.md)是一种包含由 Kubernetes 编排的基于微服务的存储控制器的软件。
这些存储控制器可在 Kubernetes 能运行的任何地方运行，包括任何云、裸机服务器或传统共享存储系统之上。
至关重要的是，数据本身也可以通过容器访问，而不是存储在平台外共享的横向扩展存储系统中。

![openebs-1](../images/openebs-1.svg)

CAS 是一种非常符合非聚合数据的趋势以及运行小型、松散耦合工作负载的小型自治团队。
开发人员能保持自主并可以使用 Kubernetes 集群可用的任何存储来启动他们自己的 CAS 容器。

CAS 也反映了更广泛的解决方案趋势：通过在 Kubernetes 和微服务上构建并为基于 Kubernetes 的微服务环境提供功能来重塑特定类别的资源或创建新类别资源。
例如，安全、DNS、网络、网络策略管理、消息传递、跟踪、日志记录等新项目已经出现在云原生生态系统中。

### CAS 的优势

#### 敏捷

CAS 中的每个存储卷都有一个容器化存储控制器和对应的容器化副本。
因此，围绕这些组件的资源维护和调整是真正敏捷的。
Kubernetes 的滚动升级能力可以实现存储控制器和存储副本的无缝升级。
CPU 和内存等资源可以使用容器 cGroups 进行调整。

#### 存储策略的粒度

将存储软件容器化并将存储控制器专用于每个卷，可以最大程度地细化存储策略。
使用 CAS 架构，您可以在每个卷的基础上配置所有存储策略。
此外，您可以监控每个卷的存储参数并动态更新存储策略以获得每个工作负载的预期结果。
存储吞吐量、IOPS 和延迟的控制随着卷存储策略中粒度的增加而增加。

#### 避免锁定

避免云供应商锁定是许多 Kubernetes 用户的共同目标。
然而，有状态应用程序的数据通常仍然依赖于云提供商和技术，或者依赖于底层的传统共享存储系统、NAS 或 SAN。
使用 CAS 方法，存储控制器可以根据工作负载在后台迁移数据，实时迁移变得更加简单。
换句话说，CAS 的控制粒度以一种无中断的方式简化了有状态工作负载从一个 Kubernetes 集群到另一个集群的移动。

#### 云原生

CAS 将存储软件容器化，并使用 Kubernetes 自定义资源定义 (CRD) 来表示底层存储资源，例如磁盘和存储池。
该模型使存储能够无缝集成到其他云原生工具中。
可以使用 Prometheus、Grafana、Fluentd、Weavescope、Jaeger 等云原生工具来配置、监控和管理存储资源。

与超融合系统类似，CAS 中卷的存储和性能是可扩展的。
由于每个卷都有自己的存储控制器，因此存储可以在节点存储容量的允许范围内扩展。
随着给定 Kubernetes 集群中容器应用程序数量的增加，将添加更多节点，从而提高存储容量和性能的整体可用性，从而使存储可用于新的应用程序容器。
这种可扩展性过程类似于 Nutanix 等成功的超融合系统。

## OpenEBS 原理及架构

OpenEBS 是容器附加存储 (CAS) 模式的领先开源实现。
作为这种方法的一部分，OpenEBS 使用容器来动态配置卷并提供高可用性等数据服务。
OpenEBS 依赖并扩展 Kubernetes 本身来编排其卷服务。

![openebs-2](../images/openebs-2.svg)

OpenEBS 有很多组件，可以分为以下两大类：

- OpenEBS 数据引擎
- OpenEBS 控制平面

### 数据引擎

数据引擎是 OpenEBS 的核心，负责代表它们所服务的有状态工作负载对底层持久存储执行读写操作。

数据引擎负责：

- 聚合分配给它们的块设备中的可用容量，然后为应用程序划分卷。
- 提供标准系统或网络传输接口 (NVMe/iSCSI) 以连接到本地或远程卷
- 提供卷服务，如同步复制、压缩、加密、维护快照、访问数据的增量或完整快照等
- 在将数据持久化到底层存储设备的同时提供强一致性

OpenEBS 遵循微服务模型来实现数据引擎，其中功能进一步分解为不同的层，允许灵活地交换层并使数据引擎为未来的应用程序和数据中心技术的变化做好准备。

OpenEBS 数据引擎由以下几层组成：

![openebs-3](../images/openebs-3.svg)

#### 卷访问层

有状态工作负载使用标准的 POSIX 兼容机制来执行读写操作。
根据工作负载的类型，应用程序可能更喜欢直接对原始块设备或使用标准文件系统（如 XFS、Ext4）执行读取和写入。

CSI 节点驱动程序或 Kubelet 将负责将卷附加到 Pod 运行所在的所需节点，必要时进行格式化并挂载文件系统以供 Pod 访问。
用户可以选择在此层设置挂载选项和文件系统权限，这将由 CSI 节点驱动程序或 kubelet 执行。

附加卷（使用本地、iSCSI 或 NVMe）和安装（Ext4、XFS 等）所需的详细信息可通过持久卷规范获得。

#### 卷服务层

该层通常称为 Volume Target Layer 甚至 Volume Controller 层，因为它负责提供逻辑卷。
应用程序读取和写入是通过 Volume Targets 执行的 - 它控制对卷的访问、将数据同步复制到集群中的其他节点，
并帮助决定哪个副本充当主副本并促进将数据重建到旧的或重新启动的副本。

数据引擎用于提供高可用性的实现模式是 OpenEBS 与其他传统存储控制器的区别。
与使用单个存储控制器在多个卷上执行 IO 不同，OpenEBS 为每个卷创建一个存储控制器（称为 Target/Nexus），并具有将保存卷数据的特定节点列表。
每个节点将具有使用同步复制分发的卷的完整数据。

使用单个控制器将数据同步复制到固定的节点集（而不是通过多个元数据控制器分发），减少了管理元数据的开销，还减少了与节点故障和参与重建的其他节点相关的爆炸半径失败的节点。

OpenEBS 卷服务层将卷公开为：

- 本地 PV 情况下的设备或目录路径，
- cStor 和 Jiva 情况下的 iSCSI 目标
- NVMe Target 在 Mayastor 的情况下

#### 卷数据层

OpenEBS 数据引擎在存储层之上创建卷副本。卷副本被固定到一个节点，并在存储层之上创建。
副本可以是以下任何一项：

- 子目录 - 如果使用的存储层是文件系统目录
- 完整设备或分区设备 - 如果使用的存储层是块设备
- 逻辑卷 - 如果使用的存储层是来自 LVM 或 ZFS 的设备池。
- 如果应用程序只需要本地存储，那么将使用上述目录、设备（或分区）或逻辑卷之一创建持久卷。
   OpenEBS 控制平面将用于提供上述副本之一。

OpenEBS 可以使用其复制引擎之一——Jiva、cStor 和 Mayastor，在本地存储之上添加高可用性层。
在这种情况下，OpenEBS 使用轻量级存储定义的存储控制器软件，该软件可以通过网络端点接收读/写操作，
然后传递到底层存储层。OpenEBS 然后使用此副本网络端点来跨节点维护卷的同步副本。

OpenEBS 卷副本通常经历以下状态：

- 正在初始化，在初始配置期间正在注册到其卷
- 健康，当副本可以参与读/写操作时
- 离线，当副本所在的节点或存储发生故障时
- 重建，当节点或存储故障已得到纠正并且副本正在从其他健康副本接收数据时
- 终止，当卷被删除并且副本被删除并且空间被回收时

#### 存储层

存储层构成了持久化数据的基本构建块。存储层由连接到节点的块设备组成（通过 PCIe、SAS、NVMe 在本地或通过远程 SAN/云）。
存储层也可以是挂载文件系统之上的子目录。

存储层不在 OpenEBS 数据引擎的范围内，可用于使用标准操作系统或 Linux 软件结构的 Kubernetes 存储结构。

数据引擎将存储用作设备或设备池或文件系统目录。

### 控制平面

OpenEBS 上下文中的控制平面是指部署在集群中的一组工具或组件，它们负责：

- 管理 kubernetes 工作节点上可用的存储
- 配置和管理数据引擎
- 与 CSI 接口以管理卷的生命周期
- 与 CSI 和其他工具进行接口，执行快照、克隆、调整大小、备份、恢复等操作。
- 集成到其他工具中，如 Prometheus/Grafana 以进行遥测和监控
- 集成到其他工具中进行调试、故障排除或日志管理

OpenEBS 控制平面由一组微服务组成，这些微服务本身由 Kubernetes 管理，使 OpenEBS 真正成为 Kubernetes 原生的。
由 OpenEBS 控制平面管理的配置被保存为 Kubernetes 自定义资源。控制平面的功能可以分解为以下各个阶段：

![openebs-4](../images/openebs-4.svg)

#### YAML 或 Helm 图表

管理员可以使用高度可配置的 Helm chart 或 kubectl/YAML 安装 OpenEBS 组件。
OpenEBS 安装也通过管理 Kubernetes 产品支持，例如 OpenShift、EKS、DO、Rancher 作为市场应用程序或作为附加组件或插件紧密集成到 Kubernetes 发行版中，例如 MicroK8s、Kinvolk、Kubesphere。

作为 OpenEBS 安装的一部分，所选数据引擎的控制平面组件将使用标准 Kubernetes 原语（如 Deployments、DaemonSets、Statefulsets 等）安装为集群和/或节点组件。
OpenEBS 安装还负责将 OpenEBS 自定义资源定义加载到 Kubernetes 中。

OpenEBS 控制平面组件都是无状态的，依赖于 Kubernetes etcd 服务器（自定义资源）来管理其内部配置状态并报告各种组件的状态。

#### 声明式 API

OpenEBS 支持声明式 API 来管理其所有操作，并且 API 公开为 Kubernetes 自定义资源。
Kubernetes CRD 验证器和 admission webhooks 用于验证用户提供的输入并验证操作是否被允许。

声明式 API 是 Kubernetes 管理员和用户习惯的自然扩展，他们可以在其中通过 YAML 定义意图，然后 Kubernetes 和相关的 OpenEBS 操作员将状态与用户的意图相协调。

声明式 API 可用于配置数据引擎和设置卷配置文件/策略。
甚至数据引擎的升级也是使用此 API 执行的。这些 API 可用于：

- 管理每个数据引擎的配置
- 管理存储需要管理的方式或存储池
- 管理卷及其服务 - 创建、快照、克隆、备份、恢复、删除
- 管理池和卷的升级

#### 数据引擎 Operator

从发现底层存储到创建池和卷的所有数据引擎操作都打包为 Kubernetes Operators。
每个数据引擎要么在安装期间提供的配置之上运行，要么通过相应的 Kubernetes 自定义资源进行控制。

数据引擎 Operator 可以在集群范围内或在特定节点上操作。
集群范围操作员通常参与涉及与 Kubernetes 组件交互的操作 - 协调各种节点上池和卷的调度或迁移。
节点级运算符对本地操作进行操作，例如在节点上可用的存储或池上创建卷、副本、快照等。

数据引擎 Operator 通常也被称为数据引擎的控制平面，因为它们有助于管理相应数据引擎提供的卷和数据服务。
根据提供或需要的功能，某些数据引擎（如 cstor、jiva 和 mayastor）可以有多个 Operator，其中本地卷操作可以直接嵌入到相应的 CSI 控制器/配置器中。

#### CSI 驱动程序（动态卷配置器）

CSI 驱动程序充当管理 Kubernetes 中卷生命周期的促进者。
CSI 驱动程序操作由 StorageClass 中指定的参数控制或自定义。CSI 驱动程序由三层组成：

- Kubernetes 或 Orchestrator 功能 - Kubernetes 原生并将应用程序绑定到卷
- Kubernetes CSI 层 - 将 Kubernetes 本机调用转换为 CSI 调用 - 以标准方式将用户提供的信息传递给 CSI 驱动程序
- 存储驱动程序——它们是 CSI 投诉的，与 Kubernetes CSI 层密切合作以接收请求并处理它们。
- 存储驱动程序负责：

公开数据引擎的功能：

- 直接与数据引擎或数据引擎操作员交互以执行卷创建和删除操作
- 与数据引擎接口以将卷附加/分离到使用卷的容器正在运行的节点
- 与标准 linux 实用程序接口以格式化、安装/卸载卷到容器

#### Plug-in 插件

OpenEBS 专注于存储操作，并为其他流行工具提供插件，以执行核心存储功能之外的操作，但对于在生产中运行 OpenEBS 非常重要。
此类操作的示例包括：

- 应用程序一致的备份和恢复（通过集成到 Velero 中提供）
- 监控和警报（通过集成到 Prometheus、Grafana、警报管理器中提供）
- 执行安全策略（通过与 PodSecurityPolicies 或 Kyerno 的集成提供）
- 日志记录（通过与 ELK、Loki、Logstash 等管理员设置的任何标准日志记录堆栈集成提供）
- 可视化（通过标准 Kubernetes 仪表板或自定义 Grafana 仪表板提供）

#### CLI 命令行

OpenEBS 上的所有管理功能都可以通过 kubectl 执行，因为 OpenEBS 使用自定义资源来管理其所有配置并报告组件的状态。

此外，OpenEBS 还发布了 alpha 版 kubectl 插件，以帮助使用单个命令提供有关池和卷的信息，该命令聚合了通过多个 kubectl 命令获得的信息。
