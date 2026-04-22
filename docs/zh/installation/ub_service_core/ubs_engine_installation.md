# UBS-Engine(UBSE) 安装指南

## 构建软件包

### 获取源码

```shell
git clone https://atomgit.com/openeuler/ubs-engine.git
```

### 构建依赖

ubs-engine构建依赖信息记录在spec文件（[ubs-engine.spec](https://atomgit.com/openeuler/ubs-engine/blob/master/ubs-engine.spec)）中。
由于当前依赖的ubs-comm-devel版本只发布在openEuler 24.03 LTS SP3/SP4，通过`cat /etc/openEuler-release`确认编译系统版本：

- 如果编译系统版本是`openEuler release 24.03 (LTS-SP3)/(LTS-SP4)`，参考[openEuler release 24.03 (LTS-SP3)/(LTS-SP4) 依赖安装说明](#openeuler-release-2403-lts-sp3lts-sp4-依赖安装说明)
- 如果编译系统版本不是`openEuler release 24.03 (LTS-SP3)/(LTS-SP4)`，参考[其他openEuler版本依赖安装说明](#其他openeuler版本依赖安装说明)

#### openEuler release 24.03 (LTS-SP3)/(LTS-SP4) 依赖安装说明

- 方式一：DNF一键安装

  ```shell
  # 安装spec中指定的BuildRequires
  cd ubs_engine
  dnf builddep -y ubs-engine.spec
  ```

- 方式二：yum手动安装

  ```shell
  yum install -y libboundscheck libxml2-devel numactl-libs openssl-devel rapidjson-devel patch 'cpp-httplib-devel >= 0.27.0' 'ubs-comm-devel >= 1.0.0-15'
  ```

#### 其他openEuler版本依赖安装说明

执行以下命令安装依赖：

```shell
yum install -y libboundscheck libxml2-devel numactl-libs openssl-devel rapidjson-devel patch 'cpp-httplib-devel >= 0.27.0'
```

源码编译ubs-comm-devel和ubs-comm-lib：

```shell
# 安装基础工具链和 rpm 构建工具
sudo dnf install -y git rpm-build rpmdevtools gcc gcc-c++ make
mkdir -p /tmp/src && cd /tmp/src
# 下载 ubs-comm 源码
git clone -b openEuler-24.03-LTS-SP3 https://gitcode.com/src-openeuler/ubs-comm.git
cd ubs-comm
# 初始化 rpmbuild 目录结构
rpmdev-setuptree
# 复制 ubs-comm 源码中的 spec 文件到 rpmbuild 目录
cp hcom.spec ~/rpmbuild/SPECS/
cp -rf ubs-comm.tar.gz ~/rpmbuild/SOURCES/
# 安装 hcom.spec 中指定的 BuildRequires
sudo dnf builddep -y ~/rpmbuild/SPECS/hcom.spec
# 构建 hcom 包
rpmbuild -ba ~/rpmbuild/SPECS/hcom.spec
# 安装构建得到的 rpm 包
# 架构为aarch64
sudo dnf install -y ~/rpmbuild/RPMS/aarch64/ubs-comm-devel-*.rpm
sudo dnf install -y ~/rpmbuild/RPMS/aarch64/ubs-comm-lib-*.rpm
# 架构为x86_64
sudo dnf install -y ~/rpmbuild/RPMS/x86_64/ubs-comm-devel-*.rpm
sudo dnf install -y ~/rpmbuild/RPMS/x86_64/ubs-comm-lib-*.rpm
```

### 执行构建

```shell
# 执行 Release 构建（没有调试信息，-O2 优化）
bash build.sh

# 执行 Debug 构建（附加调试信息）
bash build.sh -D

# 执行 RelWithDebInfo 构建（附加调试信息，-O2 优化）
bash build.sh -T RelWithDebInfo

# 执行 MinSizeRel 构建（没有调试信息，二进制文件最小构建，-Os 优化）
bash build.sh -T MinSizeRel
```

## 形成rpm包

```shell
# 构建项目，并打包成 rpm 文件输出到项目顶层目录的 output/ 下。
bash build.sh package
```

产物如下：

```path
└── output                                                            # 打包输出文件目录
    ├── ubs-engine-<version>-<release>.aarch64.rpm                    # 主程序包
    ├── ubs-engine-client-libs-<version>-<release>.aarch64.rpm        # 客户端库
    ├── ubs-engine-client-devel-<version>-<release>.aarch64.rpm       # 客户端开发包
    ├── python3-ubs-engine-<version>-<release>.aarch64.rpm            # python API 包
    └── ubs-engine-debuginfo-<version>-<release>.aarch64.rpm          # 调试信息
```

## 软件包准备

> [!NOTE]说明
>
> 从构建产物中选择安装部署所需的软件包。

ubs engine的发布件包含：开发包、运行包

ubs engine的部署分为开发包部署和运行包部署

- 开发包：主要用于支持开发者使用ubs engine提供的接口，进行业务开发

- 运行包：主要用于在生产环境中与周边ubfm等模块交互，提供ub池化资源管理能力

ubs engine的发布件，由以下四个 RPM 包组成：

| RPM包                                                       | 说明                                |
| ----------------------------------------------------------- | ---------------------------------- |
| ubs-engine-\<version\>-\<release\>.aarch64.rpm              | 主程序包，包含服务、CLI、配置等       |
| ubs-engine-client-libs-\<version\>-\<release\>.aarch64.rpm  | 客户端运行时库（供第三方程序动态链接） |
| ubs-engine-client-devel-\<version\>-\<release\>.aarch64.rpm | 开发包（含头文件与静态库）            |
| python3-ubs-engine-\<version\>-\<release\>.aarch64.rpm      | python API 接口 |

## 安装开发包

### 环境要求

业务开发环境中部署即可，无特殊要求

### 安装命令

- 方式一

  ```bash
  # 安装开发包（用于集成开发）
  # 注：需要系统配置了openEuler release 24.03 (LTS-SP3)镜像源
  sudo dnf install -y ubs-engine-client-devel
  # 安装python 模块（可选，使用UBSE Python API时需要安装）
  sudo dnf install -y python3-ubs-engine
  ```

- 方式二

  ```bash
  # 安装开发包（用于集成开发）
  # 通过rpm包安装开发包
  sudo dnf install -y ubs-engine-client-devel-<version>-<release>.aarch64.rpm
  # 安装python 模块（可选，使用UBSE Python API时需要安装）
  sudo dnf install -y python3-ubs-engine-<version>-<release>.aarch64.rpm
  ```

### 安装结果

**ubs-engine 开发包安装结果：**

| 文件/目录                      | 其它说明                                     |
| ------------------------------ | -------------------------------------------- |
| `/usr/include/ubse`            | 目录下头文件（`*.h`）权限：`644`             |
| `/usr/lib64/libubse-client.so` | 软链接，指向`/usr/lib64/libubse-client.so.1` |
| `/usr/lib64/libubse-client.a`  | 二进制静态。库                                 |

**ubs-engine Python API 包安装结果：**

| 文件/目录                      | 其它说明                                     |
| ------------------------------ | -------------------------------------------- |
| `/usr/lib64/python3.11/site-packages/ubse` | 内部文件（`*.py`）权限：`644`             |
| `/usr/lib/python3.11/site-packages/ubse-1.0.0-py3.11.egg-info` | 内部文件权限：`644`，Python包相关信息             |

**安装注意事项**

  开发包需要依赖客户端包，即ubs-engine-client-devel依赖ubs-engine-client-libs，安装ubs-engine-client-devel前需要安装ubs-engine-client-libs。

## 安装运行包

### 系统要求

* <b>操作系统：</b> openEuler 24.03 LTS 或更高版本
* <b>CPU架构：</b> aarch64
* <b>内存：</b> ≥ 64GB
* <b>磁盘：</b> SSD，IOPS 500MB/s
* <b>芯片互联：</b> UB
* <b>网卡：</b>可选依赖（可选使用TCP辅助UB建链，默认采用UB自举建链）
* <b>用户权限：</b> 安装与管理需 <code>root</code> 权限

### 节点规划

- 所有集群节点均需安装 <code>ubs-engine</code> 主包。

- 建议提前规划节点 IP 地址列表（用于 TCP 模式）。

### 依赖运行时库

ubs-engine运行依赖信息记录在spec文件（[ubs-engine.spec](https://atomgit.com/openeuler/ubs-engine/blob/master/ubs-engine.spec)）中。
运行依赖所需系统库，通常由包管理器自动安装。

**安装注意事项**

  - 非高安场景下，UBSE与UBM使用UDS通信，需要将ubse用户加到ubm_nuds用户组中，该用户组由UBM服务创建，如果UBM服务未在UBSE前安装，ubse用户可能无法加入ubm_nuds用户组，导致UBSE服务与UBM服务通信异常。
  - UBSE需要调用ubturbo接口，ubturbo接口有权限校验，需要将ubse用户加到ubturbo用户组中，该用户组由ubturbo服务创建，如果ubturbo服务未安装，ubse用户可能无法加入ubturbo用户组，导致UBSE服务调用ubturbo接口异常。

### 包安装步骤

- 方式一

  ```bash
  # 注：需要系统配置了openEuler release 24.03 (LTS-SP3)镜像源
  # 安装主程序包
  sudo dnf install -y ubs-engine
  # 安装客户端运行时库（第三方集成必需）
  sudo dnf install -y ubs-engine-client-libs
  # 安装python 模块（可选，使用UBSE Python API时需要安装）
  sudo dnf install -y python3-ubs-engine
  ```

- 方式二

  ```bash
  # 通过rpm包安装运行包
  # 安装主程序包
  sudo dnf install -y ubs-engine-<version>-<release>.aarch64.rpm
  # 安装客户端运行时库（第三方集成必需）
  sudo dnf install -y ubs-engine-client-libs-<version>-<release>.aarch64.rpm
  # 安装python 模块（可选，使用UBSE Python API时需要安装）
  sudo dnf install -y python3-ubs-engine-<version>-<release>.aarch64.rpm
  ```

### 安装结果

- ubs-engine 主程序安装结果：

  | 路径                                 | 用途          |
  | ------------------------------------ | -------------|
  | /usr/bin/ubse, /usr/bin/ubsectl      | 主程序与 CLI  |
  | /usr/lib/systemd/system/ubse.service | systemd 服务  |
  | /etc/ubse/                           | 配置目录      |
  | /var/log/ubse/                       | 日志目录      |
  | /var/lib/ubse/                       | 持久化数据    |
  | /var/lib/ubse/cert/                  | 证书目录      |
  | /var/lib/ubse/lcne_cert/             | 高安部署证书目录|
  | /var/run/ubse/                       | 运行时 socket |

- ubs-engine 客户端运行库安装结果：

  | 文件                                 | 其它说明                                          |
  | ------------------------------------ | ------------------------------------------------- |
  | `/usr/lib64/libubse-client.so.1.0.0` | 二进制动态库实体                                  |
  | `/usr/lib64/libubse-client.so.1`     | 软链接，指向 `/usr/lib64/libubse-client.so.1.0.0` |

- ubs-engine Python API 包安装结果：

  | 文件/目录                      | 其它说明                                     |
  | ------------------------------ | -------------------------------------------- |
  | `/usr/lib64/python3.11/site-packages/ubse` | 内部文件（`*.py`）权限：`644`             |
  | `/usr/lib/python3.11/site-packages/ubse-1.0.0-py3.11.egg-info` | 内部文件权限：`644`，Python包相关信息。             |

## 通信模式选择：URMA vs TCP

ubs engine支持两种通信模式，可根据硬件和网络环境选择：

| 通信方式   | URMA(默认)             | TCP                    |
| ---------- | ---------------------- | ---------------------- |
| 性能       | 高性能、低延迟、零拷贝 | 标准 TCP，性能适中     |
| 硬件要求   | 需支持 URMA 的智能网卡 | 普通以太网卡即可       |
| 配置复杂度 | 免配置，自动发现节点   | 需手动配置 IP 列表     |
| 使用场景   | 高性能计算、金融交易   | 普通数据中心、开发测试 |

### URMA通信模式（推荐）

ubs engine采用urma通信方式，有如下特点：

* 安装后直接启动服务，无需任何配置。

* 节点通过 URMA 自动发现并建立连接。

* 适用于具备 URMA 硬件支持的环境。

#### 安装URMA

确保已安装 URMA 驱动和运行时库，否则服务将无法启动或通信失败

```python
# 示例：安装 URMA 运行时包（具体包名根据发行版可能不同）
sudo yum install -y umdk-urma-lib    # OpenEuler
sudo yum install -y umdk-urma-kmod
```

#### 启动ubs engine服务

```bash
sudo systemctl start ubse
sudo systemctl enable ubse
```

### TCP通信模式

#### 修改配置

1. 编辑配置文件：

    ```bash
    sudo vi /etc/ubse/ubse.conf
    ```

2. 修改以下配置项(默认无此配置项,打开此配置时，使用tcp通信，否则默认使用urma通信)：

    ```ini
    [ubse.rpc]
    cluster.ipList=192.168.100.100-192.168.100.102
    ```

    > 支持配置IP地址范围：192.168.100.100-192.168.100.102

#### 重启ubs engine服务

```bash
sudo systemctl restart ubse
```

## ubs engine服务管理

### 启动服务

```bash
sudo systemctl start ubse
```

### 停止服务

```bash
sudo systemctl stop ubse
```

### 查看状态

```bash
sudo systemctl status ubse
```

### 开机自启

```bash
sudo systemctl enable ubse
```

## 验证部署

### 检查服务状态

```bash
systemctl is-active ubse  # 应输出 "active"
```

### 查看日志

- 方式一

  ```bash
  journalctl -u ubse -f

  ```

- 方式二

  ```bash
  /var/log/ubse/xxx
  ```

### 使用CLI工具

```bash
sudo /usr/bin/ubsectl --help
```

## 升级与卸载

### 升级

```bash
# 通过rpm包进行升级
sudo dnf update -y *ubs-engine*.rpm
# 配置openEuler release 24.03 (LTS-SP3)镜像源进行升级
sudo dnf update -y ubs-engine ubs-engine-client-libs ubs-engine-client-devel python3-ubs-engine
```

### 卸载

```bash
sudo dnf remove -y ubs-engine ubs-engine-client-libs ubs-engine-client-devel python3-ubs-engine
```

> 卸载不会删除 <code>/var/lib/ubse/data</code> 中的数据，如需清理请手动删除。

## 常见问题

### Q1: 服务启动失败，提示权限错误？

* 检查 <code>/var/log/ubse</code> 和 <code>/var/lib/ubse</code> 所属用户是否为 <code>ubse:ubse</code>。

* 使用 <code>chown -R ubse:ubse /var/lib/ubse /var/log/ubse</code> 修复。

### Q2: CLI 命令补全无效？

* 安装 <code>bash-completion</code>：<code>sudo dnf install bash-completion</code>

* 重新登录或执行：<code>source /etc/bash_completion.d/cli_commands.sh</code>

### Q3: 如何查看当前通信模式？

- 使用命令快速查看配置：

  ```bash
  grep -E "cluster.ipList" /etc/ubse/ubse.conf
  ```

  输出示例:

  ```text
  # cluster.ipList=192.168.100.100-192.168.100.102,192.168.100.104
  ```

* 若显示 cluster.ipList 行首有 # 注释符 , 当前使用 URMA 模式
* 若显示 cluster.ipList 行首无 # 注释符, 当前使用 TCP 模式
