## 概述 {#overview}

使用Ansible playbook是安装OpenShift容器平台集群的高级安_装_方法。在你熟悉Ansible的前提下，可以使用此配置作为参考，实现自定义化的配置。

## 准备开始 {#advanced-before-you-begin}

在安装OpenShift Container Platform之前，您必须首先确保已经完成配置《主机准备》的[先决条件](https://docs.openshift.com/container-platform/3.5/install_config/install/prerequisites.html#install-config-install-prerequisites)和[主机准备](https://docs.openshift.com/container-platform/3.5/install_config/install/host_preparation.html#install-config-install-host-preparation)主题。这包括对每个组件类型的系统和环境要求进行验证，并正确安装和配置Docker。它还包括安装可执行版本2.2.0或更高版本，因为高级安装方法基于可复制的剧本，因此需要直接调用可复制。

如果您有兴趣使用容器化方法安装OpenShift容器平台（RHEL可选，但RHEL Atomic Host必需），请参阅[在容器化主机](https://docs.openshift.com/container-platform/3.5/install_config/install/rpm_vs_containerized.html#install-config-install-rpm-vs-containerized)上[安装，](https://docs.openshift.com/container-platform/3.5/install_config/install/rpm_vs_containerized.html#install-config-install-rpm-vs-containerized)以确保您了解这些方法之间的差异，然后返回到此主题继续。

对于大规模安装，包括优化安装时间的建议，请参阅“[扩展和性能指南”](https://docs.openshift.com/container-platform/3.5/scaling_performance/install_practices.html#scaling-performance-install-best-practices)。

遵循[先决条件](https://docs.openshift.com/container-platform/3.5/install_config/install/prerequisites.html#install-config-install-prerequisites)主题中的说明并确定RPM和容器化方法之间的关系，您可以在本主题中继续[配置](https://docs.openshift.com/container-platform/3.5/install_config/install/advanced_install.html#configuring-ansible)可[配置的库存文件](https://docs.openshift.com/container-platform/3.5/install_config/install/advanced_install.html#configuring-ansible)。

## 配置可选的库存文件 {#configuring-ansible}

将_**在/ etc / ansible /主机**_文件是用来安装OpenShift容器装载的剧本Ansible的清单文件。库存文件描述了OpenShift Container Platform集群的配置。您必须用所需的配置替换文件的默认内容。

以下部分描述了在高级安装期间在库存文件中设置的常用变量，后跟可用作安装起点的[清单文件示例](https://docs.openshift.com/container-platform/3.5/install_config/install/advanced_install.html#adv-install-example-inventory-files)。

描述的许多可选变量是可选的。接受默认值应足以满足开发环境，但对于生产环境，建议您阅读并熟悉各种可用选项。

示例清单描述了各种环境拓扑，包括[使用多个主器件实现高可用性](https://docs.openshift.com/container-platform/3.5/install_config/install/advanced_install.html#multiple-masters)。您可以选择符合您的要求的示例，修改它以匹配您自己的环境，并在[运行高级安装](https://docs.openshift.com/container-platform/3.5/install_config/install/advanced_install.html#running-the-advanced-installation)时将其用作库存文件。

### 图像版本策略 {#advanced-install-image-version-policy}

图片需要版本号政策才能维护更新。有关详细信息，请参阅“架构指南”中的“[映像版本标记策略”](https://docs.openshift.com/container-platform/3.5/architecture/core_concepts/containers_and_images.html#architecture-images-tag-policy)部分

### 配置集群变量 {#configuring-cluster-variables}

要在安全性较高的应用程序中将环境变量分配给整个OpenShift容器平台集群，请在**\[OSEv3：vars\]**部分的单独一行的_**/ etc / ansible / hosts**_文件中指定所需的变量。例如：

