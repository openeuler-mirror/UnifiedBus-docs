# 虚拟机UB设备直通简介

UB设备直通技术是一种基于硬件的虚拟化解决方案，通过该技术，虚拟机可以以直通的方式访问UB设备以获取最佳的性能。

在UB设备虚拟化系统中，包含Host、UB Device组件，Host包含了UB Controller、UMMU、Hypervisor等软硬组件。

UB Device实现有多个UB Entity，在Hypervisor的管理下，可以将UB Entity分配给不同的虚机使用。UB Entity在虚拟化应用中，有以下特点：

- UB Entity的数据面直接和虚机交互；管理面为了安全考虑，需要经由Hypervisor来进行控制。
  - 数据面包括：资源空间访问、中断、设备业务数据流。
  - 管理面主要指对设备UB Entity的管理，主要通过配置空间的访问来完成。
- 一个UB Entity也可以在某个虚机中卸载，经过一系列的安全处理后重新分配给其他虚机使用。
- 每个UB Entity互不影响，具备一定的安全隔离性（依赖设备厂商实现基于UB Entity粒度的设备资源隔离）。
