# 虚拟机UB设备直通使用说明

UB设备直通技术是一种基于硬件的虚拟化解决方案，通过该技术，虚拟机可以以直通的方式访问UB设备以获取最佳的性能。
在UB设备虚拟化系统中，包含Host、UB Device组件， Host包含了UB Controller、UMMU、Hypervisor等软硬组件。

## 一、UB控制器配置及使用说明

UB总线（灵衢总线）支持任意拓扑结构的连接，节点之间可通过物理设备的port相互连接，port与port之间构成UB link链路。

- UB总线通过UB Controller实现对UB设备的控制管理。

- UB总线支持Host内部实现功能设备UB iDevice，与UB Controller通过vport连接。UB iDevice支持共享使用UB Controller的port。

- UB总线组网支持多路径连接，两个UB节点间的通讯路径可有多条访问路径选择。

- UB支持通过port对接其他host，实现host之间的交互。

### 1.1 配置约束限制

- 单个VM最多支持1个UB控制器

- 单个UB控制器最多支持256个port

- HostOS和GuestOS需支持灵衢总线驱动

- 虚机GuestOS内Cluster Mode仅支持单机模式

### 1.2 控制器配置方法

#### 完整xml配置

```xml
<controller type='ub' index='0' model='ubc'>
    <ports num='10'/>
    <alias name='ua-ubc0'/>
    <source>
        <businstance guid='cc08-a120-0-0-000000-0000000000000123'/>
    </source>
    <address type='ub' eid='0x1' guid='cc08-a120-0-2-000000-0000000000000123'/>
</controller>
```

#### 最简xml配置

```xml
<controller type='ub'>
      <source>
         <businstance guid='cc08-a120-0-0-000000-0000000000000123'/>
      </source>
</controller>
```

#### 配置关键参数说明

|参数名|说明|取值|是否必配|
|---|---|---|---|
|controller.type|表示配置的控制器类型|'ub'|是|
|controller.index|控制器索引，从0开始|'0'，当前仅支持一个ub控制器|否|
|controller.model|ub控制器类型|'ubc'|否|
|controller.ports.num|UB控制器的port数量，虚拟机每添加一个UB设备需占用一个port|配置范围[1, 256]，不配置时libvirt自动生成，默认128|否|
|controller.alias.name|标签配置ub控制器id|自定义时，自定义的alias必须以'ua-'开头，且不能含有'.'，不配置时libvirt自动生成|否|
|controller.source.businstance.guid|用于配置host businstance信息|为虚拟机创建的bus instance的guid，由管理员或者管控面软件自动按guid规则生成，并且需要确保在物理UB clan域内唯一|是|
|controller.address|GuestOS的内部UB控制器总线地址信息|type：配置地址类型'ub'；eid：UB控制器的eid, qemu内部拓扑模拟使用，和虚机内ubus总线自动生成的eid无关；guid：UB控制器的guid|不配置controller.address的情况下，libvirt默认自动生成|

### 1.3 GUID格式及关键说明

格式：VendorId-DeviceId-Version-Type-Reserved-SequenceNumber（示例：cc08-a120-0-0-000000-0000000000000123）

其中各字段说明如下表：

|字段名称|比特位置|说明|
|---|---|---|
|Vendor ID|127:112|器件厂商向UB协议管理组织提出申请，UB协议管理组织集中管理和分配。|
|Device ID|111:96|设备身份编码，由厂商定义。|
|Version|95:92|同一{VendorID, Device ID}设备的子版本编码。常见用法：设备在功能或者性能特性发生变更时，可通过不同版本数字进行区分表示。编码的层次化约定和解析，由具体器件自行完成。|
|Type|91:88|协议定义的设备类型|
|Reserved|87:64|同一{VendorID, Device ID}设备的子版本编码。常见用法：设备在功能或者性能特性发生变更时，可通过不同版本数字进行区分表示。编码的层次化约定和解析，由具体器件自行完成。|
|Sequence Number|63:00|部署环境中UB Function Entity的唯一序列号。|

## 二、UB设备直通配置及使用说明

UB设备直通技术是一种基于硬件的虚拟化解决方案，通过该技术，虚拟机可以以直通的方式访问UB设备以获取最佳的性能。
UB Device实现有多个UB Entity，在Hypervisior的管理下，可以将UB Entity分配给不同的虚机使用，UB Etity在虚拟化应用中，有以下特点：

- UB Entity的数据面直接和虚机交互；管理面为了安全考虑，需要经由Hypervisor来进行控制。数据面包括：资源空间访问、中断、设备业务数据流。管理面主要指对设备UB Entity的管理，主要通过配置空间的访问来完成。

- 一个UB Entity也可以在某个虚机中卸载，经过一系列的安全处理后重新分配给其他虚机使用。

- 每个UB Entity互不影响，具备一定的安全隔离性(依赖设备厂商实现基于UB Entity粒度的设备资源隔离)。

### 2.1 直通约束限制

- 需先配置虚拟UB控制器和虚拟UMMU、iommufd

- 单VM最多支持16个UB直通设备

- HostOS、GuestOS需支持灵衢总线驱动，Host需使能UMMU

- 虚拟机必须使用大页

### 2.2 支持的直通设备类型

仅支持Hi1650 UDie内集成的iDev URMA设备

### 2.3 直通操作步骤

#### 步骤1：直通前准备工作

- 确认GuestOS、HostOS支持灵衢总线驱动，HostOS加载相关依赖驱动

- GuestOS内安装直通设备供应商提供的驱动

- 开启Host OS内核IOMMU和UMMU功能

#### 步骤2：获取待直通UB设备GUID

