# eBPF 和可观测性，不得不说的关系

**![图片](images/ebpf.png)**

当下，eBPF 技术无疑是最火的技术之一，它大大提高了云原生网络、安全和可观测性的能力。其中，提起可观测性，我们首先又会想到 OpenTelemetry。那么 eBPF 是如何运作在可观测性中的呢？三者之间又有什么样的关系？让我们来看一看。

------

## 什么是 eBPF

**eBPF 是一种无需更改 Linux 内核代码，便能让程序在内核中运行的技术。**开发者可以通过执行 eBPF 程序，给运行中的操作系统添加额外的能力。这催生了很多基于 eBPF 的项目，涵盖了广泛用例，包括云原生网络、安全和可观测性。例如：当下正流行的 Cilium ，是基于eBPF 实现数据转发的 CNI 网络插件；Falco 是 CNCF 开源孵化的运行时安全工具，专门为 Kubernetes 、Linux 和云原生构建；Pixie 使用 eBPF 自动收集遥测数据，也已开源应用，并进入了 CNCF 沙箱的可观测项目；同时一些服务网格产品也在探索使用 eBPF。**eBPF 似乎改变了竞争环境。**

## **两者的关系 **

为什么 eBPF 与可观测性相关？

传统意义上的观测性，是指在外部洞悉应用程序运行状况的能力。而 eBPF 是一种**无需入侵应用代码，直接向操作系统内核层添加黑盒代码**的革命性技术。这种查看内核中的操作，却不会 “干扰” 应用程序或内核本身的技术，从而使得 eBPF 获得可观测性强大能力。

因此使用 eBPF，即使不依赖操作系统公开的固定指标，我们也能直接从内核中收集和聚合自定义指标，并根据各种可能性来源，生成可见性事件。通过这种方式，我们将**可见性扩展到内核**，甚至可以**通过仅收集所需的可见性数据，实现整体系统开销的降低。**

### eBPF 和 OpenTelemetry 之间有什么关系？

让我们从 OpenTelemetry 开始。

OpenTelemetry 是 CNCF 的开源项目。它为可观测性带来了新的标准规范，用于生成和采集日志、指标和链路追踪数据。OpenTelemetry 能检测分布式服务，从系统发生事件中收集遥测数据，帮助我们了解程序和软件的性能和行为。因此，与 eBPF 类似，OpenTelemetry 也能使我们获得可观测性，但两者之间并不可以完全替代。

我们可以从两个角度来对比：

- 遥测数据的收集
- 遥测数据格式规范

**遥测数据的收集：OpenTelemetry 和 eBPF 部分重叠**

OpenTelemetry 和 eBPF 都允许我们收集遥测数据 (OpenTelemetry，使用 OTEL SDK)。二者收集遥测数据的方法各有取舍：

- eBPF 由于内核的邻近性，在收集操作系统指标并生成深度分析和数据可见性时，功能尤为强大。
- 而 OpenTelemetry SDK 部署在应用代码中，它更易于开发与维护，同时在分布式系统中更方便实现端到端追踪。

对于开发人员来说，他们更熟悉在应用程序中修改遥测代码，而 eBPF 是一项全新的技术，则需要更高的使用门槛与学习成本。

**遥测数据格式规范：OpenTelemetry 是标准**

尽管收集遥测数据的方法有很多种，但事实上 OpenTelemetry 已经成为了遥测数据格式的唯一标准。

选择采用 eBPF 来收集遥测数据时，也应使用 OpenTelemery 规范中的数据格式，让 eBPF 可以与 OpenTelemery 的组件无缝集成，从而将遥测数据的范围从应用层扩展到了内核层。

### OpenTelemetry + eBPF 的结论

OpenTelemetry 和 eBPF **在可观测性领域有一些重叠的功能，他们适用于不同的应用场景。**

