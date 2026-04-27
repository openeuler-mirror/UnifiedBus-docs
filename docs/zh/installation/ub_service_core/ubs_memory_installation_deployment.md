# 安装部署

## 约束限制

- 如果不做特殊说明，均默认为ARM64架构。
- Linux通用内存映射均是Cacheable属性映射内存，访存时内存会经过Cache。
- 内核线性映射区支持BLOCK粒度（如PUD/PMD）和PAGE粒度的地址映射。使用BLOCK粒度有助于减少TLB miss，提升性能。但BLOCK与PAGE之间的映射粒度在系统启动后不可动态切换。若需修改BLOCK中某个单页（如4KB）的属性，理论上需将整个BLOCK拆分为PAGE粒度，并按照ARM规范中的BBM（Break-Before-Make）规则进行更新。然而，目前ARM内核的线性映射区不支持运行时对BLOCK的动态拆分，因此该操作在启动后无法实现。
- Linux内存管理采用4KB粒度的基础页（openEuler默认版本为4KB，但ARM64支持16KB、64KB基础页），后续所有内存管理均假设为4KB基础页模式。

- Linux通用内存管理器有以下两种：

    - 伙伴系统：最大分配连续物理页大小为4MB。
    - 静态大页（HugeTLB），虚机化场景为了性能，会采用静态预留方式，在启动时预留大部分内存放到HugeTLB管理器中，通常采用1GB/2MB粒度。

- 对于伙伴系统分配大页2MB，如果有空闲的连续2MB内存，则直接分配，如果不存在，则需要进行内存规整（小页合成大页），但因为内存碎片化问题（1个2MB地址空间中有部分页面被分配，且无法迁移），会导致即使有空闲内存，也无法分配2MB页，即分配2MB成功率难以保证。

- Linux内存热插拔是系统级别行为，会影响其他子系统，上下线内存速度在重载下耗时久，可能会导致失败。

    - 需要获取热插拔全局锁，上线与上线、上线与下线，下线与下线等均需要串行操作。
    - 每个内存上下线粒度至少为128MB。
    - 通过热插拔notify事件通知其他子系统做相应变动。
    - 上线新内存也需要分配管理元数据占用本地内存。

- Linux内核改动需要遵循Linux规范，不能破坏已有接口语义（常见规则如Documentation/filesystems/proc.rst，Documentation/ABI/等）。

## 安装前准备

### 硬件环境

