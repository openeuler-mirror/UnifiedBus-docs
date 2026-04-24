# 部署指南

当前工具仅支持部署UB OS Component部分组件。

## 1. 硬件要求

本系统需部署在支持灵衢超节点上。

## 2.  OS版本要求

OS系统版本仅支持openEuler-24.03-LTS-SP3及以上版本

## 3.  安装软件包

### 3.1 ub-pkg-mem

ub-pkg-mem 提供 UB OS 的内存池管理功能

```bash
yum install -y ub-pkg-mem
```

检查ub-pkg-mem服务是否安装成功并启动

```bash
systemctl status ub-pkg-mem
```

### 3.2 ub-pkg-urma

ub-pkg-urma 提供 UB OS 的统一资源管理功能

```bash
yum install -y ub-pkg-urma
```

检查ub-pkg-urma服务是否安装成功并启动

```bash
systemctl status ub-pkg-urma
```

### 3.3 ub-pkg-virt

ub-pkg-virt 提供 UB OS 的设备虚拟化功能

```bash
yum install -y ub-pkg-virt
```

检查ub-pkg-virt服务是否安装成功并启动

```bash
systemctl status ub-pkg-virt
```

### 3.4 ub-pkg-manager

如果需要需要拥有UB OS 的设备虚拟化、统一资源管理、内存池管理等功能，请直接安装ub-pkg-manager

```bash
yum install -y ub-pkg-manager
```

检查ub-pkg-manager服务是否安装成功并启动

```bash
systemctl status ub-pkg-manager
```
