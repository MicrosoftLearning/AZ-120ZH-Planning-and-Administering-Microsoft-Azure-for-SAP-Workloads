# AZ 120 模块 1：Azure 上的 SAP 认证产品
# 实验室 1b：在 Azure VM 上实施 Windows 群集

预计用时：120 分钟

本实验室中的所有任务都是从 Azure 门户执行的（包括 PowerShell Cloud Shell 会话）  

   > **注意**：如果不使用 Cloud Shell，则必须在实验室虚拟机上安装 Az PowerShell 模块，获取网址：[**https://docs.microsoft.com/zh-cn/powershell/azure/install-az-ps-msi?view=azps-2.8.0**](https://docs.microsoft.com/zh-cn/powershell/azure/install-az-ps-msi?view=azps-2.8.0).

实验室文件：无

## 方案
  
为了准备在 Azure 上部署 SAP NetWeaver，使用 SQL Server 作为数据库管理系统，Adatum Corporation 希望探索在运行 Windows Server 2019 的 Azure VM 上实施群集的过程。

## 目标
  
完成本实验室后，你将能够：

-   预配支持高可用性 SAP NetWeaver 部署所需的 Azure 计算资源

-   配置运行 Windows Server 2019 的 Azure VM 的操作系统，以支持高可用性 SAP NetWeaver 部署。

-   配置支持高可用性 SAP NetWeaver 部署所需的 Azure 网络资源。

## 要求

-   具有足够数目可用 DSv3 vCPU (4 x 4) 和 DSv2 (1 x 1) vCPU 的 Microsoft Azure 订阅

-   运行 Windows 10、Windows Server 2016 或 Windows Server 2019 的实验室计算机可以访问 Azure

## 练习 1：配置支持高可用性 SAP NetWeaver 部署所需的 Azure 计算资源。

持续时间：50 分钟

在本练习中，你将部署在运行 Windows Server 2019 的 Azure VM 上配置故障转移群集所必需的 Azure 基础结构计算组件。这将涉及部署一对 Active Directory 域控制器，然后在同一虚拟网络中的同一可用性集内部署一对运行 Windows Server 2019的 Azure VM。要自动部署域控制器，你将使用 Azure 资源管理器快速启动模板，下载网址：<https://github.com/Azure/azure-quickstart-templates/tree/master/active-directory-new-domain-ha-2-dc>

### 任务 1：使用 Azure 资源管理器模板，部署一对运行高可用性 Active Directory 域控制器的 Azure VM

1.  从实验室计算机中启动网页浏览器，并导航到 Azure 门户网站：https://portal.azure.com

1.  如果出现提示，则使用你在本实验室中使用的 Azure 订阅的所有者或参与者角色登录工作或学校或个人 Microsoft 帐户。

1.  在 Azure 门户上，单击 **+新建资源**。

1.  从 **新建** 边栏选项卡，新建 **模板部署（使用自定义模板部署）**

1.  从 **自定义部署** 边栏选项卡，在 **加载 GitHub 快速启动模板** 下拉列表中，选择条目 **active-directory-new-domain-ha-2-dc**，并单击 **选择模板**。

    > **注意**：或者，你可以通过导航到位于 <https://github.com/Azure/azure-quickstart-templates> 上的 Azure 快速启动模板页面来启动部署，找到以下名称的模板 **新建 2 个 Windows VM、在可用性集中新建 AD 林、域和 2 个 DC**，并通过单击 **部署到 Azure 按钮**启动其部署。

1.  在 **使用 2 个域控制器新建 AD 域** 边栏选项卡上，单击 **编辑模板**。 

1.  在 **编辑模板** 边栏选项卡上，找到将值分配给 **adVMSize** 变量的行：

```
    "adVMSize": "Standard_DS2_v2"

```

1.  在 **编辑模板** 边栏选项卡上，将 **adVMSize** 变量的值设置为 **Standard_D4S_v3**，然后单击 **保存**。

```
    "adVMSize": "Standard_D4s_v3"

```

1.  在 **使用 2 个域控制器新建 AD 域** 边栏选项卡，指定以下设置并单击“购买”启动部署：

    -   订阅：*你的 Azure 订阅名*

    -   资源组：*新资源组，名为* **az12001b-ad-RG**

    -   位置： *供你部署 Azure VM 的 Azure 区域，该区域离实验室位置最近*

    -   管理员用户名：**Student**

    -   密码：**Pa55w.rd1234**

    -   域名：**adatum.com**

    -   DnsPrefix：*任何唯一有效的 DNS 前缀*

    -   Pdc RDP 端口：**3389**

    -   Bdc RDP 端口：**13389**

    -   _artifacts 位置：**https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/active-directory-new-domain-ha-2-dc/**

    -   _artifacts 位置 Sas 令牌：*留空*

    -   位置：**[resourceGroup().location]**

    -   我同意以上条款和条件：*已启用*

    > **注意**：本部署会花上35分钟左右。请耐心等待部署完成，再继续执行下一个任务。

### 任务 2：在同一可用性集中部署运行 Windows Server 2016 的一对 Azure VM。

1.  在实验室计算机的 Azure 门户上单击 **+创建资源**。

1.  从 **新建** 边栏选项卡，使用以下设置开始预配 **Windows Server 2019 Datacenter** Azure VM：

    -   订阅：*你的 Azure 订阅名*

    -   资源组：*新资源组，名为* **az12001b-cl-RG**

    -   虚拟机名称：**az12001b-cl-vm0**

    -   区域：*上一个任务中部署 Azure VM 的 Azure 区域*

    -   可用性选项：**可用性集**

    -   可用性集：*新可用性集名为* **az12001b-cl-avset** *具备 2 个容错域与 5 个更新域*

    -   图像：**Windows Server 2019 Datacenter**

    -   大小：**标准 D4s v3**

    -   用户名：**Student**

    -   密码：**Pa55w.rd1234**

    -   公共入站端口：**允许选定的端口**

    -   选择入站端口：**RDP (3389)**

    -   是否已拥有 Windows 许可证？：**否**

    -   操作系统磁盘类型：**高级 SSD**

    -   虚拟网络：**adVNET**

    -   子网名：*新子网名为* **clSubnet**

    -   子网地址范围：**10.0.1.0/24**

    -   公共 IP 地址：*新 IP 地址，名为* **az12001b-cl-vm0-ip**

    -   NIC 网络安全组：**基本**

    -   公共入站端口：**允许选定的端口**

    -   选择入站端口：**RDP (3389)**

    -   加速网络：**开**

    -   将此虚拟机置于现有负载均衡解决方案之后：**否**

    -   免费启用基本计划：**否**

    -   启动诊断：**关**

    -   OS 来宾诊断：**关**

    -   系统分配的托管标识：**关**

    -   使用 AAD 凭据登录（预览）：**关**

    -   启用自动关闭：**关**

    -   启用备份：**关**

    -   扩展：*无*

    -   标签：*无*

1.  不用等待预配完成，继续进行下一项步骤。

1.  使用以下设置预配另一个 **Windows Server 2019 Datacenter** Azure VM：

    -   订阅：*你的 Azure 订阅名*

    -   资源组：**az12001b-cl-RG**

    -   虚拟机名称：**az12001b-cl-vm1**

    -   区域：*上一个任务中部署 Azure VM 的 Azure 区域*

    -   可用性选项：**可用性集**

    -   可用性集：**az12001b-cl-avset**

    -   图像：**Windows Server 2019 Datacenter**

    -   大小：**标准 D4s v3**

    -   用户名：**Student**

    -   密码：**Pa55w.rd1234**

    -   公共入站端口：**允许选定的端口**

    -   选择入站端口：**RDP (3389)**

    -   是否已拥有 Windows 许可证？：**否**

    -   操作系统磁盘类型：**高级 SSD**

    -   虚拟网络：**adVNET**

    -   子网名：**clSubnet**

    -   公共 IP 地址：*新 IP 地址，名为* **az12001b-cl-vm1-ip**

    -   NIC 网络安全组：**基本**

    -   公共入站端口：**允许选定的端口**

    -   选择入站端口：**RDP (3389)**

    -   加速网络：**开**

    -   将此虚拟机置于现有负载均衡解决方案之后：**否**

    -   免费启用基本计划：**否**

    -   启动诊断：**关**

    -   OS 来宾诊断：**关**

    -   系统分配的托管标识：**关**

    -   使用 AAD 凭据登录（预览）：**关**

    -   启用自动关闭：**关**

    -   启用备份：**关**

    -   扩展：*无*

    -   标签：*无*

1.  等待预配完成。这可能需要几分钟。

### 任务 3：创建并配置 Azure VM 磁盘

1.  在 Azure 门户 Cloud Shell 中启动 PowerShell 会话。 

    > **注意**：如果这是你第一次在当前 Azure 订阅中启动 Cloud Shell，则会要求你创建 Azure 文件共享以保留 Cloud Shell 文件。如果是，请接受默认值，将在自动生成的资源组中创建存储帐户。

1.  在 Cloud Shell 窗格中，运行以下命令，以创建第一组共计 4 个托管磁盘，这些磁盘将附加到你在上一个任务中部署的第一个 Azure VM：

```
    $resourceGroupName = 'az12001b-cl-RG'

    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $diskConfig = New-AzDiskConfig -Location $location -DiskSizeGB 128 -AccountType Premium_LRS -OsType Windows -CreateOption Empty

    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm0-DataDisk$i -Disk $diskConfig}
```

1.  在 Cloud Shell 窗格中，运行以下命令，以创建第二组共计 4 个托管磁盘，这些磁盘将附加到你在上一个任务中部署的第二个 Azure VM：

```
    for ($i=0;$i -lt 4;$i++) {New-AzDisk -ResourceGroupName $resourceGroupName -DiskName az12001b-cl-vm1-DataDisk$i -Disk $diskConfig}
```

1.  从 Azure 门户导航到你在上一个任务 (**az12001b-cl-vm0**) 中预配的第一个 Azure VM。

1.  从 **az12001b-cl-vm0** 边栏选项卡导航到 **az12001b-cl-vm0 - 磁盘** 边栏选项卡。

1.  在 **az12001b-cl-vm0 - 磁盘** 边栏选项卡中，使用以下设置将数据磁盘附加到 az12001b-cl-vm0：

    -   LUN：**0**

    -   磁盘名称： **az12001b-cl-vm0-DataDisk0**

    -   资源组：**az12001b-cl-RG**

    -   主机缓存：**只读**

1.  重复上一步，使用前缀 **az12001b-cl-vm0-DataDisk** 附加剩余的 3 个磁盘（总共4个）。分配与磁盘名的最后一个字符匹配的 LUN 编号。对于最后一个磁盘 (LUN **4**)，将主机缓存设置为 **没有**。

1.  保存你的更改。 

1.  从 Azure 门户导航到你在上一个任务 (**az12001b-cl-vm1**) 中预配的第二个 Azure VM。

1.  从 **az12001b-cl-vm1** 边栏选项卡导航到 **az12001b-cl-vm1 - 磁盘** 边栏选项卡。

1.  在 **az12001b-cl-vm1 - 磁盘** 边栏选项卡中，使用以下设置将数据磁盘附加到 az12001b-cl-vm1：

    -   LUN：**0**

    -   磁盘名称： **az12001b-cl-vm1-DataDisk0**

    -   资源组：**az12001b-cl-RG**

    -   主机缓存：**只读**

1.  重复上一步，使用前缀 **az12001b-cl-vm1-DataDisk** 附加剩余的 3 个磁盘（总共4个）。分配与磁盘名的最后一个字符匹配的 LUN 编号。对于最后一个磁盘 (LUN **4**)，将主机缓存设置为 **没有**。

1.  保存你的更改。 

> **结果**：完成本练习后，你已预配了支持高可用 SAP NetWeaver 部署所需的 Azure 计算资源。


## 练习 2：配置运行 Windows Server 2019 的 Azure VM 操作系统，以支持高可用性 SAP NetWeaver 安装

持续时间：40 分钟

### 任务 1：将 Windows Server 2019 Azure VM 加入 Active Directory 域。

   > **注意**：开始此任务之前，请确保你在上个练习的最后一个任务中启动的模板部署已经完成。 

1.  在 Azure 门户中，导航到 **adVNET** 虚拟网络的边栏选项卡，该虚拟网络已在本实验室的第一个练习中自动预配。

1.  显示 **adVNET - DNS 服务器** 边栏选项卡，请注意，虚拟网络配置了专用 IP 地址，该地址是分配给本实验室第一个练习中部署的域控制器，作为其 DNS 服务器。

1.  在 Azure 门户 Cloud Shell 中启动 PowerShell 会话。 

1.  在 Cloud Shell 窗格中，运行以下命令，将你在上一个练习的第二个任务中部署的 Windows Server 2019 Azure VM 加入到 **adatum.com** Active Directory域：

```
    $resourceGroupName = 'az12001b-cl-RG'

    $location = (Get-AzureRmResourceGroup -Name $resourceGroupName).Location

    $settingString = '{"Name": "adatum.com", "User": "adatum.com\\Student", "Restart": "true", "Options": "3"}'

    $protectedSettingString = '{"Password": "Pa55w.rd1234"}'

    $vmNames = @('az12001b-cl-vm0','az12001b-cl-vm1')

    foreach ($vmName in $vmNames) { Set-AzVMExtension -ResourceGroupName $resourceGroupName -ExtensionType 'JsonADDomainExtension' -Name 'joindomain' -Publisher "Microsoft.Compute" -TypeHandlerVersion "1.0" -Vmname $vmName -Location $location -SettingString $settingString -ProtectedSettingString $protectedSettingString }
```

1.  请等待脚本完成，再继续执行下一个任务。


### 任务 2：在运行 Windows Server 2019 的 Azure VM 上配置存储，以支持高可用性 SAP NetWeaver 安装。

1.  在 Azure 门户中，导航到 **az12001b-cl-vm0** 虚拟机的边栏选项卡，该虚拟机是你在本实验室的第一个练习中预配的。

1.  从 **az12001b-cl-vm0** 边栏选项卡，使用远程桌面连接到虚拟机来宾操作系统。当提示要身份验证时，请提供以下凭据：

    -   用户名：**ADATUM\Student**

    -   密码：**Pa55w.rd1234**

1.  与在服务器管理器中的 az12001b-cl-vm0 的 RDP 会话中，导航到 **本地服务器** 视图并暂时关闭 **IE 增强安全配置**。

1.  与在服务器管理器中的 az12001b-cl-vm0 的 RDP 会话中，导航到 **文件和存储服务** -> **服务器** 节点。 

1.  导航到 **存储池** 查看并验证你是否看到了在上一个练习中附加到 Azure VM 的所有磁盘。

1.  使用 **新建存储池向导** 边栏选项卡，创建新的存储池，设置如下：

    -   名称：**Data Storage Pool**

    -   物理盘：*选择 3 个磁盘，磁盘编号对应前三个 LUN 编号 (0-2)，并将其分配设置为* **自动**

    > **注意**：使用 **机壳** 列中的条目以标识 **LUN** 编号。

1.  使用 **新建虚拟磁盘向导** 边栏选项卡创建新的虚拟磁盘，设置如下：

    -   虚拟磁盘名称：**数据虚拟磁盘**

    -   存储布局：**简单**

    -   预配：**固定**

    -   大小：**最大**

1.  使用 **新建卷向导** 边栏选项卡创建新的卷，设置如下：

    -   服务器和磁盘：*接受默认值*

    -   大小：*接受默认值*

    -   盘字母：**M**

    -   文件系统：**ReFS**

    -   分配单位大小：**默认**

    -   卷标签：**Data**

1.  回到 **存储池** 查看，按以下设置使用 **新建存储池向导** 新建存储池：

    -   名称： **Log Storage Pool**

    -   物理盘：*选择 4 个磁盘中的最后一个并将其分配设置为* **自动**

1.  使用 **新建虚拟磁盘向导** 边栏选项卡创建新的虚拟磁盘，设置如下：

    -   虚拟磁盘名称：**Log Virtual Disk**

    -   存储布局：**简单**

    -   预配：**固定**

    -   大小：**最大**

1.  使用 **新建卷向导** 边栏选项卡创建新的卷，设置如下：

    -   服务器和磁盘：*接受默认值*

    -   大小：*接受默认值*

    -   盘字母：**L**

    -   文件系统：**ReFS**

    -   分配单位大小：**默认**

    -   卷标签：**日志**

1.  重复此任务中的上一步，在 az12001b-cl-vm1 上配置存储。

### 任务 3：准备在运行 Windows Server 2019 的 Azure VM 上配置故障转移群集，以支持高可用性 SAP NetWeaver 安装。

1.  在与 az12001b-cl-vm0 的 RDP 会话中，通过运行以下命令启动 Windows PowerShell ISE 会话并在 az12001b-cl-vm0 和 az12001b-cl-vm1 上均安装故障转移群集和远程管理工具功能：

```
    $nodes = @('az12001b-cl-vm1', 'az12001b-cl-vm0')

    Invoke-Command $nodes {Install-WindowsFeature Failover-Clustering -IncludeAllSubFeature -IncludeManagementTools} 

    Invoke-Command $nodes {Install-WindowsFeature RSAT -IncludeAllSubFeature -Restart} 
```

> **注意**：这将重启两个 Azure VM 的来宾操作系统。

1.  在实验室计算机的 Azure 门户上单击 **新建资源**。

1.  从 **新建** 边栏选项卡，使用以下设置新建 **存储帐户**：

    -   订阅：*你的 Azure 订阅名*

    -   资源组：**az12001b-cl-RG**

    -   存储帐户名称：*任何由 3 到 24 个字母和数字组成的唯一名称*

    -   位置：*在上一个任务中部署 Azure VM 的相同 Azure 区域*

    -   性能：**标准**

    -   帐户种类：**存储（通用 v1）**

    -   复制：**本地-冗余存储 (LRS)**

    -   连接方式：**公共端点（所有网络）**

    -   需要安全传输：**已启用**

    -   大文件共享：**禁用**

    -   blob 软删除：**禁用**

    -   分层命名空间：**禁用**

### 任务 4：在运行 Windows Server 2019 的 Azure VM 上配置故障转移群集，以支持高可用性 SAP NetWeaver 安装。

1.  在 Azure 门户中，导航到 **az12001b-cl-vm0** 虚拟机的边栏选项卡，该虚拟机是你在本实验室的第一个练习中预配的。

1.  从 **az12001b-cl-vm0** 边栏选项卡，使用远程桌面连接到虚拟机来宾操作系统。当提示要身份验证时，请提供以下凭据：

    -   用户名：**ADATUM\\Student**

    -   密码：**Pa55w.rd1234**

1.  在与 az12001b-cl-vm0 的 RDP 会话中，从服务器管理器的 **工具** 菜单中，启动 **Active Directory 管理中心**。

1.  在 Active Directory 管理中心中，在 adatum.com 域的根目录下创建一个名为 **群集** 的新组织单位。

1.  在 Active Directory 管理中心中，从 **计算机** 容器移动 **az12001b-cl-vm0** 和 **az12001b-cl-vm1** 计算机帐户到 **群集** 组织单位。

1.  在在与 az12001b-cl-vm0 的 RDP 会话中，通过运行以下命令启动 Windows PowerShell ISE 会话并创建新群集：

```
    $nodes = @('az12001b-cl-vm0','az12001b-cl-vm1')

    New-Cluster -Name az12001b-cl-cl0 -Node $nodes -NoStorage -StaticAddress 10.0.1.6
```

1.  在与 az12001b-cl-vm0 的 RDP 会话中，切换到 **Active Directory 管理中心** 控制台。

1.  在 Active Directory 管理中心中，导航到 **群集** 组织单位并展示其 **属性** 窗口。 

1.  在 **群集** 组织单位 **属性** 窗口中，导航到 **扩展** 部分，显示 **安全** 选项卡。 

1.  在 **安全** 选项卡，单击 **高级** 按钮，打开 **群集高级安全设置** 窗口。 

1.  在 **计算机高级安全设置** 窗口的 **权限** 选项卡，单击 **添加**。

1.  在 **群集权限条目** 窗口，单击 **选择主体**

1.  在 **选择用户、服务帐户或组** 对话框，单击 **对象类型**，启用 **计算机** 条目旁边的复选框，然后单击 **确认**。 

1.  回到 **选择用户、计算机、服务帐户或组** 对话框，在 **输入要选择的对象名**，键入 **az12001b-cl-cl0** 并单击 **确认**。

1.  在 **群集权限条目** 窗口，确保 **允许** 出现在 **类型** 下拉列表。接下来，在 **适用于** 下拉列表，选择 **此对象和所有后代对象**。在 **权限** 列表中，选择 **创建计算机对象** 和 **删除计算机对象** 复选框，然后单击 **确认** 两次。

1.  在 Windows PowerShell ISE 会话中，通过运行以下命令安装 Azure PowerShell 模块：

```
    Install-PackageProvider -Name NuGet -Force

    Install-Module -Name Az -Force
```

1.  在 Windows PowerShell ISE 会话中，通过运行以下命令使用 Azure AD 凭据进行身份验证：

```
    Add-AzAccount
```

> **注意**：出现提示时，使用你在本实验室中使用的 Azure 订阅的所有者或参与者角色登录工作或学校或个人 Microsoft 帐户。

1.  在 Windows PowerShell ISE 会话中，通过运行以下命令设置新群集的云见证仲裁：

```
    $resourceGroupName = 'az12001b-cl-RG'

    $cwStorageAccountName = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName)[0].StorageAccountName

    $cwStorageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $resourceGroupName -Name $cwStorageAccountName).Value[0]

    Set-ClusterQuorum -CloudWitness -AccountName $cwStorageAccountName -AccessKey $cwStorageAccountKey
```

1.  如果要在 az12001b-cl-vm0 的 RDP 会话中验证生成的配置，需在服务器管理器的 **工具** 菜单中，启动 **故障转移群集管理器**。

1.  在 **故障转移群集管理器** 控制台，查看 **az12001b-cl-cl0** 的群集配置，包括其节点、见证和网络设置。请注意，群集没有任何共享存储。

1.  终止与 az12001b-cl-vm0 的 RDP 会话。

> **结果**：完成本练习后，你已配置运行 Windows Server 2019 的 Azure VM 操作系统，以支持高可用性 SAP NetWeaver 安装


## 练习 3：预配支持高可用性 SAP NetWeaver 部署所需的 Azure 网络资源

持续时间：30 分钟

在本练习中，你将实施 Azure 负载均衡器以适应 SAP NetWeaver 的群集安装。

### 任务 1：配置 Azure VM 以辅助负载均衡设置。

   > **注意**：由于你将设置一对 Stardard SKU 的 Azure 负载均衡器，因此你需要首先删除与两个 Azure VM 网络适配器关联的公共 IP 地址，这两个 Azure VM 将用作负载均衡后端池。

1.  在实验室计算机上的 Azure 门户中，导航到 **az12001b-cl-vm0** Azure VM 边栏选项卡。 

1.  从 **az12001b-cl-vm0** 边栏选项卡，导航到与其网络适配器相关联的 **az12001b-cl-vm0-ip** 公共 IP 地址边栏选项卡。

1.  在 **az12001b-cl-vm0-ip** 边栏选项卡中，首先解除公共 IP 地址与网络接口的关联，然后将其删除。

1.  在 Azure 门户中，导航到 **az12001b-cl-vm1** Azure VM 边栏选项卡。 

1.  从 **az12001b-cl-vm1** 边栏选项卡，导航到与其网络适配器相关联的 **az12001b-cl-vm1-ip** 公共 IP 地址边栏选项卡。

1.  在 **az12001b-cl-vm1-ip** 边栏选项卡中，首先解除公共 IP 地址与网络接口的关联，然后将其删除。

1.  在 Azure 门户中，导航到 **az12001a-vm0** Azure VM 的边栏选项卡。

1.  从 **az12001a-vm0** 边栏选项卡，导航到**网络**边栏选项卡。 

1.  从**az12001a-vm0 - 网络**边栏选项卡，导航到 az12001a-vm0 的网络接口。 

1.  从 az12001a-vm0 的网络接口边栏选项卡，导航到其 IP 配置边栏选项卡，显示其 **ipconfig1** 边栏选项卡。

1.  在**ipconfig1** 边栏选项卡上，将专用 IP 地址分配设置为**静态**，然后保存更改。

1.  在 Azure 门户中，导航到 **az12001a-vm1** Azure VM 的边栏选项卡。

1.  从 **az12001a-vm1** 边栏选项卡，导航到**网络**边栏选项卡。 

1.  从**az12001a-vm1 - 网络**边栏选项卡，导航到 az12001a-vm1 的网络接口。 

1.  从 az12001a-vm1 的网络接口边栏选项卡，导航到其 IP 配置边栏选项卡，显示其 **ipconfig1** 边栏选项卡。

1.  在**ipconfig1** 边栏选项卡上，将专用 IP 地址分配设置为**静态**，然后保存更改。

### 任务 2：创建和配置处理入站流量的 Azure 负载均衡器

1.  在 Azure 门户上，单击 **+新建资源**。

1.  从 **新建** 边栏选项卡中，使用以下设置创建新的 Azure 负载均衡器：

    -   订阅：*你的 Azure 订阅名*

    -   资源组：**az12001b-cl-RG**

    -   名称：**az12001b-cl-lb0**

    -   区域：*在本实验室的第一项练习中部署 Azure VM 的相同 Azure 区域*

    -   类型：**内部**

    -   SKU：**标准**

    -   虚拟网络：**adVNET**

    -   子网：**clSubnet**

    -   IP 地址分配：**静态**

    -   IP 地址: **10.0.1.240**

    -   可用性区域：**区域冗余**

1.  等到负载均衡器预配完成，然后导航到其在 Azure 门户的边栏选项卡。

1.  从 **az12001b-cl-lb0** 边栏选项卡，添加具有以下设置的后端池：

    -   名称：**az12001b-cl-lb0-bepool**

    -   虚拟网络：**adVNET**

    -   虚拟机：**az12001b-cl-vm0** IP 地址：**ipconfig1**

    -   虚拟机：**az12001b-cl-vm1** IP 地址：**ipconfig1**

1.  从 **az12001b-cl-lb0** 边栏选项卡，添加具有以下设置的健康状况探测：

    -   名称：**az12001b-cl-lb0-hprobe**

    -   协议：**TCP**

    -   端口：**59999**

    -   间隔：**5** *秒*

    -   不正常阈值：**2 个** *连续故障*

1.  从 **az12001b-cl-lb0** 边栏选项卡，添加具有以下设置的负载均衡规则：

    -   名称：**az12001b-cl-lb0-lbruletcp1433**

    -   IP 版本：**IPv4**

    -   前端 IP 地址：**192.168.0.240 (LoadBalancerFrontEnd)**

    -   HA 端口：**禁用**

    -   协议：**TCP**

    -   端口：**1433**

    -   后端端口：**1433**

    -   后端池：**az12001b-cl-lb0-bepool（2 个虚拟机）**

    -   健康状况探测：**az12001b-cl-lb0-hprobe (TCP:59999)**

    -   会话持久性：**无**

    -   空闲超时（分钟）：**4**

    -   浮动 IP（直接服务器返回）：**已启用**

### 任务 3：创建和配置处理出站流量的 Azure 负载均衡器

1.  从 Azure 门户，在 Cloud Shell 窗格中启动 PowerShell 会话。 

1.  在 Cloud Shell 窗格中，运行以下命令，创建第二个负载均衡器将使用的公共 IP 地址：

```
    $resourceGroupName = 'az12001b-cl-RG'

    $location = (Get-AzResourceGroup -Name $resourceGroupName).Location

    $pipName = 'az12001b-cl-lb0-pip'

    az network public-ip create --resource-group $resourceGroupName --name $pipName --sku Standard --location $location
```

1.  在 Cloud Shell 窗格中，运行以下命令以创建第二个负载均衡器：

```
    $lbName = 'az12001b-cl-lb1'

    $lbFeName = 'az12001b-cl-lb1-fe'

    $lbBePoolName = 'az12001b-cl-lb1-bepool'
   
    $pip = Get-AzPublicIpAddress -ResourceGroupName $resourceGroupName -Name $pipName

    $feIpconfiguration = New-AzLoadBalancerFrontendIpConfig -Name $lbFeName -PublicIpAddress $pip

    $bePoolConfiguration = New-AzLoadBalancerBackendAddressPoolConfig -Name $lbBePoolName

    New-AzLoadBalancer -ResourceGroupName $resourceGroupName -Location $location -Name $lbName -Sku Standard -BackendAddressPool $bePoolConfiguration -FrontendIpConfiguration $feIpconfiguration
```

1.  关闭 Cloud Shell 窗格。

1.  在 Azure 门户中，导航到 Azure 负载均衡器 **az12001b-cl-lb1** 边栏选项卡。

1.  在 **az12001b-cl-lb1** 边栏选项卡，单击 **后端池**。

1.  在 **az12001b-cl-lb1 - 后端池** 边栏选项卡，单击 **az12001b-cl-lb1-bepool**。

1.  在 **az12001b-cl-lb1-bepool** 边栏选项卡，指定以下设置并单击 **保存**：

    -   虚拟网络：**adVNET（4 个 VM）**

    -   虚拟机：**az12001b-cl-vm0** IP 地址：**ipconfig1**

    -   虚拟机：**az12001b-cl-vm1** IP 地址：**ipconfig1**

1.  在 **az12001b-cl-lb1** 边栏选项卡，单击 **健康状况探测**。

1.  从 **az12001b-cl-lb1 - 健康状况探测** 边栏选项卡，添加具有以下设置的健康状况探测：

    -   名称：**az12001b-cl-lb1-hprobe**

    -   协议：**TCP**

    -   端口：**80**

    -   间隔：**5** *秒*

    -   不正常阈值：**2 个** *连续故障*

1.  在 **az12001b-cl-lb1** 边栏选项卡，单击 **负载均衡规则**。

1.  从 **az12001b-cl-lb1 - 负载均衡规则** 边栏选项卡，添加具有以下设置的负载均衡规则：

    -   名称：**az12001b-cl-lb1-lbharule**

    -   IP 版本：**IPv4**

    -   前端 IP 地址： *接受默认值*

    -   协议：**TCP**

    -   端口：**80**

    -   后端端口：**80**

    -   后端池：**az12001b-cl-lb1-bepool（2 个虚拟机）**

    -   健康状况探测：**az12001b-cl-lb1-hprobe (TCP:80)**

    -   会话持久性：**无**

    -   空闲超时（分钟）：**4**

    -   浮动 IP（直接服务器返回）：**禁用**

### 任务 4：部署跳转主机

   > **注意**：由于无法再从 Internet 直接访问两个群集 Azure VM，因此你将部署运行 Windows Server 2019 Datacenter 的 Azure VM，该 VM 将用作跳转主机。 

1.  在实验室计算机的 Azure 门户上单击 **+创建资源**。

1.  在 **新建** 边栏选项卡中，基于 **Windows Server 2019 Datacenter** 图像，开始创建新的 Azure VM。

1.  使用以下设置预配 Azure VM：

    -   订阅：*你的 Azure 订阅名*

    -   资源组：**az12001b-cl-RG**

    -   虚拟机名称：**az12001b-vm2**

    -   区域：*在本实验室的第一项练习中部署 Azure VM 的相同 Azure 区域*

    -   可用性选项：**无需基础结构冗余**

    -   图像：**Windows Server 2019 Datacenter**

    -   大小：**标准 DS1 v2**

    -   用户名：**student**

    -   密码：**Pa55w.rd1234**

    -   公共入站端口：**允许选定的端口**

    -   选择入站端口：**RDP (3389)**

    -   是否已拥有 Windows 许可证？：**否**

    -   操作系统磁盘类型：**标准 HDD**

    -   虚拟网络：**adVNET**

    -   子网： *新子网名为* **bastionSubnet**

    -   地址范围：**10.0.255.0/24**

    -   公共 IP 地址：*新 IP 地址名为* **az12001b-vm2-ip**

    -   NIC 网络安全组：**基本**

    -   公共入站端口：**允许选定的端口**

    -   选择入站端口：**RDP (3389)**

    -   加速网络：**关**

    -   将此虚拟机置于现有负载均衡解决方案之后：**否**

    -   启动诊断：**关**

    -   OS 来宾诊断：**关**

    -   系统分配的托管标识：**关**

    -   使用 AAD 凭据登录（预览）：**关**

    -   启用自动关闭：**关**

    -   启用备份：**关**

    -   扩展：*无*

    -   标签：*无*

1.  等待预配完成。这可能需要几分钟。

1.  通过 RDP 连接到新预配的 Azure VM。 

1.  在 az12001b-vm2 的 RDP 会话中，确保你可以通过其专用 IP 地址（分别为 10.0.1.4 和 10.0.1.5）与 az12001b-cl-vm0 和 az12001b-cl-vm1 建立 RDP 会话。 

> **结果**：完成本练习后，你已配置了支持高可用性 SAP NetWeaver 部署所需的 Azure 网络资源

## 练习 4：删除实验室资源

持续时间：10 分钟

在本练习中，你将删除本实验室中预配的所有资源。

#### 任务 1：打开 Cloud Shell

1. 在门户顶部，单击 **Cloud Shell** 图标以打开“Cloud Shell”窗格，然后选择“PowerShell”作为 Shell。

1. 在门户底部的 **Cloud Shell** 命令提示符下，键入以下命令，然后按 **输入**列出你在此练习中创建的所有资源组：

```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like 'az12001b-*'} | Select-Object ResourceGroupName
```

1. 验证输出中仅包含你在本实验中创建的资源组。这些组将在下一个任务中删除。

#### 任务 2：删除资源组

1. 在 **Cloud Shell** 命令提示符处，键入以下命令，然后按 **输入** 删除你在此实验中创建的资源组。

```
    Get-AzResourceGroup | Where-Object {$_.ResourceGroupName -like 'az12001b-*'} | Remove-AzResourceGroup -Force  
```

1. 关闭门户底部的 **Cloud Shell** 提示符。


> **结果**：完成本练习后，你已经删除了本实验中使用的资源。
