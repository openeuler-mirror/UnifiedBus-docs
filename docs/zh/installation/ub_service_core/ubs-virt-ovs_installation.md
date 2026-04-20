# ubs-virt-ovs 安装指南

## 获取源码

执行以下命令获取`ubs-virt-ovs`源代码。

- 方法一

```shell
$ git clone <ubs-virt-ovs-repo-url>
$ git submodule update --init --recursive
```

- 方法二

```shell
$ git clone <ubs-virt-ovs-repo-url> --recurse-submodules
```

## 源码构建

`ubs-virt-ovs` 源码目录结构如下：

```shell
.
├── build     // Stores script files used for compilation and building in the project.
├── sdk       // Stores the SDK code that is exposed to the outside world.
├── src       // This directory contains the source code for the project's functionalities; only this directory participates in the build process.
└── test      // Ut and dtfuzz, etc., are used to store projects.
```

## 软件包构建

`ubs-virt-ovs` 在代码仓库中提供了构建脚本（即`build.sh`），默认情况下无需进行任何配置修改，可直接执行该脚本文件进行编译和构建。

编译后的输出文件位于 `build/output` 目录中。

```shell
$ ./build/build.sh
[INFO] Build type: RelwithDebinfo
[INFO] Project root: /tmp/xxx
[INFO] Spec file: /tmp/xxx/ubs-virt-ovs/build/ubs-virt-ovs.spec
[INFO] Source tar created: /tmp/xxx/ubs-virt-ovs/build/rpmbuild/SOURCES/ubs-virt-ovs-1.0.0.tar.gz
...
[INFO] Outpud RPMs:
total 716K
-rw-------. 1 root root  50K ubs-virt-ovs-1.0.0-1.rel.aarch64.rpm
-rw-------. 1 root root 640K ubs-virt-ovs-debuginfo-1.0.0-1.rel.aarch64.rpm
-rw-------. 1 root root  23K ubs-virt-ovs-debugsource-1.0.0-1.rel.aarch64.rpm
```

## 安装部署

将编译后的输出文件上传到环境中，并使用RPM命令进行安装。

```shell
$ rpm -ivh ubs-virt-ovs-*
```
