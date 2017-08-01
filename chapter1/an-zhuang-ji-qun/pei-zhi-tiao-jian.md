## 系统要求 {#system-requirements}

以下部分定义了OpenShift Container Platform环境中所有主机的硬件规格和系统级要求。

### 最低硬件要求 {#hardware}

##### **Master：**

* 物理或虚拟系统，或在公共或私有IaaS上运行的实例。
* 基本操作系统：具有“最小”安装选项的RHEL 7.3和来自Extras通道的最新软件包，或RHEL Atomic Host 7.3.2或更高版本。使用Docker 1.12及其依赖项也支持RHEL 7.2。
* 2 vCPU。
* 最小16 GB RAM。包含/ var /的文件系统的最小40GB硬盘空间

##### **Node：**

* 物理或虚拟系统，或在公共或私有IaaS上运行的实例。
* 基本操作系统：RHEL 7.3或更高版本具有“最小”安装选项，或RHEL Atomic Host 7.3.2或更高版本。使用Docker 1.12及其依赖项也支持RHEL 7.2。NetworkManager 1.0或更高版本。
* 1个vCPU。
* 最低8 GB RAM。包含/ var /的文件系统的最小15 GB硬盘空间。另外还有最少15 GB的未分配空间用于Docker的后端存储

**Etcd Node：**

最少20 GB的硬盘空间用于etcd数据。

目前，OpenShift Container Platform将图像，build和部署元数据存储在etcd中。请将etcd放在具有大量内存和快速SSD驱动器的机器上。

###  {#production-level-hardware-requirements}

### 配置核心使用 {#configuring-core-usage}