比如，我们希望实现分布式跟踪时，建议使用 OpenTelemetry SDK，而 eBPF 在一些特殊场景下功能十分强大。

未来，eBPF 与 OpenTelemetry 标准的集成将不断加强，**让用户从不同来源收集的数据，也能够在同一环境下处理。**

当然，这两种工具提供的不仅仅是收集遥测数据。OpenTelemetry 是一组 SDK 和一系列遥测规范，而 eBPF 除了具备实现可观测性的能力，同时在安全、网络等领域也有无限的潜力。

## 应用与展望

### 过去：分散监控带来的痛点

一般的软件公司都有一个用于日志记录的平台，以及使用其他应用、平台或开源工具查看指标和链路追踪。

因此，我们不难想象到：

首先，工程师从监控告警平台收到警报，指出服务器 (包含所有微服务) 的 CPU 占用 90%，内存即将打满。

接着，工程师访问日志记录的平台，搜索该时间段内的事件，但不一定会找到相关的日志记录。

然后，可能会转到链路追踪的程序查看哪些请求花费的时间最长，前提是这些请求也占用了最多的 CPU。

于是很可能在这些应用程序之间来回切换，直到到达有问题的代码行或峰值波动的部分，然后修复它。

使用不同的三种工具查看指标、日志和链路追踪的过程，不仅繁琐缓慢，还很容易出错。
同时，在这些应用程序之间跳转，也会消耗大量的精力和时间。
由此可见，**把这三者统一到一个可视化平台，并提供快速查看的能力十分重要。**

### 现在与未来：统一的可观测性

OpenTelemetry 已经为可观测性定义了规范，并且还在不断发展。

让我们回到之前的场景：

1. 我们收到一条警报，指出服务器的 CPU 使用率为 90%。当我们使用了一个统一的可观测性系统时，就可以看到与之前不采用统一系统的区别；

2. 工程师去供应商的平台上查看指标，指标将通过带有示例的 OpenTelemetry SDK 发送，这些示例是时间序列中的突出显示值，
   包含一组键值对，这些键值对将存储相关信息，包括在此期间处于活动状态的trace ID。

3. 单击 trace ID 可以让我们立即查看该 trace，或识别可能存在问题的 trace 之后再进行更深入的调查。
   这个系统甚至可以让我们按执行时间对这些 ID 进行排序，在 trace 的视图中，我们将能够看到与该 trace 关联的日志，因此我们可以在给定时间内看到相关的内容。

4. 由于 OpenTelemtry 的规范能够让日志存储 trace ID，所以我们能够确切地知道每个日志的 trace ID，同时，我们也可以按 trace ID 查找日志。

这样一来，工程师**在特定时间查看到正确信息的几率就会呈指数级增长**。
在出现问题时关联日志、链路和指标的时间范围，只需**单击一个按钮**就可以在 OpenTelemetry 的日志、链路和指标之间**跳转**。
仅此一点就可以为工程师节省无数的时间。

另外，如果在使用 OpenTelemetry 的基础上再加上 eBPF，还会得到更多的好处：

- eBPF 可以将处理数据包从用户空间移动到内核空间，**提高网络速度和性能。**
- 在与内核及其资源交互时，它专注于尽可能**减少干扰，具有低侵入性。**
- 与构建和维护内核模块相比，eBPF 创建挂载内核函数的代码**工作量更少，也更方便。**
- eBPF 提供了一个单一、功能强大且易于访问的框架，用于**统一的追踪进程，增强了可见性和安全性等。**

总结：OpenTelemetry 加上 eBPF 的技术等于强强联合，为可观测性打造了光明未来。

以上是我对于 eBPF、OpenTelemetry 和可观测性的简短概述，当然还有更多值得深入研究的地方，让我们继续关注。

------

参考内容：

https://ebpf.io/what-is-ebpf

https://www.cncf.io/blog/2021/06/07/what-is-ebpf-and-why-does-it-matter-for-observability/