安装前，需要检查以下硬件配置，如[表1](#table001)所示。

**表 1 <a id="table001"></a>**  硬件环境

| 类型 | 配置参考|
|-----|-----|
| 服务器 | <ul><li>TaiShan 500 2280</li> <li>其他配备支持UB的CPU的服务器</li></ul> |

### 软件环境

在安装UBS Memory之前，需要准备以下软件环境：

#### 安装系统依赖

```shell
# 安装基础依赖包
yum install -y spdlog openssl-libs libboundscheck
```

#### 安装UBS核心组件

**方法一：通过yum源安装（推荐）**

```shell
# 安装UBS通信库
yum install -y ubs-comm-lib ubs-comm-devel

# 安装UBS引擎
yum install -y ubs-engine ubs-engine-client-libs ubs-engine-client-devel
```

**方法二：手动安装RPM包**

```shell
# 安装UBS通信库
rpm -ivh ubs-comm-lib-*.rpm
rpm -ivh ubs-comm-devel-*.rpm

# 安装UBS引擎
rpm -ivh ubs-engine-1.*.rpm
rpm -ivh ubs-engine-client-libs-1.*.rpm
rpm -ivh ubs-engine-client-devel-1.*.rpm
```

### 准备软件包

| 安装包名称 | 说明 |
|--|--|
| ubs-mem-kshmem-*x.x.x-x.x*.aarch64.rpm | UBS Memory安装包。 |

## 安装UBS Memory

### 前提条件

- 已完成[软件环境](#软件环境)章节所示的各项依赖的安装。
- 已获取UBS Memory安装包。

### 操作步骤

1. 使用\{UBSM-install-user\}用户登录服务器。
2. 将获取的所有软件包上传到任意目录，并进入该目录。
3. 卸载、安装固件。

    1. （可选）卸载已存在的UBS Memory、HCOM。

        ```bash
        rpm -e ubs-mem-kshmem
        rpm -e ubs-comm-lib-x.x.x
        rpm -e ubs-comm-devel-x.x.x
        ```

    2. 安装HCOM、UBS Memory。

        ```bash
        rpm -ivh ubs-comm-lib-x.x.x-*rpm
        rpm -ivh ubs-comm-devel-x.x.x-*rpm
        rpm -ivh ubs-mem-kshmem-x.x.x-x.x.*.rpm
        ```

    >[!NOTE]说明
    >- 由于UBS Engine依赖HCOM，卸载HCOM之前需先卸载UBS Engine。
    >- 内存服务以ubsmd用户的身份运行，在使用RPM包安装时，若系统中不存在ubsmd用户，安装脚本将自动创建该用户。
    >- 安装成功后，so会默认安装到“/usr/local/ubs\_mem/lib”目录，使用时需要export该路径。
    >- 安装成功后，.h头文件会默认安装到“/usr/local/ubs\_mem/include”目录。
    >- 配置环境变量 **UBSM\_SDK\_TRACE\_ENABLE = 1**，开启性能打点统计，会在默认的日志路径（/var/log/ubsm）生成对应的打点数据。
    >- 配置环境变量 **MXM\_CHANNEL\_TIMEOUT= xx**，控制IPC通信的channel超时时间（单位s），当大块内存操作耗时较久时，可以配置较长时间，默认为60s。

4. 启动UBS Engine服务。

    ```bash
    systemctl start ubse.service
    ```

5. 修改ubsmd.conf配置文件。
    1. 打开“/usr/local/ubs\_mem/config/ubsmd.conf”配置文件。

        ```bash
        vim /usr/local/ubs_mem/config/ubsmd.conf
        ```

    2. 按“i”进入编辑模式，根据实际情况对相关参数进行配置，参数详情请参见附录的[表1 ubsmd.conf配置文件参数说明](https://gitcode.com/openeuler/ubs-mem/blob/master/docs/zh/appendixes.md#%E9%85%8D%E7%BD%AE%E5%8F%82%E6%95%B0%E8%AF%B4%E6%98%8E)。

        ```yaml
        # the log level of ubsm server, (DEBUG, INFO, WARN, ERROR, CRITICAL)
        ubsm.server.log.level = INFO
        # the log file path, must be canonical path
        ubsm.server.log.path = /var/log/ubsm
        # log file count, min is 1, max is 50
        ubsm.server.log.rotation.file.count = 10
        # log file size(MB), min is 2, max is 100
        ubsm.server.log.rotation.file.size = 20
        # enable or disable audit log, (on, off)
        ubsm.server.audit.enable = on
        # audit log, the configuration item value range is the same as 'ubsm.server.log.*'
        ubsm.server.audit.log.path = /var/log/ubsm
        ubsm.server.audit.log.rotation.file.count = 10
        ubsm.server.audit.log.rotation.file.size = 20
        # CC lock
        ubsm.lock.enable = off
        ubsm.lock.expire.time = 300
        ubsm.lock.dev.name = bonding_dev_0
        ubsm.lock.dev.eid = 0
        # CC lock tls options
        ubsm.lock.tls.enable = on
        ubsm.lock.tls.ca.path = /path/cacert.pem
        ubsm.lock.tls.crl.path = /path/crl.pem
        ubsm.lock.tls.cert.path = /path/cert.pem
        ubsm.lock.tls.key.path = /path/key.pem
        ubsm.lock.tls.keypass.path = /path/keypass.txt
        # Zen discovery
        # election timeout, min is 0, max is 2000
        ubsm.discovery.election.timeout = 1000
        # min nodes, min is 0, max is 30
        ubsm.discovery.min.nodes = 0
        ubsm.server.rpc.local.ipseg = 127.0.0.1:7201
        ubsm.server.rpc.remote.ipseg = 127.0.0.1:7301
        # tls options
        ubsm.server.tls.enable = on
        ubsm.server.tls.ciphersuits = aes_gcm_128
        ubsm.server.tls.ca.path = /path/cacert.pem
        ubsm.server.tls.crl.path = /path/crl.pem
        ubsm.server.tls.cert.path = /path/cert.pem
        ubsm.server.tls.key.path = /path/key.pem
        ubsm.server.tls.keypass.path = /path/keypass.txt
        # max is 8192, default 256
        ubsm.hcom.max.connect.num = 256
        # enable or disable memory lease cache, (on, off)
        ubsm.server.lease.cache.enable = off
        # enable performance statistics, (on off)
        ubsm.performance.statistics.enable = off
        ```

        >[!NOTE]说明
        >
        >- 使用共享内存的分布式锁功能时，需要在配置文件中主动设置当前节点的IP地址和端口号以及集群中其他节点的节点信息，启动当前节点的ubsmd进程，会同步启动其他节点。
        >- 开启TLS（Transport Layer Security，安全传输层协议）认证功能操作详情可参见[开启TLS认证](https://gitcode.com/openeuler/ubs-mem/blob/master/docs/zh/security_management_and_hardening.md#%E5%BC%80%E5%90%AFtls%E8%AE%A4%E8%AF%81)，如果不使用该功能，将配置项 `ubsm.server.tls.enable` 设为 `off` 即可。

    3. 按“Esc”键，输入**:wq!**，按“Enter”保存并退出编辑。

6. 启动ubsmd。

    ```bash
    systemctl start ubsmd
    ```

    >[!NOTE]说明
    >ubsmd进程启动依赖UBSE，该服务启动成功方可加载成功。

7. 查看ubsmd状态。

    ```bash
    systemctl status ubsmd
    ```

    回显如下表明启动成功：

    ```bash
    ● ubsmd.service - UBS memory daemon
         Loaded: loaded (/etc/systemd/system/ubsmd.service; enabled; preset: disabled)
         Active: active (running) since xx YYYY-mm-dd HH:MM:SS CST; xxs ago
       Main PID: xxx (ubsmd)
         Status: "available"
          Tasks: 31 (limit: 822900)
       FD Store: 1 (limit: 3)
         Memory: xxG ()
         CGroup: /system.slice/ubsmd.service
                 └─xxx /usr/local/ubs_mem/bin/ubsmd -binpath=/usr/local/ubs_mem
    ```

8. 停止ubsmd。

    >[!NOTE]说明
    >- 内存借用提供类似 **malloc/free** 接口，当应用侧异常退出时，由ubsmd完成资源回收操作。
    >- 内存共享提供类似 **shmem\_allocate/shmem\_deallocate** 接口，应用侧需要跨多节点共享访问，由应用侧负责共享内存的申请释放操作。

    ```bash
    systemctl stop ubsmd.service
    ```

## 卸载UBS Memory

### 操作步骤

1. 使用\{UBSM-install-user\}用户登录服务器。
2. 卸载UBS Memory。

    >[!CAUTION]注意
    >
    >- 卸载会自动停止ubsmd并释放占用的远端内存，因此在卸载ubsmd时，应确保UBSE正常运行且无业务正在进行。
    >- 为了避免权限问题，卸载后用户和用户组ubsmd将会保留。
    >- 如需卸载UBS Engine，请参见[UBS Engine 部署说明](https://atomgit.com/openeuler/ubs-engine/blob/master/docs/build_install/%E9%83%A8%E7%BD%B2%E8%AF%B4%E6%98%8E.md)文档。

    ```bash
    rpm -e ubs-mem-kshmem
    ```
