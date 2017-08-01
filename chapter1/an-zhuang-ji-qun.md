## 系统要求 {#system-requirements}

以下部分标识了OpenShift Container Platform环境中所有主机的硬件规格和系统级要求。

### 红帽订阅 {#red-hat-subscription}

您的Red Hat帐户必须有一个活跃的OpenShift Container Platform订阅才能继续。如果没有，请联系您的销售代表以获取更多信息。

|  | OpenShift容器平台3.5需要Docker 1.12。 |
| :--- | :--- |


### 最低硬件要求 {#hardware}

系统要求因主机类型而异：

| [大师](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#master) | 物理或虚拟系统，或在公共或私有IaaS上运行的实例。基本操作系统：具有“最小”安装选项的RHEL 7.3和来自Extras通道的最新软件包，或RHEL Atomic Host 7.3.2或更高版本。使用Docker 1.12及其依赖项也支持RHEL 7.2。2 vCPU。最小16 GB RAM。包含_**/ var /**_的文件系统的最小40 GB硬盘空间。 |
| :--- | :--- |
| [节点](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#node) | 物理或虚拟系统，或在公共或私有IaaS上运行的实例。基本操作系统：RHEL 7.3或更高版本具有“最小”安装选项，或RHEL Atomic Host 7.3.2或更高版本。使用Docker 1.12及其依赖项也支持RHEL 7.2。NetworkManager 1.0或更高版本。1个vCPU。最低8 GB RAM。包含_**/ var /**_的文件系统的最小15 GB硬盘空间。另外还有最少15 GB的未分配空间用于Docker的后端存储;请参阅[配置Docker存储](https://docs.openshift.com/container-platform/3.5/install_config/install/host_preparation.html#configuring-docker-storage)。 |
| 外部etcd节点 | 最少20 GB的硬盘空间用于etcd数据。请参阅[硬件建议，](https://github.com/coreos/etcd/blob/master/Documentation/op-guide/hardware.md#hardware-recommendations)以正确调整您的etcd节点。目前，OpenShift Container Platform将图像，构建和部署元数据存储在etcd中。您必须定期[修剪旧资源](https://docs.openshift.com/container-platform/3.5/admin_guide/pruning_resources.html#admin-guide-pruning-resources)。如果您计划利用大量的映像/构建/部署，请将etcd放在具有大量内存和快速SSD驱动器的机器上。 |

|  | OpenShift Container Platform仅支持具有x86\_64架构的服务器。 |
| :--- | :--- |


|  | 满足RHEL Atomic Host中的_**/ var /**_file系统大小要求需要更改默认配置。有关在安装期间或安装后进行配置的说明，请参阅[管理Red Hat Enterprise Linux Atomic主机中的存储](https://access.redhat.com/documentation/en/red-hat-enterprise-linux-atomic-host/version-7/getting-started-with-containers/#managing_storage_in_red_hat_enterprise_linux_atomic_host)。 |
| :--- | :--- |




