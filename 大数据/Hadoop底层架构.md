# 核心组件

- hdfs-数据存储
- yarn-资源调度，调用计算资源

- MapReduce-数据计算

# hdfs实现机制![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1615478493975-4a1684f9-2175-443c-9510-b747151d1cf9.png)![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1615478794425-148e58b5-a0ee-4bb9-ad1c-8a4b59f6d5c7.png)

## NameNode

- 决定怎么存储block![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1616896590922-348b6d62-ee68-467e-b281-9216af99021f.png)



## MetaData![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1616896527787-a309741f-cdcb-444b-a136-0583f08996d6.png)



- - 什么时候checkpoint

- - - fs.checkpoint.period指定两次check的时间间隔，默认3600s
    - fs.checkpoint.size规定edits文件的大小限制，默认64M

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1616896886166-f1d09510-d7cb-43f0-a174-d483e6575c02.png)

## DataNode

- 定期向NameNode汇报自身所存储的block信息。

### block

- 最基本的存储单位，早起 1.X 是默认64M， 2.x 默认大小为128M，由dfs.block.size参数决定。
- HDFS文件系统中，如果一个文件的大小小于一个block的大小，并不会占用整个block存储空间，所以，HDFS不适用于大量的小文件的处理（这样子会浪费大量的元数据存储空间）。

- 副本存放策略，block多副本，默认三个。（最后的结果是2/3的副本存放在同一个机架上，1/3存放在另一个机架上，机架：可以理解一组datanode组成的集群，不同的机架之间通过交换机通信。）

- - 先在客户端所连接的datanode存放一个副本。
  - 再在另一个机架上挑选一个datanode存放另一个副本。

- - 最后在本机架上根据负载随机挑选一个datanode存放另一个副本。

- 副本数量指定的优先级

- - 服务端hdfs-site.xml 中可以配置replication
  - 客户端可以指定dfs.replication（客户端指定的优先级最高，可以理解为JVM参数值优先值最高）



# 

# MpaReduce

# ![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1615479036365-a402b961-c8c0-45a1-b183-371621dc9e45.png)

- 移动运算，而不是移动数据。
- map阶段：并行的，互不干扰的本地计算。

- reduce阶段：汇总计算map阶段产生的结果。