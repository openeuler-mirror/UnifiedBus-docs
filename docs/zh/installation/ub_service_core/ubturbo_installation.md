# ubturbo 安装指南

## 须知

- 需切换为root用户执行安装与启动。
- 初始安装由安装者保证环境纯净，如环境不纯净则会使用环境中的配置文件，可能会导致业务功能运行异常。
- ubturbo依赖smap，两者的安装部署顺序概述为：
    1. 创建ubturbo用户。
    2. 安装smap（包括插ko）。
    3. 安装ubturbo。
    4. 启动ubturbo，详细流程见后续章节。

## ubturbo 安装

### 创建ubturbo用户和组

使用如下指令在终端创建ubturbo用户和组：

```bash
SYSTEM_USER="ubturbo"
SYSTEM_GROUP="ubturbo"

# 一行初始化：先定义日志函数，再创建组和用户
(
  log_message() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1: $2"; }
  handle_error() { log_message "ERROR" "$1"; exit 1; }

  # 创建系统组（-r 表示系统组）
  if ! getent group "$SYSTEM_GROUP" > /dev/null; then
    groupadd -r "$SYSTEM_GROUP" || handle_error "Failed to create group $SYSTEM_GROUP"
    log_message "INFO" "Group $SYSTEM_GROUP created"
  else
    log_message "INFO" "Group $SYSTEM_GROUP already exists"
  fi

  # 创建系统用户（-r 系统用户，-g 指定组，-s 指定 shell）
  if ! getent passwd "$SYSTEM_USER" > /dev/null; then
    useradd -r -g "$SYSTEM_GROUP" -s /sbin/nologin "$SYSTEM_USER" || handle_error "Failed to create user $SYSTEM_USER"
    log_message "INFO" "User $SYSTEM_USER created"
  else
    log_message "INFO" "User $SYSTEM_USER already exists"
  fi
)
```

### 检查前置依赖SMAP

>说明
>
> - /dev/shm/smap_config保存了NUMA和进程配置等信息，如果UBTurbo进程需要切换用户，则需要先删除该文件。
> - /dev/shm/ubturbo_page_type.dat保存了SMAP的初始化类型信息，如果UBTurbo进程需要切换用户或者场景（例如虚拟化场景切换到大数据场景），则需要先删除该文件。
> - 该步骤中只安装smap的包，不插入smap驱动。

执行以下命令查询SMAP包是否安装。

```bash
rpm -qa | grep ubturbo-smap
```

若返回如下信息，表示安装成功。

```bash
[root@controller ~]# rpm -qa | grep ubturbo-smap
ubturbo-smap-*.aarch64
```

