## 初步规划 {#inital-planning}

对于生产环境，影响安装的因素有几个。在阅读文档时，请考虑以下问题：

* _要使用哪种安装方式？_“[安装方法”](https://docs.openshift.com/container-platform/3.5/install_config/install/planning.html#installation-methods)部分提供了有关快速和高级安装方法的一些信息。

* _在集群中需要多少个主机？_“[环境方案”](https://docs.openshift.com/container-platform/3.5/install_config/install/planning.html#environment-scenarios)部分提供了单主站和多主站配置的多个示例。

* _您的群集中需要多少个pod？_“[大小注意事项”](https://docs.openshift.com/container-platform/3.5/install_config/install/planning.html#sizing)部分为节点和pod提供了限制，因此您可以计算出环境所需的大小。

* _是否需要_[_高可用性_](https://docs.openshift.com/container-platform/3.5/admin_guide/high_availability.html#admin-guide-high-availability)_？_推荐使用高可用性进行容错。在这种情况下，您可能希望使用[多Mater](https://docs.openshift.com/container-platform/3.5/install_config/install/planning.html#multi-masters-using-native-ha)[使用Native HA](https://docs.openshift.com/container-platform/3.5/install_config/install/planning.html#multi-masters-using-native-ha)示例作为您的环境的基础。

* _您要使用哪种安装类型：_[_RPM或集装箱式_](https://docs.openshift.com/container-platform/3.5/install_config/install/planning.html#rpm-vs-containerized)_？_这两个安装都提供了一个工作的OpenShift容器平台环境，但您可能希望安装，管理和更新服务的特定方法。

## 安装方法 {#installation-methods}

快速和高级的安装方法都支持开发和生产环境。如果您想要快速获取OpenShift容器平台，并首次尝试运行，请使用快速安装程序，让交互式CLI指导您完成与您的环境相关的配置选项。

为了最大限度地控制群集的配置，您可以使用高级安装方法。如果您已经熟悉了Ansible，此方法特别适用。

如果最初使用快速安装程序进行安装，则可以随时进一步调整集群的配置，并使用相同的安装程序工具调整集群中的主机数量。如果以后要切换到使用高级方法，可以为您的配置创建一个库存文件，并继续执行此操作。

## 规模大小 {#sizing}

确定OpenShift容器平台集群所需的节点和容器数量。集群可扩展性与集群环境中的容器数量相关。该数量将影响您的设置中的其他数量。

下表提供了节点和容器的最数量限制：

| 类型 | 最大值 |
| :--- | :--- |
| 每个群集的最大节点数 | 1000 |
| 每个群集的最大容器数 | 120000 |
| 每个节点的最大容器数量 | 250 |
| 每个核\(core\)的最大容器数量 | 10 |

确定每个节点有多少个pod？

```
节点总数 = 每个群集中的最大容器数量 / 每个节点的预期容器数量
```

示例场景

如果需求为1个群集有2200个容器，设每个节点有250个容器，所以至少需要9个节点：

```
2200/250 = 8.8
```

如果将节点的容器数量为20个，则将有110个节点：

```
2200/20 = 110
```

## 环境情景 {#environment-scenarios}

本节概述了OpenShift容器平台环境的不同场景示例。使用这些方案作为规划自己的OpenShift Container Platform集群。

```
！注意：不支持安装后从单个master的集群升级为多个master的集群
```

### 单主站和多节点 {#single-master-multi-node}

下表描述了单个[主控](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#master)（具有嵌入式**等级**）和两个[节点](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#node)的示例环境：

| 主机名 | 要安装的基础设施组件 |
| :--- | :--- |
| **master.example.com** | 主节点 |
| **node1.example.com** | 节点 |
|  | **node2.example.com** |

### 单主，多等，多节点 {#single-master-multi-etcd-multi-node}

下表描述了用于单个的示例环境[主](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#master)，三台[**ETCD**](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#master)主机，以及两个[节点](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#node)：

| 主机名 | 要安装的基础设施组件 |
| :--- | :--- |
| **master.example.com** | 主节点 |
| **etcd1.example.com** | **ETCD** |
|  | **etcd2.example.com** |
|  | **etcd3.example.com** |
| **node1.example.com** | 节点 |
|  | **node2.example.com** |

|  | 指定多个**etcd**主机时，安装并配置external**etcd**。不支持OpenShift Container Platform的嵌入式**集群**。 |
| :--- | :--- |


### 使用本机HA的多个大师 {#multi-masters-using-native-ha}

下面描述三个的示例环境[的主人](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#master)，一个HAProxy的负载平衡器，三台[**ETCD**](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#master)主机，以及两个[节点](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#node)使用`native`HA方法：

| 主机名 | 要安装的基础设施组件 |
| :--- | :--- |
| **master1.example.com** | 主（使用本机HA集群）和节点 |
|  | **master2.example.com** |
|  | **master3.example.com** |
| **lb.example.com** | HAProxy负载平衡API主端点 |
| **etcd1.example.com** | **ETCD** |
|  | **etcd2.example.com** |
|  | **etcd3.example.com** |
| **node1.example.com** | 节点 |
|  | **node2.example.com** |

|  | 指定多个**etcd**主机时，安装并配置external**etcd**。不支持OpenShift Container Platform的嵌入式**集群**。 |
| :--- | :--- |


### 独立注册表 {#planning-stand-alone-registry}

您还可以使用OpenShift Container Platform的集成注册表来安装OpenShift Container Platform作为独立注册表。有关此方案的详细信息，请参阅[安装独立注册表](https://docs.openshift.com/container-platform/3.5/install_config/install/stand_alone_registry.html#install-config-installing-stand-alone-registry)。

## RPM与Containerized {#rpm-vs-containerized}

RPM安装通过包管理安装所有服务，并将服务配置为在同一用户空间内运行，而容器化安装使用容器映像安装服务，并在单个容器中运行单独的服务。

有关配置安装以使用容器化服务的更多详细信息，请参阅[在集群化主机上安装](https://docs.openshift.com/container-platform/3.5/install_config/install/rpm_vs_containerized.html#install-config-install-rpm-vs-containerized)主题

