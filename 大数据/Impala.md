# 1.1 什么是Impala



Cloudera公司推出，提供对HDFS、HBase数据的高性能、低延迟的交互式SQL查询功能。

基于Hive，使用内存计算，兼顾数据仓库、具有实时、批处理、多并发等优点。

是CDH平台首选的PB级大数据实时查询分析引擎。



# 1.2 Impala的优缺点

## 优点

1. 基于内存运算，不需要把中间结果写入磁盘，省掉了大量的I/O开销。
2. 无需转换为Mapreduce，直接访问存储在HDFS，HBase中的数据进行作业调度，速度快。

1. 使用了支持Data locality的I/O调度机制，尽可能地将数据和计算分配在同一台机器上进行，减少了网络开销。
2. 支持各种文件格式，如TEXTFILE 、SEQUENCEFILE 、RCFile、Parquet。

1. 可以访问hive的metastore，对hive数据直接做数据分析。



## 缺点

1. 对内存的依赖大，且完全依赖于hive。
2. 实践中，分区超过1万，性能严重下降。

1. 只能读取文本文件，而不能直接读取自定义二进制文件。
2. 每当新的记录/文件被添加到HDFS中的数据目录时，该表需要被刷新。

# 1.3 Impala的架构



![img](https://cdn.nlark.com/yuque/0/2021/webp/10386414/1634196833699-0d90d64d-cf93-43ef-b304-c57b6000b890.webp)

Impala自身包含三个模块：Impalad、Statestore(存放Hive的元数据)和Catalog(拉取真实数据)，除此之外它还依赖Hive Metastore和HDFS。



## Impalad



接受Client的请求，Query执行并返回给中心协调节点；  

子节点上的守护进程， 负责向Statestore保持通信，汇报工作。

## Catalog  



分发表的元数据信息到各个Impalad中；

接收来自Statestore的所有请求。

# Statestore



负责收集分布在集群中各个Impalad进程的资源信息、各节点健康状况，同步节点信息；  