如未安装SMAP，参考[安装SMAP](#安装smap)。

### 安装SMAP

> 须知
> 
> 若环境中已安装SMAP，则可跳过本章节。

**前提条件**

* 安装前先创建ubturbo用户和用户组，配置免密登录，命令如下。
  > 说明
  >
  > 若已创建ubturbo用户和用户组，并已配置免密登录则无需重复操作。

  ```bash
  groupadd -r ubturbo
  useradd -r -g ubturbo -s /sbin/nologin ubturbo
  touch /etc/sudoers.d/ubturbo
  echo "ubturbo ALL=(root) NOPASSWD:/user/local/bin/cat.sh" > /etc/sudoers.d/ubturbo
  ```

* 当前部署环境需开启ACPI，并提前安装好obmm、URMA、libvirt库和numactl库。执行以下命令关闭numa\_balancing和透明大页。

  ```bash
  echo 0 > /proc/sys/kernel/numa_balancing
  echo 0 > /proc/sys/vm/compaction_proactiveness
  echo never > /sys/kernel/mm/transparent_hugepage/defrag
  echo never > /sys/kernel/mm/transparent_hugepage/enabled
  ```
  
* 安装libvirt，并配置，然后启动服务。容器场景略过此步骤。
  
  ```bash
  yum install -y qemu*
  yum install -y libvirt*

  # 进行libvirt配置
  systemctl start libvirtd
  ```
  
* 由于SMAP与HCOM对obmm均有依赖，若先安装smap再安装HCOM会导致HCOM出现异常，因此建议安装SMAP前先安装HCOM。
* /dev/shm/smap\_config保存了NUMA和进程配置等信息，使用SMAP动态库的进程如果需要切换用户，则需要先删除该文件。
* smap的period.config文件依赖ubturbo-rmrs组件，若环境上未安装ubturbo-rmrs，则需要手动创建目录/opt/ubturbo/conf，且确保目录权限和smap二进制文件权限一致。

**安装步骤**

1. 安装SMAP。
   
   安装smap软件包。
   
   ```bash
    rpm -ivh smap-x.x.x-x.oe2403sp1.aarch64.rpm
   ```

2. 载入扫描驱动。

    ```bash
    cd /lib/modules/smap
    insmod smap_tracking_core.ko
    ```
   
   * 安装smap_histogram_tracking.ko。

      ```bash
      insmod smap_histogram_tracking.ko
      ``` 

   * 安装smap_access_tracking.ko。在UB仿真中安装时，如使能SMAP硬件判热功能，则增加enable_hist=1参数。

      ```bash
      insmod smap_access_tracking.ko smap_scene=2
      ```

3. 检查扫描驱动是否载入成功。

    ```bash
    lsmod | grep tracking
    ```
   
   * 在环境能查询到插入的3个ko，则表示载入成功，示例如下：

      ```terminal
      [root@controller ~]# lsmod | grep tracking
      smap_access_tracking         65536  0
      smap_histogram_tracking      28672  1 access_tracking
      smap_tracking_core           28672  1 smap_access_tracking
      ```

4. 虚拟化场景需安装qemu-system-aarch64。
    
    ```bash
    yum install qemu-system-aarch64 -y
    ```
   
5. 进入以下目录，载入迁移驱动。
   
   ```bash
   cd /lib/modules/smap
   ```
   
   * 容器场景

      ```bash
      insmod smap_tiering.ko smap_pgsize=0
      ```

   * 虚拟化场景
   
      ```bash
      insmod smap_tiering.ko smap_scene=2
      ```
      
6. 检查迁移驱动是否载入成功，返回smap信息即表示安装成功。

    ```bash
      lsmod | grep smap
    ```

### 安装ubturbo

```bash
rpm -ivh ubturbo-rmrs-1.1.1-1.oe2203sp1.aarch64 --force
```

**目录结构：**

| 目录 | 用途说明 | 权限 | 所属用户组 |
| - | - | - | - |
| /opt/os\_turbo                                     | 程序根目录 | 750 | ubturbo:ubturbo |
| /opt/os\_turbo/bin                                 | 可执行文件目录 | 500 | ubturbo:ubturbo |
| /opt/os\_turbo/conf                                | 配置文件目录 | 700 | ubturbo:ubturbo |
| /opt/os\_turbo/lib                                 | 动态库目录 | 500 | ubturbo:ubturbo |
| /var/log/os\_turbo                                 | 日志目录 | 700 | ubturbo:ubturbo |

### 检查ubturbo是否安装成功

```bash
rpm -qa | grep ubturbo-rmrs
```

返回如下信息即表示安装成功：

```bash
[root@controller ~]# rpm -qa | grep ubturbo-rmrs
ubturbo-rmrs-*.aarch64
```

### 按实际需求修改配置文件

**说明：**

- UBTurbo默认不启用任何插件，请依据业务场景在ubturbo_plugin_admission.conf中打开插件（取消对应插件的注释）。
- 在ubturbo_plugin_admission.conf中打开插件前，需保证插件的动态库和配置文件已分别放置在/opt/ubturbo/lib和/opt/ubturbo/conf下，否则UBTurbo及对应插件将启动异常.

**ubturbo.conf**

| 序号 | 参数 | 说明 | 取值 | 配置节点 | 应用场景 |
| - | - | - | - | - | - |
| 1 | log.level | 日志等级 | 默认值：INFO<br>取值范围：DEBUG、INFO、WARN、ERROR、CRIT | 所有节点 | 决定主进程和插件的日志输出等级。 |

**os\_turbo\_plugin\_admission.conf** 

| 序号 | 参数 | 说明 | 取值 | 配置节点 | 应用场景 |
| - | - | - | - | - | - |
| 1 | rmrs | 内存碎片插件 | 默认值：777 | 所有节点 | 通算虚拟化场景 |

**说明：**

* 默认不启用任何插件，请依据业务场景在os\_turbo\_plugin\_admission.conf中打开插件（取消对应插件的注释）。
* 在os\_turbo\_plugin\_admission.conf中打开插件前，需保证对应的插件已执行其自身的安装步骤，否则UBTurbo及对应插件将启动异常。
* 插件准入配置文件os\_turbo\_plugin\_admission.conf的说明如下：
  * 只有在该准入配置中配置的插件才会被加载并初始化，未在该文件中配置的插件将不予加载。
  * 配置中key对应插件的插件名称(需要与插件自身配置文件中的名称相对应)，value为初始化函数中需要使用的moduleCode。
  * moduleCode是唯一值。
* 进程启动成功后，通过执行以下命令来查询哪些插件已加载成功:
  
  ```bash
  cat /var/log/ubturbo/ubturbo.log | grep "loaded successfully"
  ```

## 启动ubturbo服务

### 启动服务

```bash
systemctl start ubturbo
```

### 查询服务状态

- 查询服务当前状态，状态为active即表示服务已经启动：
  
  ```bash
  systemctl status ubturbo
  ```

- 查询服务对应进程，观察进程是否存在：
  
  ```bash
  ps -ef | grep ubturbo
  ```

### 查看启动日志

观察进程是否正常启动。启动成功的标志：TurboMain::Run end.

```bash
journalctl -u ubturbo
```
