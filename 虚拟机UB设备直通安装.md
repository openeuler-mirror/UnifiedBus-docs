# 虚拟机UB设备直通安装说明
## 1. 功能说明
虚拟机 UB 设备直通特性完整实现在系统配套 QEMU 与 libvirt 中，在 openEuler 操作系统上需通过标准流程安装 QEMU 和 libvirt 组件后，即可正常使用 UB 设备直通功能，无需额外安装其他独立组件。
## 2. 环境准备
- 操作系统：openEuler 24.03 LTS SP3
- 权限：使用 root 权限的账号操作
## 3. 虚拟化组件安装
UB 设备直通依赖系统配套 QEMU 和 libvirt，按照官方标准流程安装虚拟化平台即可：
1. 安装虚拟化相关组件包
2. 启动 libvirtd 服务
详细安装步骤、命令及配置项请参考官方虚拟化安装文档：https://docs.openeuler.org/zh/docs/24.03_LTS_SP3/virtualization/virtulization_platform/virtulization/virtualization_installation.html
