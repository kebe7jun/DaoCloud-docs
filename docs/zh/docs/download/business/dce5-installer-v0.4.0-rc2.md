---
date: 2023-01-05
---

# DCE 5.0 商业版 v0.4.0-rc2

本页可下载 DCE 5.0 商业版的离线安装包和校验文件。

## 下载

| 文件名                      | 版本    | 架构 | 文件大小 | 下载                                           | 更新日期   |
| ----------------------------- | ------- | -------- | ---------------------------------------------- | ---------- | ----------------------------- |
| offline-centos7-v0.4.0-rc2-amd64.tar | v0.4.0-rc2 | AMD 64 | 16.33GB | [:arrow_down: 下载](https://qiniu-download-public.daocloud.io/DaoCloud_Enterprise/dce5/offline-centos7-v0.4.0-rc2-amd64.tar) | 2023-01-05 |
| offline-kylin-v10sp2-v0.4.0-rc2-arm64.tar | v0.4.0-rc2 | ARM 64 | 14.96GB | [:arrow_down: 下载](https://qiniu-download-public.daocloud.io/DaoCloud_Enterprise/dce5/offline-kylin-v10sp2-v0.4.0-rc2-arm64.tar) | 2023-01-05 |

## 校验

进入离线安装包下载目录。

=== "AMD 64"

    执行以下命令校验安装包：

    ```sh
    echo "9fdc24ea0eda2994cb1ab253d3c6f079b76b138618fb5601176be5a4d660893c3cc64dfb151e32d57a1f843c6b598bed270a20654e6cd8d815c7901124b04431  offline-centos7-v0.4.0-rc2-amd64.tar" | sha512sum -c
    ```

    校验成功会打印：

    ```none
    offline-centos7-v0.4.0-rc2-amd64.tar: OK
    ```

=== "ARM 64"

    执行以下命令校验安装包：

    ```sh
    echo "ec6eca321a80f26f70c11739eb15dc67914579af610e2fb9cb49fcf8d9691de2ceddac7cfd8bd40ce3f3927cfea3166d15366f06909fcbbf15a330bc718fd358  offline-kylin-v10sp2-v0.4.0-rc2-arm64.tar" | sha512sum -c
    ```

    校验成功会打印：

    ```none
    offline-kylin-v10sp2-v0.4.0-rc2-arm64.tar: OK
    ```

## 安装

成功校验离线包之后，

=== "AMD 64"

    执行以下命令解压缩 tar 包：

    ```sh
    tar -zxvf offline-centos7-v0.4.0-rc2-amd64.tar
    ```

=== "ARM 64"

    执行以下命令解压缩 tar 包：

    ```sh
    tar -zxvf offline-kylin-v10sp2-v0.4.0-rc2-arm64.tar
    ```

- 安装请参阅[商业版安装流程](../../install/commercial/start-install.md)
- 成功安装之后请进行[正版授权](https://qingflow.com/f/e3291647)

## 模块

DCE 5.0 商业版包含以下模块，按需即插即用满足各类应用场景：

| 模块                 | 介绍                                                                     | 最新动态                                                      |
| -------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------- |
| 全局管理             | 负责用户访问控制、权限、企业空间、审计日志、个性化外观设置等             | [v0.11](../../ghippo/01ProductBrief/release-notes.md#v011)    |
| 容器管理             | 管理集群、节点、工作负载、Helm 应用、CRD、命名空间等 K8s 核心功能        | [v0.12](../../kpanda/03ProductBrief/release-notes.md#v012)    |
| 可观测性             | 提供丰富的仪表盘、场景监控、数据查询、告警等图文信息                     | [v0.11](../../insight/03ProductBrief/releasenote.md#v011)     |
| 应用工作台           | 基于容器的 DevOps 应用平台，支持 Jenkins, Tekton, GitOps 等流水线作业    | [v0.9](../../amamba/01ProductBrief/release-notes.md#v09)      |
| 多云编排             | 集中管理多云、混合云、跨云资源的应用编排，具备多云灾备、故障恢复等能力   | [v0.3](../../kairship/01product/release-notes.md#v03)         |
| 微服务引擎           | 提供注册发现、服务治理、配置管理、微服务网关等治理能力                   | [v0.11](../../skoala/intro/release-notes.md#v011)             |
| 服务网格             | 基于 Istio 开源技术构建的面向云原生应用的下一代服务网格                  | [v0.10](../../mspider/01Intro/release-notes.md#v010)          |
| 中间件 Elasticsearch | 目前首选的全文搜索引擎                                                   | [v0.3](../../middleware/elastic-search/release-notes.md#v034) |
| 中间件 Kafka         | 基于开源软件 Kafka 提供的分布式消息队列服务                              | [v0.1](../../middleware/kafka/release-notes.md#v012)          |
| 中间件 MinIO         | 一款非常热门的轻量、开源对象存储方案                                     | [v0.1](../../middleware/minio/release-notes.md#v012)          |
| 中间件 MySQL         | 应用最广泛的开源关系数据库                                               | [v0.4](../../middleware/mysql/release-notes.md#v04)           |
| 中间件 RabbitMQ      | 实现了高级消息队列协议 (AMQP) 的开源消息代理软件                         | [v0.6](../../middleware/rabbitmq/release-notes.md#v06)        |
| 中间件 Redis         | 一款内存数据库缓存服务                                                   | [v0.2](../../middleware/redis/release-notes.md#v02)           |
| 镜像仓库             | 用于存储 K8s、DevOps 和容器应用开发的镜像                                | [发布说明](../../release/rn5.0.md)                            |
| 网络                 | 针对不同的 Linux 内核，支持多种 CNI 组合方案                             | [发布说明](../../release/rn5.0.md)                            |
| 存储                 | 提供统一数据存储服务，支持文件、对象、块、本地存储，轻松接入存储厂商方案 | [发布说明](../../release/rn5.0.md)                            |

## 更多

- [在线文档](https://docs.daocloud.io/dce/what-is-dce/)
- [报告 bug](https://github.com/DaoCloud/DaoCloud-docs/issues)