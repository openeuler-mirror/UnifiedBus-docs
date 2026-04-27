# ub os component 安装

## ub-pkg-urma 安装

ub-pkg-urma 提供 UB OS 的通信功能，安装方法：

```bash
yum install -y ub-pkg-urma
```

检查ub-pkg-urma服务是否安装成功并启动

```bash
systemctl status ub-pkg-urma
```

## ub-pkg-mem 安装

ub-pkg-mem 提供 UB OS 的内存池化功能，安装方法：

```bash
yum install -y ub-pkg-mem
```

检查ub-pkg-mem服务是否安装成功并启动

```bash
systemctl status ub-pkg-mem
```

## ub-pkg-virt 安装

ub-pkg-virt 提供 UB OS 的虚拟化功能，安装方法：

```bash
yum install -y ub-pkg-virt
```

检查ub-pkg-virt服务是否安装成功并启动

```bash
systemctl status ub-pkg-virt
```

## ub-pkg-manager 安装

如果需要需要拥有UB OS 的通信、内存池化、虚拟化等全部功能，请直接安装ub-pkg-manager，安装方法：

```bash
yum install -y ub-pkg-manager
```

检查ub-pkg-manager服务是否安装成功并启动

```bash
systemctl status ub-pkg-manager
```