- 查询UB设备列表：`lsub`

- 查询设备GUID：`cat /sys/bus/ub/devices/[设备编号]/guid`

- 查询设备class code：`cat /sys/bus/ub/devices/[设备编号]/class_code`（筛选可直通设备）

- 记录目标设备GUID（如00007号设备：cc08-a002-0-2-000000-0000000000000001）

#### 步骤3：创建虚拟机BusInstance

```bash
# setub -b create -g [控制器businstance guid] -e [管控面分配eid] -u [管控面分配upi]
```

查询验证：`lsub -b` 或 `cat /sys/bus/ub/instance`

#### 步骤4：绑定设备与BusInstance

```bash
# setub -b bind -d [待直通设备GUID] -g [创建的BusInstance GUID]
```

查询验证：`cat /sys/bus/ub/devices/[设备编号]/instance`

#### 步骤5：绑定vfio-ub驱动

```bash
# 解绑现有驱动（若有）
echo [设备编号] > /sys/bus/ub/devices/[设备编号]/driver/unbind
# 绑定驱动
echo vfio-ub > /sys/bus/ub/devices/[设备编号]/driver_override
echo [设备编号] > /sys/bus/ub/drivers_probe
```

#### 步骤6：配置虚拟机并启动

- 将待直通设备GUID配置到虚拟机

- 启动虚拟机，通过`lsub`查询直通设备是否正常

### 2.4 直通设备XML配置

#### 完整xml配置

```xml
<hostdev mode='subsystem' type='ub' managed='yes' iommufd='1'>
    <driver name='vfio'/>
    <source>
        <address guid="cc08-a002-0-2-000000-0000000000000001"/>
    </source>
    <address type='ub' eid='0x2' guid="cc08-a002-0-2-000000-0000000000000001"/>
    <alias name='ua-hostdev0'/>
    <ports num='1'>
        <port index='0' teid='0x1' tport='0'/>
    </ports>
</hostdev>
```

#### 最简xml配置

```xml
<hostdev mode='subsystem' type='ub' iommufd='1'>
    <driver name='vfio'/>
    <source>
        <address guid="cc08-a002-0-2-000000-0000000000000001"/>
    </source>
</hostdev>
```

#### 直通配置关键参数说明

|参数名|说明|取值|是否必配|
|---|---|---|---|
|hostdev.mode|固定为subsystem|'subsystem'|是|
|hostdev.type|hostdev直通设备的总线类型|'ub'|是|
|hostdev.managed|libvirt处理UB设备驱动的两种模式，即是否由libvirt自动管理用户态vfio驱动，以及对虚拟机的BusInstance进行管理|'yes'或'no'；不配置时默认取值为'no'|否|
|hostdev.iommufd|配置使用的iommufd编号，编号从1开始，在iommufd配置数量范围内|'1'；建议配置1个iommufd，所有的vfio-ub设备必须使用相同的iommufd编号|是|
|hostdev.driver.name|name固定配置为vfio，表示使用vfio直通|'vfio'|是|
|hostdev.source.address.guid|HostOS上的UB设备的GUID|Host OS上的UB设备的GUID|是|
|hostdev.address|设备直通到GuestOS后的内部UB总线地址信息|type：配置地址类型是'ub'；eid： vUB设备的eid, qemu内部拓扑模拟使用，和虚机内ubus总线自动生成的eid无关；guid：vUB设备的guid；不配置时libvirt默认自动生成|否|
|hostdev.alias|配置vfio-ub直通设备的id|自定义时，需以'ua-'开头且不含'.'；不配置时libvirt自动生成|否|
|hostdev.ports|GuestOS视角的虚拟UB拓扑|hostdev.ports.num：vUB设备的port数量，配置范围[1, 256],默认为1；hostdev.ports.port.index: 端口编号，从0开始；hostdev.ports.port.teid: vUB设备连接的对端UB控制器的eid；hostdev.ports.port.tport: 对端UB控制器上的端口号；不配置时libvirt默认自动生成|否|

#### 直通注意事项

hostdev.managed='yes'时，步骤5可自动完成（目前暂不支持）

## 三、虚拟机UMMU配置及使用说明

UMMU(UB内存管理单元)是UB体系下实现多机互联访问的一个重要组件。在UB体系中，根据地址空间转换的方向将内存管理单元划分为UMMU和UB Decoder两块功能。
UMMU在系统中完成UB总线地址（UBA）到Home内部PA地址转换功能，此时系统总线地址通常为本端的物理地址。此外UMMU还完成访问本端地址的权限控制功能。
UMMU的整体功能是通过DstEID+TokenID+UBA这样一组输入来完成地址翻译和权限校验的功能。

### 3.1 UMMU约束限制

- 虚拟UMMU依赖iommufd

- 默认仅模拟1个UMMU，与UB控制器数量一致

- 使用UB直通设备时，虚拟UMMU为必选（不支持stage1 bypass）

- HostOS和GuestOS需支持灵衢总线驱动

### 3.2 UMMU配置方法

#### XML配置添加方式

```xml
<iommufds>1</iommufds>
<devices>
    <iommu model='ummu'>
    </iommu>
</devices>
```

### 3.3 UMMU关键参数说明

|参数名|说明|取值|是否必配|
|---|---|---|---|
|iommufds|配置支持的iommufd数量|iommufd的数量，对于UB设备直通取值'1'就够了，所有的UB直通设备可使用同一个iommufd；如果有除了UB直通设备之外的其他类型的直通设备也需要iommufd，则按需配置数量，最大配置不大于68|是|
|devices.iommu.model|虚拟iommu类型|'ummu'|是|
