---
lab:
    title: '3a: 在 Azure VM 上实施 Linux 群集'
    module: '模块 3：Azure 上的 SAP 认证产品'
---

# AZ 120 模块 3：Azure 上的 SAP 认证产品
# 实验室 3a: 在 Azure VM 上实施 Linux 群集

预计用时：90 分钟

本实验室中的所有任务都是从 Azure 门户执行的（包括 Bash Cloud Shell 会话）  

   > **注意**：不使用 Cloud Shell 时，实验室虚拟机必须安装 Azure CLI[**https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli-windows?view=azure-cli-latest**](https://docs.microsoft.com/zh-cn/cli/azure/install-azure-cli-windows?view=azure-cli-latest)并包括一个 SSH 客户端，例如 PuTTY，可从以下地址获得[**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)。

实验室文件：无

## 方案
  
为准备在 Azure 上部署 SAP HANA，Adatum Corporation 希望探索在运行 Linux 的 SUSE 发行版的 Azure VM 上实施群集的过程。

## 目标
  
完成本实验室后，你将能够：

-   预配支持高可用性 SAP HANA 部署所需的 Azure 计算资源

-   配置运行 Linux 的 Azure VM 操作系统，以支持高可用性 SAP HANA 安装

-   预配支持高可用性 SAP HANA 部署所需的 Azure 网络资源

## 要求

-   具有足够数量的可用 DSv3 vCPU (2 x 4) 和 DSv2 (1 x 1) vCPU 的 Microsoft Azure 订阅

-   运行 Windows 10、Windows Server 2016 或 Windows Server 2019 的实验室计算机可以访问 Azure


## 练习 1：预配支持高可用性 SAP HANA 部署所需的 Azure 计算资源

持续时间：30 分钟

在本练习中，你将部署配置 Linux 群集所必需的 Azure 基础结构计算组件。这将涉及在同一可用性集中创建一对运行 Linux SUSE 的 Azure VM。

### 任务 1：部署运行 Linux SUSE 的 Azure VM

1.  从实验室计算机中启动网页浏览器，并导航到 Azure 门户网站：https://portal.azure.com

1.  如果出现提示，则使用你在本实验室中使用的 Azure 订阅的所有者或参与者角色登录工作或学校或个人 Microsoft 帐户。

1.  在 Azure 门户上，单击 **+新建资源**。

1.  在 **新建** 边栏选项卡中，基于 **SLES 12 SP3 (BYOS)** 图像，开始使用以下设置创建新的 Azure VM：

    -   订阅： *你的 Azure 订阅名*

    -   资源组： *新资源组，名为* **az12001a-RG**

    -   虚拟机名称： **az12001a-vm0**

    -   区域： *Azure 区域，你可以在该区域中部署 Azure VM，并且最靠近实验室*

    -   可用性选项： **可用性集**

    -   可用性集： *新可用性集，名为* ** az12001a-avset**，*具有 2 个故障域和 5 个更新域*。

    -   图像： **SUSE Linux Enterprise Server (SLES) 12 SP3 (BYOS)**

    -   大小： **标准 D4s v3**

    -   验证类型： **密码**

    -   用户名： **student**

    -   密码： **Pa55w.rd1234**

    -   公共入站端口： **允许选定的端口**

    -   选定的入站端口： **SSH (22)**

    -   操作系统磁盘类型： **高级 SSD**

    -   虚拟网络： *一个新虚拟网络名为* **az12001a-RG-vnet**

    -   地址空间： **192.168.0.0/20**

    -   子网名称： **subnet-0**

    -   子网地址范围： **192.168.0.0/24**

    -   公共 IP 地址： *新 IP 地址名为* **az12001a-vm0-ip**

    -   NIC 网络安全组： **高级**

    -   配置网络安全组： *新网络安全组，名为* **az12001a-vm0-nsg** *，使用默认设置（允许入站 SSH 连接）*

    -   加速网络： **关**

    -   将此虚拟机置于现有负载均衡解决方案之后： **否**

    -   免费启用基本计划： **否**

    -   启动诊断： **关**

    -   OS 来宾诊断： **关**

    -   系统分配的托管标识： **关**

    -   使用 AAD 凭据登录（预览）： **关**

    -   启用自动关闭： **关**

    -   扩展： *无*

    -   标签： **无**

1.  等待预配完成。这可能需要几分钟。

1.  在 Azure 门户上，单击 **+新建资源**。

1.  在 **新建** 边栏选项卡中，基于 **SLES 12 SP3 (BYOS)** 图像，开始使用以下设置创建新的 Azure VM：

    -   订阅： *你的 Azure 订阅名*

    -   资源组： **az12001a-RG**

    -   虚拟机名称： **az12001a-vm1**

    -   区域： *你部署第一个 Azure VM 的 Azure 区域*

    -   可用性选项： **可用性集**

    -   可用性集： **az12001a-avset**

    -   图像： **SUSE Linux Enterprise Server (SLES) 12 SP3 (BYOS)**

    -   大小： **标准 D4s v3**

    -   验证类型： **密码**

    -   用户名： **student**

    -   密码： **Pa55w.rd1234**

    -   公共入站端口： **允许选定的端口**

    -   选定的入站端口： **SSH (22)**

    -   操作系统磁盘类型： **高级 SSD**

    -   虚拟网络： **az12001a-RG-vnet**

    -   子网名称： **subnet-0 (192.168.0.0/24)**

    -   公共 IP 地址： *新 IP 地址名为* **az12001a-vm1-ip**

    -   NIC 网络安全组： **高级**

    -   配置网络安全组： *新网络安全组，名为* **az12001a-vm1-nsg** *，使用默认设置（允许入站 SSH 连接）*

    -   加速网络： **关**

    -   将此虚拟机置于现有负载均衡解决方案之后： **否**

    -   免费启用基本计划： **否**

    -   启动诊断： **关**

    -   OS 来宾诊断： **关**

    -   系统分配的托管标识： **关**

    -   使用 AAD 凭据登录（预览）： **关**

    -   启用自动关闭： **关**

    -   扩展： *无*

    -   标签： **无**

1.  等待预配完成。这可能需要几分钟。


### 任务 2：创建并配置 Azure VM 磁盘

1.  从 Azure 门户，在 Cloud Shell 启动 Bash 会话。 

    > **注意**：如果这是你第一次在当前 Azure 订阅中启动 Cloud Shell，则会要求你创建 Azure 文件共享以保留 Cloud Shell 文件。如果是，请接受默认值，将在自动生成的资源组中创建存储帐户。

1.  在 Cloud Shell 窗格中，运行以下命令，以创建第一组共计 8 个托管磁盘，这些磁盘将附加到你在上一个任务中部署的第一个 Azure VM：

```
RESOURCE_GROUP_NAME='az12001a-RG'

LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
```

1.  在 Cloud Shell 窗格中，运行以下命令，以创建第二组共计 8 个托管磁盘，这些磁盘将附加到你在上一个任务中部署的第二个 Azure VM：

```
for I in {0..7}; do az disk create --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm1-DataDisk$I --size-gb 128 --location $LOCATION --sku Premium_LRS; done
```

1.  在 Azure 门户中，导航到你在前一个任务 (**az12001a-vm0**) 中预配的第一个 Azure VM 边栏选项卡。

1.  从 **az12001a-vm0** 边栏选项卡，导航到 **az12001a-vm0 - 磁盘** 边栏选项卡。

1.  在 **az12001a-vm0 - Disks** 边栏选项卡，使用以下设置将数据磁盘附加到 az12001a-vm0：

    -   LUN： **0**

    -   磁盘名称： **az12001a-vm0-DataDisk0**

    -   资源组： **az12001a-RG**

    -   主机缓存： **无**

1.  重复上一步，使用前缀 **az12001a-vm0-DataDisk** 附加剩余的 7 个磁盘（总共 8 个）。分配与磁盘名的最后一个字符匹配的 LUN 编号。

1.  保存你的更改。 

1.  在 Azure 门户中，导航到你在前一个任务 (**az12001a-vm1**) 中预配的第二个 Azure VM 边栏选项卡。

1.  从 **az12001a-vm1** 边栏选项卡，导航到 **az12001a-vm1 - 磁盘** 边栏选项卡。

1.  在 **az12001a-vm1 - Disks** 边栏选项卡，使用以下设置将数据磁盘附加到 az12001a-vm1：

    -   LUN： **0**

    -   磁盘名称： **az12001a-vm1-DataDisk0**

    -   资源组： **az12001a-RG**

    -   主机缓存： **只读**

1.  重复上一步，使用前缀 **az12001a-vm1-DataDisk** 附加剩余的 7 个磁盘（总共 8 个）。分配与磁盘名的最后一个字符匹配的 LUN 编号。使用 LUN **1** 将磁盘的主机缓存设置为 **只读** ，而其他部分，将主机缓存设置为 **无**。

1.  保存你的更改。 

> **结果**：完成此练习后，你已配置了支持高可用性 SAP HANA 部署所必需的 Azure 计算资源。


## 练习 2：配置运行 Linux 的 Azure VM 操作系统，以支持高可用性 SAP HANA 安装

持续时间：30 分钟

在本练习中，你将在运行 SUSE Linux Enterprise Server 的 Azure VM 上配置操作系统和存储，以适应 SAP HANA 的群集安装。

### 任务 1：连接到 Azure Linux VM

1.  从 Azure 门户，在 Cloud Shell 启动 Bash 会话。 

1.  在 Cloud Shell 窗格中，运行以下命令以标识你在上一项练习中配置的第一个 Azure VM 公共 IP 地址：

```
RESOURCE_GROUP_NAME='az12001a-RG'

PIP=$(az network public-ip show --resource-group $RESOURCE_GROUP_NAME --name az12001a-vm0-ip --query ipAddress --output tsv)
```

1.  在 Cloud Shell 窗格中，运行以下命令，与前一步骤中标识的 IP 地址 建立 SSH 会话：

```
ssh student@$PIP
```

1.  当系统提示你是否确定要继续连接时，请键入 `yes` 并按 **回车**。 

1.  提示输入密码时，输入 `Pa55w.rd1234` 并按 **回车** 键。 

1.  通过单击 Cloud Shell 工具栏中的 **打开新会话** 图标，打开另一个 Cloud Shell Bash 会话。

1.  在新打开的 Cloud Shell Bash 会话中，重复此任务中的所有步骤，以通过其 IP 地址 **az12001a-vm0-ip** 连接到 **az12001a-vm1** Azure VM。


### 任务 2：配置运行 Linux 的 Azure VM 存储

1.  在 Cloud Shell 窗格中，在 az12001a-vm0 的 SSH 会话中，运行以下命令以提升特权，并在出现提示时提供 Student 用户帐户的密码： 

```
sudo su -
```

1.  提示输入密码时，输入 `Pa55w.rd1234` 并按 **回车** 键。 

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，运行 `lsscsi` 以标识新添加的设备与其 LUN 编号之间的映射：
    
```
lsscsi
```

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令为 6 个（总共 8 个）数据磁盘创建物理卷：
    
```
pvcreate /dev/sdc
pvcreate /dev/sdd
pvcreate /dev/sde
pvcreate /dev/sdf
pvcreate /dev/sdg
pvcreate /dev/sdh
```

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令创建卷组：
    
```
vgcreate vg_hana_data /dev/sdc /dev/sdd
vgcreate vg_hana_log /dev/sde /dev/sdf
vgcreate vg_hana_backup /dev/sdg /dev/sdh
```

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令创建逻辑卷：

```
lvcreate -l 100%FREE -n hana_data vg_hana_data
lvcreate -l 100%FREE -n hana_log vg_hana_log
lvcreate -l 100%FREE -n hana_backup vg_hana_backup
```

    > **注意**：我们为每个卷组创建一个逻辑卷

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令格式来格式化逻辑卷：

```
mkfs.xfs /dev/vg_hana_data/hana_data
mkfs.xfs /dev/vg_hana_log/hana_log
mkfs.xfs /dev/vg_hana_backup/hana_backup
```

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令分区 **/dev/sdi** 磁盘：

```
fdisk /dev/sdi
```

1.  出现提示时，请按顺序键入`n`、`p`、`1`（每次输入后按 **回车**）按 **回车** 两次，然后键入`w`以完成写入。

1.  在 Cloud Shell 窗格中，在 az12001a-vm0 的 SSH 会话中，运行以下命令对 **/dev/sdj** 磁盘进行分区：

```
fdisk /dev/sdj
```

1.  出现提示时，请按顺序键入`n`、`p`、`1`（每次输入后按 **回车**）按 **回车** 两次，然后键入`w`以完成写入。

1.  在 Cloud Shell 窗格中，在 az12001a-vm0 的 SSH 会话中，运行以下命令格式化新建的分区：

```
mkfs.xfs /dev/sdi -f
mkfs.xfs /dev/sdj -f
```

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令创建将用作装载点的目录：

```
mkdir -p /hana/data
mkdir -p /hana/log
mkdir -p /hana/backup
mkdir -p /hana/shared
mkdir -p /usr/sap
```

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令显示逻辑卷的 ID：

```
blkid
```

    > **注意**：确定 **UUID** 与新创建的卷组和分区关联的值，包括 **/dev/sdi**（用于 **/hana/shared**）和 **/dev/sdj** （用于 **/usr/sap**）。


1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令打开 vi 编辑器（可自由使用任何其他编辑器） 中的 **/etc/fstab**：

```
vi /etc/fstab
```

1.  在编辑器中，将以下条目添加到 **/etc/fstab** （其中， `\<UUID of /dev/vg\_hana\_data-hana\_data\>`、 `\<UUID of /dev/vg\_hana\_log-hana\_log\>`、`\<UUID of /dev/vg\_hana\_backup-hana\_backup\>`、`\<UUID of /dev/vg\_hana\_shared-hana\_shared\>` 和 `\<UUID of /dev/vg\_usr\_sap-usr\_sap\>` 代表你在上一步标识的 ID）：

```
/dev/disk/by-uuid/<UUID of /dev/vg_hana_data-hana_data> /hana/data xfs  defaults,nofail  0  2
/dev/disk/by-uuid/<UUID of /dev/vg_hana_log-hana_log> /hana/log xfs  defaults,nofail  0  2
/dev/disk/by-uuid/<UUID of /dev/vg_hana_backup-hana_backup> /hana/backup xfs  defaults,nofail  0  2
/dev/disk/by-uuid/<UUID of /dev/vg_hana_shared-hana_shared (/dev/sdi)> /hana/shared xfs  defaults,nofail  0  2
/dev/disk/by-uuid/<UUID of /dev/vg_usr_sap-usr_sap (/dev/sdj)> /usr/sap xfs  defaults,nofail  0  2
```

1.  保存更改并关闭编辑器。

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令装载新卷：

```
mount -a
```

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令验证装载是否成功：

```
df -h
```

1.  切换到 az12001a-vm1 的 Cloud Shell Bash 会话，并重复此任务中的所有步骤以在 **az12001a-vm1** 上配置存储。


### 任务 3：启用跨节点无密码 SSH 访问

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令生成无密码的 SSH 密钥：

```
ssh-keygen -tdsa
```

1.  出现提示时，按 **回车** 三次，然后通过运行以下命令显示公钥： 

```
cat /root/.ssh/id_dsa.pub
```

1.  将密钥值复制到剪贴板中。

1.  切换到包含 **az12001a-vm1**  的 SSH 会话的 Cloud Shell 窗格，并通过运行以下命令创建目录 **/root/.ssh/**：

```
mkdir /root/.ssh
```

1.  在 Cloud Shell 窗格，在与 az12001a-vm1 的 SSH 会话中，通过运行以下命令在 vi 编辑器（可自由使用任何其他编辑器）创建 **/root/.ssh/authorized\_keys** 文件：

```
vi /root/.ssh/authorized_keys
```

1.  在编辑器窗口中，粘贴你在 az12001a-vm0 上生成的密钥。

1.  保存更改并关闭编辑器。

1.  在 Cloud Shell 窗格，在与 az12001a-vm1 的 SSH 会话中，通过运行以下命令生成无密码的 SSH 密钥：

```
ssh-keygen -tdsa
```

1.  出现提示时，按 **回车** 三次，然后通过运行以下命令显示公钥： 

```
cat /root/.ssh/id_dsa.pub
```

1.  将密钥值复制到剪贴板中。

1.  切换到包含 az12001a-vm0 的 SSH 会话的 Cloud Shell 窗格，通过运行以下命令在 vi 编辑器（可自由使用任何其他编辑器）创建 **/root/.ssh/authorized\_keys** 文件：

```
vi /root/.ssh/authorized_keys
```

1.  在编辑器窗口中，粘贴你在 az12001a-vm1 上生成的密钥。

1.  保存更改并关闭编辑器。

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令生成无密码的 SSH 密钥：

```
ssh-keygen -t rsa
```

1.  出现提示时，按 **回车** 三次，然后通过运行以下命令显示公钥： 

```
cat /root/.ssh/id_rsa.pub
```

1.  将密钥值复制到剪贴板中。

1.  切换到包含 **az12001a-vm1** 的 SSH 会话的 Cloud Shell 窗格，通过运行以下命令在 vi 编辑器（可自由使用任何其他编辑器）打开 **/root/.ssh/authorized\_keys** 文件：

```
vi /root/.ssh/authorized_keys
```

1.  在编辑器窗口中，从新的一行开始，粘贴你在 az12001a-vm0 上生成的密钥。

1.  保存更改并关闭编辑器。

1.  在 Cloud Shell 窗格，在与 az12001a-vm1 的 SSH 会话中，通过运行以下命令生成无密码的 SSH 密钥：

```
ssh-keygen -t rsa
```

1.  出现提示时，按 **回车** 三次，然后通过运行以下命令显示公钥： 

```
cat /root/.ssh/id_rsa.pub
```

1.  将密钥值复制到剪贴板中。

1.  切换到包含 az12001a-vm0 的 SSH 会话的 Cloud Shell 窗格，通过运行以下命令在 vi 编辑器（可自由使用任何其他编辑器）打开 **/root/.ssh/authorized\_keys** 文件：

```
vi /root/.ssh/authorized_keys
```

1.  在编辑器窗口中，从新的一行开始，粘贴你在 az12001a-vm1 上生成的密钥。

1.  保存更改并关闭编辑器。

1.  在 Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令在 vi 编辑器（可自由使用任何其他编辑器）打开 **/etc/ssh/sshd\_config** 文件：

```
vi /etc/ssh/sshd_config
```

1.  在 **/etc/ssh/sshd\_config** 文件里，找到 **PermitRootLogin** 和 **AuthorizedKeysFile** 条目，并按如下方式配置它们（如果需要，请删除前导 # 字符：
```
PermitRootLogin yes
AuthorizedKeysFile      /root/.ssh/authorized_keys
```

1.  保存更改并关闭编辑器。

1.  Cloud Shell 窗格，在与 az12001a-vm0 的 SSH 会话中，通过运行以下命令重启 sshd daemon：

```
systemctl restart sshd
```

1.  在 az12001a-vm1 上重复前面的四个步骤。

1.  要验证配置是否成功，请在“Cloud Shell”窗格中，在 az12001a-vm0 的 SSH 会话中，通过运行以下命令建立从 az12001a-vm0 到 az12001a-vm1 的 SSH 会话，作为 **根**： 

```
ssh root@az12001a-vm1
```

1.  当系统提示你是否确定要继续连接时，请键入 `yes` 并按 **回车**。 

1.  确保未提示你输入密码。

1.  通过运行以下命令，关闭从 az12001a-vm0 到 az12001a-vm1 的 SSH 会话： 

```
exit
```

1.  通过运行以下命令两次，从 az12001a-vm0 注销：

```
exit
```

1.  要验证配置是否成功，请在“Cloud Shell”窗格中，在 az12001a-vm1 的 SSH 会话中，通过运行以下命令建立从 az12001a-vm1 到 az12001a-vm0 的 SSH 会话，作为 **根**： 

```
ssh root@az12001a-vm0
```

1.  当系统提示你是否确定要继续连接时，请键入 `yes` 并按 **回车**。 

1.  确保未提示你输入密码。

1.  通过运行以下命令，关闭从 az12001a-vm1 到 az12001a-vm0 的 SSH 会话： 

```
exit
```

1.  通过运行以下命令两次，从 az12001a-vm1 注销：

```
exit
```

> **结果**：完成本练习后，你已配置运行 Linux 的 Azure VM 操作系统，以支持高可用性 SAP HANA 安装


## 练习 3：预配支持高可用性 SAP HANA 部署所需的 Azure 网络资源

持续时间：30 分钟

在本练习中，你将实施 Azure 负载均衡器以适应 SAP HANA 的群集安装。


### 任务 1：配置 Azure VM 以辅助负载均衡设置。

   > **注意**：由于你将设置一对 Stardard SKU 的 Azure 负载均衡器，因此你需要首先删除与两个 Azure VM 网络适配器关联的公共 IP 地址，这两个 Azure VM 将用作负载均衡后端池。

1.  在 Azure 门户中，导航到 **az12001a-vm0** Azure VM 的边栏选项卡。

1.  从 **az12001a-vm0** 边栏选项卡，导航到与其网络适配器相关联的 **az12001a-vm0-ip** 公共 IP 地址边栏选项卡。

1.  来自 **az12001a-vm0-ip** 边栏选项卡，首先解除公共 IP 地址与网络接口的关联，然后将其删除。

1.  在 Azure 门户中，导航到 **az12001a-vm1** Azure VM 的边栏选项卡。

1.  从 **az12001a-vm1** 边栏选项卡，导航到与其网络适配器相关联的 **az12001a-vm1-ip** 公共 IP 地址边栏选项卡。

1.  来自 **az12001a-vm1-ip** 边栏选项卡，首先解除公共 IP 地址与网络接口的关联，然后将其删除。

1.  在 Azure 门户中，导航到 **az12001a-vm0** Azure VM 的边栏选项卡。

1.  从 **az12001a-vm0** 边栏选项卡，导航到 **网络** 边栏选项卡。 

1.  从**az12001a-vm0 - 网络** 边栏选项卡，导航到 az12001a-vm0 的网络接口。 

1.  从 az12001a-vm0 的网络接口边栏选项卡，导航到其 IP 配置边栏选项卡，显示其 **ipconfig1** 边栏选项卡。

1.  在 **ipconfig1** 边栏选项卡上，将专用 IP 地址分配设置为 **静态**，然后保存更改。

1.  在 Azure 门户中，导航到 **az12001a-vm1** Azure VM 的边栏选项卡。

1.  从 **az12001a-vm1** 边栏选项卡，导航到 **网络** 边栏选项卡。 

1.  从 **az12001a-vm1 - 网络** 边栏选项卡，导航到 az12001a-vm1 的网络接口。 

1.  从 az12001a-vm1 的网络接口边栏选项卡，导航到其 IP 配置边栏选项卡，显示其 **ipconfig1** 边栏选项卡。

1.  在 **ipconfig1** 边栏选项卡上，将专用 IP 地址分配设置为 **静态**，然后保存更改。


### 任务 2：创建和配置处理入站流量的 Azure 负载均衡器

1.  在 Azure 门户上，单击**+新建资源**。

1.  从**新建**边栏选项卡中，使用以下设置创建新的 Azure 负载均衡器：

    -   订阅：*你的 Azure 订阅名*

    -   资源组：**az12001a-RG**

    -   名称： **az12001a-lb0**

    -   区域：*在本实验室的第一项练习中部署 Azure VM 的相同 Azure 区域*

    -   类型：**内部**

    -   SKU：**标准**

    -   虚拟网络：**az12001a-RG-vnet**

    -   子网：**subnet-0**

    -   IP 地址分配：**静态**

    -   IP 地址: **192.168.0.240**

    -   可用性区域：**区域冗余**

1.  等到负载均衡器预配完成，然后导航到其在 Azure 门户的边栏选项卡。

1.  从 **az12001a-lb0** 边栏选项卡，添加具有以下设置的后端池：

    -   名称：**az12001a-lb0-bepool**

    -   虚拟网络：**az12001a-rg-vnet**

    -   虚拟机：**az12001a-vm0**  IP 地址：**ipconfig1**

    -   虚拟机：**az12001a-vm1**  IP 地址：**ipconfig1**

1.  从 **az12001a-lb0**- 边栏选项卡，添加具有以下设置的健康状况探测：

    -   名称：**az12001a-lb0-hprobe**

    -   协议：**TCP**

    -   端口：**62500**

    -   间隔：**5** *秒*

    -   不正常阈值：**2 个** *连续故障*

1.  从 **az12001a-lb0** 边栏选项卡，添加具有以下设置的网络负载均衡规则：

    -   名称：**az12001a-lb0-lbruleAll**

    -   IP 版本：**IPv4**

    -   前端 IP 地址：**192.168.0.240 (LoadBalancerFrontEnd)**

    -   HA 端口：**已启用**

    -   后端池：**az12001a-lb0-bepool（2 台虚拟机）**

    -   运行状况探测：**az12001a-lb0-hprobe (TCP:62504)**

    -   会话持久性：**无**

    -   空闲超时（分钟）：**4**

    -   浮动 IP（直接服务器返回）：**已启用**

### 任务 3：创建和配置处理出站流量的 Azure 负载均衡器

1.  从 Azure 门户，在 Cloud Shell 启动 Bash 会话。 

1.  在 Cloud Shell 窗格中，运行以下命令，创建第二个负载均衡器将使用的公共 IP 地址：

```
RESOURCE_GROUP_NAME='az12001a-RG'

LOCATION=$(az group list --query "[?name == '$RESOURCE_GROUP_NAME'].location" --output tsv)

PIP_NAME='az12001a-lb1-pip'

az network public-ip create --resource-group $RESOURCE_GROUP_NAME --name $PIP_NAME --sku Standard --location $LOCATION
```

1.  在 Cloud Shell 窗格中，运行以下命令以创建第二个负载均衡器：

```
LB_NAME='az12001a-lb1'

LB_BE_POOL_NAME='az12001a-lb1-bepool'

LB_FE_IP_NAME='az12001a-lb1-fe'

az network lb create --resource-group $RESOURCE_GROUP_NAME --name $LB_NAME --sku Standard --backend-pool-name $LB_BE_POOL_NAME --frontend-ip-name $LB_FE_IP_NAME --location $LOCATION --public-ip-address $PIP_NAME
```

1.  在 Cloud Shell 窗格中，运行以下命令以创建第二个负载均衡器的出站规则：

```
LB_RULE_OUTBOUND='az12001a-lb1-ruleoutbound'

az network lb outbound-rule create --resource-group $RESOURCE_GROUP_NAME --lb-name $LB_NAME --name $LB_RULE_OUTBOUND --frontend-ip-configs $LB_FE_IP_NAME --protocol All --idle-timeout 4 --outbound-ports 1000 --address-pool $LB_BE_POOL_NAME
```

1.  关闭 Cloud Shell 窗格。

1.  在 Azure 门户中，导航到显示 Azure 负载均衡器 **az12001a-lb1** 属性的边栏选项卡。

1.  在 **az12001a-lb1** 边栏选项卡，单击 **后端池**。

1.  在 **az12001a-lb1 - 后端池** 边栏选项卡，单击 **az12001a-lb1-bepool**。

1.  在 **az12001a-lb1-bepool** 边栏选项卡，指定以下设置并单击 **保存**：

    -   虚拟网络：**az12001a-rg-vnet（2个 VM）**

    -   虚拟机：**az12001a-vm0** IP 地址：**ipconfig1**

    -   虚拟机：**az12001a-vm1** IP 地址：**ipconfig1**

### 任务 4：部署跳转主机

   > **注意**：由于无法再从 Internet 直接访问两个群集 Azure VM，因此你将部署运行 Windows Server 2019 Datacenter 的 Azure VM，该 VM 将用作跳转主机。 

1.  在实验室计算机的 Azure 门户上单击 **+创建资源**。

1.  在 **新建** 边栏选项卡中，基于 **Windows Server 2019 Datacenter** 图像，开始创建新的 Azure VM。

1.  使用以下设置预配 Azure VM：

    -   订阅：*你的 Azure 订阅名*

    -   资源组：**az12001a-RG**

    -   虚拟机名称：**az12001a-vm2**

    -   区域：*在本实验室的第一项练习中部署 Azure VM 的相同 Azure 区域*

    -   可用性选项：**无需基础结构冗余**

    -   图像：**Windows Server 2019 Datacenter**

    -   大小：**标准 DS1 v2**

    -   用户名：**Student**

    -   密码：**Pa55w.rd1234**

    -   公共入站端口：**允许选定的端口**

    -   选定的入站端口：**RDP (3389)**

    -   是否已拥有 Windows 许可证？：**否**

    -   操作系统磁盘类型：**标准 HDD**

    -   虚拟网络：**az12001a-RG-vnet**

    -   子网名称： **subnet-0 (192.168.0.0/24)**

    -   公共 IP 地址：*新 IP 地址名为* **az12001a-vm2-ip**

    -   NIC 网络安全组：**基本**

    -   公共入站端口：**允许选定的端口**

    -   选择入站端口：**RDP (3389)**

    -   加速网络：**关**

    -   将此虚拟机置于现有负载均衡解决方案之后：**否**

    -   免费启用基本计划：**否**

    -   启动诊断：**关**

    -   OS 来宾诊断：**关**

    -   系统分配的托管标识：**关**

    -   使用 AAD 凭据登录（预览）：**关**

    -   启用自动关闭：**关**

    -   启用备份：**关**

    -   扩展：*无*

    -   标签：**无**

1.  等待预配完成。这可能需要几分钟。

1.  通过 RDP 连接到新预配的 Azure VM。 

1.  在与 az12001a-vm2 的 RDP 会话中，从以下网址下载 PuTTY：[**https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html**](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)。

1.  确保你可以通过其专用 IP 地址（分别为 192.168.0.4 和 192.168.0.5）与 az12001a-vm0 和 az12001a-vm1 建立 SSH 会话。

> **结果**：完成此练习后，你已配置了支持高可用 SAP HANA 部署所必需的 Azure 网络资源


## 练习 4：删除实验室资源

持续时间：10 分钟

在本练习中，你将删除本实验室中预配的所有资源。

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 **Cloud Shell** 图标以打开“Cloud Shell”窗格，然后选择“Bash”作为 Shell。

1. 在门户底部的 **Cloud Shell** 命令提示符下，键入以下命令，然后按 **回车** 列出你在此练习中创建的所有资源组：

```
az group list --query "[?starts_with(name,'az12001a-')]".name --output tsv
```

1. 验证输出中仅包含你在本实验中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在 **Cloud Shell** 命令提示符处，键入以下命令，然后按 **回车** 删除你在此实验中创建的资源组。

```
az group list --query "[?starts_with(name,'az12001a-')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
```

1. 关闭门户底部的 **Cloud Shell** 提示符。


> **结果**：完成本练习后，你已经删除了本实验中使用的资源。