默认情况下，OpenShift Container Platform主机和节点使用它们运行的​​系统中的所有可用内核。您可以通过设置[`GOMAXPROCS`环境变量](https://golang.org/pkg/runtime/)来选择您希望OpenShift Container Platform使用的核心数量。

例如，在启动服务器之前运行以下操作，使OpenShift Container Platform仅在一个核心上运行：

```
＃export GOMAXPROCS = 1
```

### SELinux的 {#prereq-selinux}

必须在安装OpenShift Container Platform之前，在所有服务器上启用安全增强型Linux（SELinux），否则安装程序将失败。另外，`SELINUXTYPE=targeted`在_**/ etc / selinux / config**_文件中_**配置**_：

```
＃此文件控制系统上SELinux的状态。

＃SELINUX =可以使用以下三个值之一：

＃     enforcecing  - 执行SELinux安全策略。

＃     permissive  -  SELinux打印警告而不是强制执行。

＃     disabled  - 没有加载SELinux策略。

SELINUX =强制执行措施

＃SELINUXTYPE =可以使用以下三个值之一：

＃     targeted - 目标进程受到保护

＃     minimum  - 修改目标政策，只有选定的进程受到保护。

＃     mls  - 多级安全防护。

SELINUXTYPE = targeted
```

### NTP {#prereq-NTP}

您必须启用网络时间协议（NTP）以防止群集中的主节点和节点不同步。设置`openshift_clock_enabled`于`true`在Ansible剧本对主人和节点Ansible安装在集群中启用NTP。

```
＃openshift_clock_enabled = true
```

### 安全警告 {#security-warning}

OpenShift Container Platform在主机上运行[容器](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/containers_and_images.html#containers)，在某些情况下，如构建操作和注册表服务，它使用特权容器。此外，这些容器访问您的主机的Docker守护程序并执行`docker build`和`docker push`操作。因此，您应该知道与`docker run`任意图像执行操作相关的固有安全风险，因为它们有效地进行root访问。

有关更多信息，请参阅这些文章：

* [http://opensource.com/business/14/7/docker-security-selinux](http://opensource.com/business/14/7/docker-security-selinux)

* [https://docs.docker.com/engine/security/security/](https://docs.docker.com/engine/security/security/)

为了解决这些风险，OpenShift Container Platform使用[安全上下文约束](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/authorization.html#security-context-constraints)来控制pod可以执行的操作以及它具有的访问能力。

## 环境要求 {#envirornment-requirements}

以下部分定义了包含OpenShift Container Platform配置的环境要求。这包括网络注意事项和对外部服务的访问，例如Git存储库访问，存储和云基础设施提供商。

### DNS {#prereq-dns}

OpenShift Container Platform需要在环境中使用功能全面的DNS服务器。这是理想的运行DNS软件的独立主机，可以为在平台上运行的主机和容器提供名称解析。

|  | 在每个主机上的_**/ etc / hosts**_文件中添加条目是不够的。此文件不会复制到平台上运行的容器中。 |
| :--- | :--- |


OpenShift Container Platform的关键组件在容器内运行，并使用以下过程进行名称解析：

1. 默认情况下，容器从其主机收到其DNS配置文件（_**/etc/resolv.conf**_）。

2. OpenShift Container Platform然后将一个DNS值插入到pod（节点的名称服务器值之上）。该值在_**/etc/origin/node/node-config.yaml**_文件中由[`dnsIP`](https://docs.openshift.com/container-platform/3.5/admin_solutions/master_node_config.html#node-config-options)参数定义，该参数默认设置为主机节点的地址，因为主机使用**dnsmasq**。

3. 如果_**node-config.yaml**_文件中的[`dnsIP`](https://docs.openshift.com/container-platform/3.5/admin_solutions/master_node_config.html#node-config-options)参数被省略，则该值默认为kubernetes服务IP，它是该pod的_**/etc/resolv.conf**_文件中的第一个名称服务器。

从OpenShift Container Platform 3.2开始，**dnsmasq**会自动在所有主节点和节点上进行配置。pod使用节点作为其DNS，节点转发请求。默认情况下，**dnsmasq**在节点上进行配置以侦听端口53，因此节点不能运行任何其他类型的DNS应用程序。

|  | 节点上需要**NetworkManager**才能使用DNS IP地址填充**dnsmasq**。 |
| :--- | :--- |


以下是[单主站和多节点](https://docs.openshift.com/container-platform/3.5/install_config/install/planning.html#single-master-multi-node)场景的DNS记录示例集：

```
主人A 10.64.33.100

node1 A 10.64.33.101

node2 A 10.64.33.102
```

如果您没有正常运行的DNS环境，您可能会遇到失败：

* 产品安装通过参考可参考的脚本

* 基础设施容器的部署（注册表，路由器）

* 访问OpenShift Container Platform Web控制台，因为它不能通过IP地址单独访问

#### 配置主机使用DNS {#dns-config-prereq}

确保您的环境中的每个主机都配置为从DNS服务器解析主机名。主机DNS解析的配置取决于是否启用DHCP。如果DHCP是：

* 禁用，然后将您的网络接口配置为静态，并将DNS名称服务器添加到NetworkManager。

* 启用后，NetworkManager调度脚本将根据DHCP配置自动配置DNS。或者，您可以[`dnsIP`](https://docs.openshift.com/container-platform/3.5/admin_solutions/master_node_config.html#node-config-options)在_**node-config.yaml**_文件中添加一个值，以便预先安装pod的_**resolv.conf**_文件。然后，第二个名称服务器由主机的第一个名称服务器定义。默认情况下，这将是节点主机的IP地址。

  |  | 对于大多数配置，`openshift_dns_ip`在OpenShift容器平台的高级安装（使用Ansible）期间，不要设置该选项，因为此选项将覆盖设置的默认IP地址[`dnsIP`](https://docs.openshift.com/container-platform/3.5/admin_solutions/master_node_config.html#node-config-options)。相反，允许安装程序配置每个节点使用**dnsmasq**并将请求转发给SkyDNS或外部DNS提供程序。如果您设置了该`openshift_dns_ip`选项，则应使用首先查询SkyDNS的DNS IP或SkyDNS服务或端点IP（Kubernetes服务IP）进行设置。 |
  | :--- | :--- |

验证您的DNS服务器可以解析主机：

1. 检查_**/etc/resolv.conf**_的内容：

   ```
   $ cat /etc/resolv.conf

   ＃由NetworkManager生成

   搜索example.com

   名称服务器10.64.33.1

   ＃nameserver由/etc/NetworkManager/dispatcher.d/99-origin-dns.sh更新
   ```

   在这个例子中，10.64.33.1是我们的DNS服务器的地址。

2. 测试_**/etc/resolv.conf**_中列出的DNS服务器能够将主机名解析为OpenShift Container Platform环境中所有主节点和IP节点的IP地址：

   ```
   $ dig 
   <
   node_hostname
   >
    @ 
   <
   IP_address
   >
    + short
   ```

   例如：

   ```
   $ dig master.example.com @ 10.64.33.1 + short

   10.64.33.100

   $ dig node1.example.com @ 10.64.33.1 + short

   10.64.33.101
   ```

#### 禁用DNSMASQ {#dns-config-prereq-disabling-dnsmasq}

如果要禁用**dnsmasq**（例如，如果您的_**/etc/resolv.conf**_由除NetworkManager之外的配置工具管理），则在“可复制”手册中设置`openshift_use_dnsmasq`为**false**。

然而，当第一个问题**SERVFAIL发生**时，某些容器没有正确地移动到下一个域名服务器。基于Red Hat Enterprise Linux（RHEL）的容器不会受到这种影响，但是**uclibc**和**musl的**某些版本。

#### 配置DNS通配符 {#wildcard-dns-prereq}

（可选）为要使用的路由器配置通配符，以便在添加新路由时不需要更新DNS配置。

DNS区域的通配符必须最终解析为OpenShift Container Platform[路由器](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/routes.html#routers)的IP地址。

例如，为具有低生存时间值（TTL）的**cloudapps**创建通配符DNS条目，并指向将部署路由器的主机的公共IP地址：

```
* .cloudapps.example.com。
300 IN A 192.168.133.2
```

在几乎所有情况下，引用虚拟机时，必须使用主机名，并且使用的主机名必须与`hostname -f`每个节点上的命令输出相匹配。

|  | 在每个节点主机上的_**/etc/resolv.conf**_文件中，确保具有通配符条目的DNS服务器未列为名称服务器，或者通配符域未列在搜索列表中。否则，由OpenShift Container Platform管理的容器可能无法正确解析主机名。 |
| :--- | :--- |


### 网络访问 {#prereq-network-access}

主节点和节点主机之间必须存在共享网络。如果您计划使用[高级安装方法](https://docs.openshift.com/container-platform/3.5/install_config/install/advanced_install.html#install-config-install-advanced-install)[为多个主机](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#high-availability-masters)配置[高可用性](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#high-availability-masters)，则还必须在安装过程中选择要配置为[虚拟IP](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#master-components)（VIP）的[IP](https://docs.openshift.com/container-platform/3.5/architecture/infrastructure_components/kubernetes_infrastructure.html#master-components)。您选择的IP必须在所有节点之间路由，如果使用FQDN进行配置，则应在所有节点上进行解析。

#### 网络管理器 {#prereq-networkmanager}

NetworkManager是一种用于提供系统自动连接到网络的检测和配置的程序。

#### 所需端口 {#required-ports}

OpenShift Container Platform安装会在每个主机上自动创建一组内部防火墙规则`iptables`。但是，如果您的网络配置使用外部防火墙（例如基于硬件的防火墙），则必须确保基础架构组件可以通过用作特定进程或服务的通信端点的特定端口相互通信。

确保OpenShift Container Platform所需的以下端口在您的网络上打开，并配置为允许主机之间访问。某些端口是可选的，具体取决于您的配置和用法。

|  |  |  |
| :--- | :--- | :--- |
| **4789** | UDP | 需要在单独主机上的pod之间进行SDN通信。 |

|  |  |  |
| :--- | :--- | :--- |
| **53**或**8053** | TCP / UDP | 群集服务（SkyDNS）的DNS解析需要。3.2之前的安装或环境升级到3.2使用端口53.默认情况下，新安装将使用8053，以便配置**dnsmasq**。 |
| **4789** | UDP | 需要在单独主机上的pod之间进行SDN通信。 |
| **443**或**8443** | TCP | 节点主机需要与主API通信，节点主机发回状态，接收任务等。 |

|  |  |  |
| :--- | :--- | :--- |
| **4789** | UDP | 需要在单独主机上的pod之间进行SDN通信。 |
| **10250** | TCP | 主机代理通过Kubelet进行节点主机的`oc`命令。 |

|  | 在下表中，**（L）**表示标记端口也用于_环回模式_，使主设备能够与自身进行通信。在单主集群中：标有**（L）的**端口必须打开。没有标记**（L）的**端口不需要打开。在多主集群中，所有列出的端口必须是打开的。 |
| :--- | :--- |


|  |  |  |
| :--- | :--- | :--- |
| **53（L）**或**8053（L）** | TCP / UDP | 群集服务（SkyDNS）的DNS解析需要。3.2之前的安装或环境升级到3.2使用端口53.默认情况下，新安装将使用8053，以便配置**dnsmasq**。 |
| **2049（L）** | TCP / UDP | 在将NFS主机配置为安装程序的一部分时需要。 |
| **2379** | TCP | 用于独立的etcd（集群）接受状态更改。 |
| **2380** | TCP | 在使用独立的etcd（集群）时，etcd需要在主机之间打开头端选择和对等连接。 |
| **4001（L）** | TCP | 用于嵌入式etcd（非群集）接受状态更改。 |
| **4789（L）** | UDP | 需要在单独主机上的pod之间进行SDN通信。 |

|  |  |  |
| :--- | :--- | :--- |
| **9000** | TCP | 如果选择`native`HA方法，可选择允许访问HAProxy统计信息页面。 |

|  |  |  |
| :--- | :--- | :--- |
| **443**或**8443** | TCP | 节点主机需要与主API进行通信，节点主机可以发回状态，接收任务等。 |

|  |  |  |
| :--- | :--- | :--- |
| **22** | TCP | 安装程序或系统管理员需要SSH。 |
| **53**或**8053** | TCP / UDP | 群集服务（SkyDNS）的DNS解析需要。3.2之前的安装或环境升级到3.2使用端口53.默认情况下，新安装将使用8053，以便配置**dnsmasq**。只需要在主机上内部打开。 |
| **80**或**443** | TCP | 对于路由器的HTTP / HTTPS使用。需要在节点主机上外部打开，特别是在运行路由器的节点上。 |
| **1936年** | TCP | 对于路由器统计使用。运行模板路由器以访问统计信息时需要打开，并且可以根据是否希望公开表示统计信息，在外部或内部打开连接。 |
| **4001** | TCP | 对于嵌入式的etcd（非群集）使用。只需要在主机上内部打开。**4001**用于服务器 - 客户端连接。 |
| **2379**和**2380** | TCP | 用于独立的etcd使用。只需要在主机上内部打开。**2379**用于服务器 - 客户端连接。**2380**用于服务器 - 服务器连接，仅当您具有集群等级时才需要。 |
| **4789** | UDP | 对于VxLAN使用（OpenShift SDN）。仅在节点主机上才需要。 |
| **8443** | TCP | 供OpenShift Container Platform Web控制台使用，与API服务器共享。 |
| **10250** | TCP | 供Kubelet使用。需要在节点上外部打开。 |

**笔记**

* 在上述示例中，端口**4789**用于用户数据报协议（UDP）。

* 当部署使用SDN时，通过服务代理访问pod网络，除非它正在从注册表部署的同一节点访问注册表。

* OpenShift Container Platform内部DNS不能通过SDN接收。根据检测到的值`openshift_facts`，或者如果`openshift_ip`和`openshift_public_ip`值被覆盖，它将是计算的值`openshift_ip`。对于非云部署，这将默认与主主机上的默认路由相关联的IP地址。对于云部署，它将默认为由云元数据定义的与第一个内部接口相关联的IP地址。

* 主主机使用端口**10250**到达节点，不会超过SDN。它取决于目标主机的部署，并使用计算的值`openshift_hostname`和`openshift_public_hostname`。

|  |  |  |
| :--- | :--- | :--- |
| **9200** | TCP | 对于Elasticsearch API使用。需要在任何基础设施节点内部打开，以便Kibana能够检索日志以进行显示。它可以外部打开，通过路线直接访问Elasticsearch。路线可以使用`oc expose`。 |
| **9300** | TCP | 对于弹性搜索集群间的使用。需要在任何基础架构节点上内部打开，以便弹性搜索集群的成员可以彼此通信。 |

### 持久存储 {#prereq-persistent-storage}

Kubernetes[持久性卷](https://docs.openshift.com/container-platform/3.5/architecture/additional_concepts/storage.html#architecture-additional-concepts-storage)框架允许您使用您的环境中可用的网络存储来为OpenShift容器平台集群提供持久存储。这可以在完成根据您的应用需求完成初始OpenShift容器平台安装之后完成，为用户提供一种请求这些资源的方法，而不需要了解底层基础架构。

“[安装和配置指南”](https://docs.openshift.com/container-platform/3.5/install_config/index.html#install-config-index)提供了有关集群管理员使用[NFS](https://docs.openshift.com/container-platform/3.5/install_config/persistent_storage/persistent_storage_nfs.html#install-config-persistent-storage-persistent-storage-nfs)，[GlusterFS](https://docs.openshift.com/container-platform/3.5/install_config/persistent_storage/persistent_storage_glusterfs.html#install-config-persistent-storage-persistent-storage-glusterfs)，[Ceph RBD](https://docs.openshift.com/container-platform/3.5/install_config/persistent_storage/persistent_storage_ceph_rbd.html#install-config-persistent-storage-persistent-storage-ceph-rbd)，[OpenStack Cinder](https://docs.openshift.com/container-platform/3.5/install_config/persistent_storage/persistent_storage_cinder.html#install-config-persistent-storage-persistent-storage-cinder)，[AWS弹性块存储（EBS）](https://docs.openshift.com/container-platform/3.5/install_config/persistent_storage/persistent_storage_aws.html#install-config-persistent-storage-persistent-storage-aws)，[GCE持久性磁盘](https://docs.openshift.com/container-platform/3.5/install_config/persistent_storage/persistent_storage_gce.html#install-config-persistent-storage-persistent-storage-gce)和[iSCSI](https://docs.openshift.com/container-platform/3.5/install_config/persistent_storage/persistent_storage_iscsi.html#install-config-persistent-storage-persistent-storage-iscsi)配置持久存储的OpenShift容器平台集群的说明。

### 云提供商注意事项 {#prereq-cloud-provider-considerations}

在云提供商上安装OpenShift Container Platform有一些方面需要考虑。

#### 配置安全组 {#configuring-a-security-group}

在AWS或OpenStack上安装时，请确保您设置了相应的安全组。这些是您应该在安全组中的一些端口，如果没有安装，安装将失败。您可能需要更多的依赖于您要安装的群集配置。有关更多信息并相应调整安全组，请参阅[所需端口](https://docs.openshift.com/container-platform/3.5/install_config/install/prerequisites.html#required-ports)以获取更多信息。

| **所有OpenShift容器平台主机** | 来自运行安装程序/可安装的主机的tcp / 22 |
| :--- | :--- |
| **etcd安全组** | tcp / 2379从主人tcp / 2380从etcd主机 |
| **主安全组** | tcp / 8443从0.0.0.0/0来自所有OpenShift Container Platform主机的tcp / 53，用于在3.2之前安装或升级到3.2的环境来自所有OpenShift Container Platform主机的udp / 53，用于在3.2之前安装或升级到3.2的环境来自所有OpenShift Container Platform主机的tcp / 8053，用于安装3.2的新环境来自所有OpenShift Container Platform主机的udp / 8053，用于安装3.2的新环境 |
| **节点安全组** | tcp / 10250从主人udp / 4789从节点 |
| **基础架构节点**（可承载OpenShift Container Platform路由器的**节点**） | tcp / 443从0.0.0.0/0tcp / 80从0.0.0.0/0 |

如果配置ELB用于负载平衡主设备和/或路由器，则还需要适当地为ELB配置Ingress和Egress安全组。

#### 覆盖检测到的IP地址和主机名 {#overriding-detected-ip-addresses-and-host-names}

某些部署要求用户覆盖主机的检测到的主机名和IP地址。要查看默认值，请运行`openshift_facts`playbook：

```
＃ansible-playbook playbooks / byo / openshift_facts.yml
```

现在，验证检测到的常见设置。如果他们不是你期望的，你可以覆盖它们。

该[高级安装](https://docs.openshift.com/container-platform/3.5/install_config/install/advanced_install.html#configuring-ansible)主题讨论中更详细地提供Ansible变量。

| 变量 | 用法 |
| :--- | :--- |
| `hostname` | 应该从实例本身解决到内部IP。`openshift_hostname`覆盖。 |
| `ip` | 应该是实例的内部IP。`openshift_ip`将覆盖。 |
| `public_hostname` | 应该从云外的主机解决外部IP。供应商`openshift_public_hostname`覆盖。 |
| `public_ip` | 应该是与实例相关联的外部可访问的IP。`openshift_public_ip`覆盖。 |
| `use_openshift_sdn` | 应该是真实的，除非云是GCE。`openshift_use_openshift_sdn`覆盖。 |

|  | 如果`openshift_hostname`设置为除元数据提供`private-dns-name`值之外的值，那么这些提供程序的本机云集成将不再起作用。 |
| :--- | :--- |


在AWS中，需要覆盖变量的情况包括：

| 变量 | 用法 |
| :--- | :--- |
| `hostname` | 用户在没有配置这两个VPC的是安装`DNS hostnames`和`DNS resolution`。 |
| `ip` | 如果他们有多个网络接口配置，并且希望使用一个默认网络接口，可能是可能的。你必须先设置`openshift_set_node_ip`到`True`。否则，SDN将尝试使用该`hostname`设置或尝试解析IP的主机名。 |
| `public_hostname` | 未配置VPC子网的主实例`Auto-assign Public IP`。对于外部访问此主服务器，您需要配置ELB或其他负载平衡器，以提供所需的外部访问，或者需要通过VPN连接连接到主机的内部名称。禁用元数据的主实例。这个值实际上不是由节点使用的。 |
| `public_ip` | 未配置VPC子网的主实例`Auto-assign Public IP`。禁用元数据的主实例。这个值实际上不是由节点使用的。 |

如果设置`openshift_hostname`为除元数据提供的`private-dns-name`值之外的其他东西，则这些提供商的本地云集成将不再起作用。

特别是对于EC2的主机，它们必须在VPC同时具有部署`DNS host names`和`DNS resolution`启用，并且`openshift_hostname`不应该被重写。

#### 云提供商的安装后配置 {#post-installation-configuration-for-cloud-providers}

安装过程之后，您可以为[AWS](https://docs.openshift.com/container-platform/3.5/install_config/configuring_aws.html#install-config-configuring-aws)，[OpenStack](https://docs.openshift.com/container-platform/3.5/install_config/configuring_openstack.html#install-config-configuring-openstack)或[GCE](https://docs.openshift.com/container-platform/3.5/install_config/configuring_gce.html#install-config-configuring-gce)配置OpenShift容器平台。

