# 一、基础组件



1. ResourceManager（RM）



负责对各个NodeManager 上的资源进行统一管理和调度。包含两个组件：



- Scheduler：调度器根据容量、队列等限制条件（如每个队列分配一定的资源，最多执行一定数量的作业等），将系统中的资源分配给各个正在运行的应用程序
- Applications Manager：应用程序管理器负责管理整个系统中所有应用程序，包括应用程序提交、与调度器协商资源以启动ApplicationMaster、监控ApplicationMaster运行状态并在失败时重新启动它等



1. NodeManager（NM）



NM 是每个节点上的资源和任务管理器。



- 定时地向RM 汇报本节点上的资源使用情况和各个Container 的运行状态
- 接收并处理来自AM 的Container启动/ 停止等各种请求



1. ApplicationMaster（AM）



用户提交的每个应用程序均包含一个AM，主要功能包括：



- 与RM 调度器协商以获取资源（用 Container 表示）
- 将得到的任务进一步分配给内部的任务

- 与 NM 通信以启动 / 停止任务
- 监控所有任务运行状态，并在任务运行失败时重新为任务申请资源以重启任务



1. Container



Container 是YARN 中的资源抽象， 它封装了某个节点上的多维度资源， 如内存、CPU、磁盘、网络等，当AM 向RM 申请资源时，RM 为AM 返回的资源便是用Container表示的。

# 二、运行机制

## 2.1 运行时图

1. 用户向YARN中提交应用程序，包括AM、AM 启停指令脚本、用户程序。
2. ResourceManager为该应用程序分配第一个Container，并与对应的NodeManager通信，要求它在这个Container中启动应用程序的AM。

1. AM首先向ResourceManager注册，以便ResourceManager查看应用程序的运行状态，为各个任务申请资源，监控运行状态，直到运行结束。

1. 1. AM采用轮询的方式通过RPC协议向ResourceManager申请和领取资源。
   2. AM申请到资源后，与对应的NodeManager通信，要求它启动任务。

1. 1. NodeManager为任务设置好运行环境（环境变量、JAR包、二进制程序），将任务启动命令写到一个脚本中，通过该脚本启动任务。
   2. 各个任务通过RPC协议向AM汇报自己的状态和进度，以便让AM随时掌握各个任务的运行状态，从而可以在任务失败时重启任务。

1. 应用程序运行完成后，AM向ResourceManager注销并关闭自己。 

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1632626179832-61da62ec-8f41-41e7-9065-50498c09e763.png)

## 2.2 内部机制

1.  YARN的核心服务由两类长期运行的进程提供：

1. 1. Resource Manager 管理资源
   2. Node Managere 运行在集群所有节点上的启动和监控容器的，容器是用于执行特定的应用的程序的进程，每个容器有资源限制，详细可以配置（如内存、CPU等）。



1.  YARN启动  

 	客户端向资源管理器发送一个请求，要求其运行一个Application Master，资源管理器（Resource Manager）便在集群容器中找到一个能运行Application Master的节点管理器，Application Master启动后将要做什么由其自己决定，可向资源管理器请求更多的容器或在所处的容器中进行一个简单的计算并将结果返回给客户端（而各个进程或节点间的通信是由hadoop RPC提供的，YARN本身不提供通信机制，但是YARN有API，可一般不调用，基本用更高级的如MapReduce的API）。



1. AM向YARN申请资源

YARN的资源请求遵从优先最近原则，既本节点为首要选择，其次为本机架，如再不满足可取本集群任意节点。YARN的资源请求可在任意时刻提出，既动态申请。理想情况下，YARN的资源申请应立即给予满足，但由于现实情况下资源有限所以一个应用请求资源经常需要进行等待，这就涉及到YARN的资源调度方式，调度方式的区别决定了资源请求的速度。



1. YARN资源调度

 	资源调度的方式通常有三种：FIFO调度器，容器调度器、公平调度器  