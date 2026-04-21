# ubs-virt

## 介绍

`Ubs-virt`支持虚拟化和池化、实时迁移策略决策、快速恢复和灾难恢复，以及虚拟机和容器之间的快速通信，从而显著提高虚拟化性能。

## 涉及组件

1. ubs-virt-ovs: ubs-virt提供网络的能力的服务，支持配置指定urma设备的带宽。
2. ubs-virt-enpu: ubs-virt提供NPU算力切分的服务，支持配置指定的算力和显存资源。
3. virt-awaresched: virt-awaresched提供对虚拟化调度调优的服务，减少vCPU无意义迁移的性能开销，提升虚机的线性度。

## 使用说明

1. ubs-virt-ovs: [ubs-virt-ovs使用说明](../../installation/ub_service_core/ubs-virt-ovs_installation.md)
2. ubs-virt-enpu: [ubs-virt-enpu使用说明](../../installation/ub_service_core/vCANN-RT_installation.md)
3. virt-awaresched：[virt-awaresched使用说明](../../installation/ub_service_core/virt-awaresched_installation.md)
