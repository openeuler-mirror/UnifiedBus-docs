# UB OS Component 安装

## 部署 UB 通信

### 关闭 numa balancing

单次生效方法：

``` bash
echo 0 > /proc/sys/kernel/numa_balancing
```

持久化生效方法：

``` bash
echo "kernel.numa_balancing = 0" > /etc/sysctl.d/99-numa-balancing.conf
```

### 配置 ipourma 参数

需要根据组网来配置`ipourma`的参数，新建`/etc/modprobe.d/ub.conf`文件，输入以下内容并保存。

如果是电互联版本：

``` bash
options ipourma tx_ring_size=16 rx_ring_size=32 page_level=16 ctp_sl=6
```

如果是光互联版本：

``` bash
options ipourma tx_ring_size=16 rx_ring_size=32 page_level=16 ctp_sl=4
```

### 部署 ub-pkg-urma

ub-pkg-urma 提供 UB OS 的通信功能，安装方法：

```bash
yum install -y ub-pkg-urma
```

检查 ub-pkg-urma 服务是否安装成功并启动

```bash
systemctl status ub-pkg-urma
```

## 部署池化内存

### 部署 ub-pkg-mem

ub-pkg-mem 提供 UB OS 的内存池化功能，安装方法：

```bash
yum install -y ub-pkg-mem
```

检查 ub-pkg-mem 服务是否安装成功并启动

```bash
systemctl status ub-pkg-mem
```

### 配置内核启动参数

内核启动参数配置，**需要重启生效**，在`/boot/efi/EFI/openEuler/grub.cfg`文件中`menuentry`的`linux`行中配置如下字段：

``` bash
pmd_mapping=100% numa_remote=nofallback,hugetlb_nowatermark,preonline crash_kexec_post_notifiers
```

参数说明如表所示：

| 参数        | 取值                         | 说明                                                         |
| ----------- | ---------------------------- | ------------------------------------------------------------ |
| pmd_mapping | 格式：nn%；取值范围：1%-100% | 允许从特定的 pfn 范围分配连续内存，该范围的内存线性映射区粒度始终不大于 PMD。pmd_mapping 指定每个 NUMA 节点的该范围所占比例。OBMM 使用 buddy_highmem分配器时，最多从每个 NUMA 节点分配 pmd_mapping 比例的内存。 |
| numa_remote | 格式：字符串                 | 将未使用的 NUMA 节点配置为远程节点，当启用 CONFIG_NUMA_REMOTE 配置项时，允许在这些节点上热插拔远程内存。默认所有未使用的 NUMA 节点均会被标记为远程节点，可通过启动参数 numa_remote_max_nodes 限制远程节点数量。preonline：允许将未就绪内存上线并保持隔离状态，以提升上线效率。nofallback：远程节点不会出现在其他节点的内存区域列表中，远程内存仅能通过显式指定节点进行分配。hugetlb_nowatermark：在远程节点分配大页内存时忽略水位线检查，允许将所有内存作为大页分配。\<int\>：设置远程 NUMA 节点的最大数量（整数限制）。 |

## 部署灵衢虚拟化

### ub-pkg-virt 安装

ub-pkg-virt 提供 UB OS 的虚拟化功能，安装方法：

```bash
yum install -y ub-pkg-virt
```

检查 ub-pkg-virt 服务是否安装成功并启动

```bash
systemctl status ub-pkg-virt
```

## 通用部署方法

如果需要UB OS 的通信、内存池化、虚拟化等全部功能，请直接安装ub-pkg-manager，该包会自动安装ub-pkg-urma、ub-pkg-mem、ub-pkg-virt，安装方法：

```bash
yum install -y ub-pkg-manager
```

检查 ub-pkg-manager 服务是否安装成功并启动

```bash
systemctl status ub-pkg-manager
```
