# UBS-Engine(UBSE)

## 介绍

`UBSE`(UB Service Core Engine，简称UBSE或UBS Engine)提供了ubse daemon程序及其相应的SDK开发库。开发者可以利用该SDK访问ubse daemon提供的服务，从而实现对MEM（内存）等资源的调度与管理，执行关键运维操作。

## 目录结构

```shell
UBSEngine/
├── 3rdparty                    //第三方软件
├── conf                        //配置文件
├── doc                         //文档
├── scripts                     //脚本
├── src                         
│   ├── include                 //全局头文件
│   ├── apiserver               //北向接口暴露
│   ├── cli                     //cli代码
│   │   ├──ubse_cli_framework   //命令注册和结果回显辅助builder类
|   │   └──ubse_cert            //证书相关
│   │   
│   ├── controllers             //控制器
│   │   ├── include             //头文件  
│   │   ├── mem                 //内存池化控制器
│   │   |    ├── algorithm      //mem调度算法
│   │   │    ├── memcontroller  //内存池化控制器---文件
│   │   │    └── memscheduler   //内存池化调度器---文件
│   │   └── node                //内存池化控制器---节点的采集信息
│   │        └── nodecontroller //内存池化控制器---文件
│   │  
│   ├── framwork                //软件框架
│   │   ├── com                 //通信组件，与hcom对接，线程切换，回调
│   │   ├── config              //配置模块
│   │   ├── context             //上下文模块
│   │   ├── event               //事件中心
│   │   ├── ha                  //ha模块
│   │   ├── http                //http组件
│   │   ├── ipc                 //ipc
│   │   ├── log                 //日志组件
│   │   ├── misc                //杂项：智能指针、锁、环形队列、CRC
│   │   ├── plugin_mgr          //插件管理
│   │   ├── security            //安全组件，开源只需提权，商用版本支持使用KMC加解密
│   │   ├── serde               //序列化反序列化
│   │   ├── thread_pool         //线程池操作库，fram自身系统线程池
│   │   ├── timer               //定时器
│   │   └── xml                 //xml解析处理
│   │   
│   ├── message                 //消息
│   ├── node                    //创建数据链路
│   ├── ras                     //故障处理
│   ├── adapter_plugins
│   │   ├── syssentry           //syssentry对接
│   │   ├── mti                 //UBM对接
│   │   └── mmi                 //内存资源接口
│   │   
│   └── sdk                     //sdk模块
│       ├── include             //sdk的对外接口声明
│       └── example              //sdk示例
└── test
    ├── IT
    ├── PT
    └── UT
```

阅读各个目录下的`README.md`获取更详细的说明。

## 架构

UBSE 架构详细介绍请查看：[Architecture](https://gitcode.com/hellsark/ubs-engine/blob/master/docs/design/architecture.md)。

## 功能

提供了ubse daemon程序及其相应的SDK开发库。开发者可以利用该SDK访问ubse daemon提供的服务，从而实现对MEM（内存）等资源的调度与管理，执行关键运维操作。UBSE 功能介绍详见api介绍。

## 安装指导

UBS Engine：[UBS Engine安装手册](../../installation/ub_service_core/ubs_engine_installation.md)
