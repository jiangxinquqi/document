# Spark内核

## 集群管理

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1626567780584-1ce6f39d-6297-465d-bfe1-b7e6ab616132.png)

## 

| Application     | 基于Spark的用户程序，包含了driver程序和集群上的executor      |
| --------------- | ------------------------------------------------------------ |
| Driver Program  | 运⾏main函数并且新建SparkContext的程序                       |
| Cluster Manager | 在集群上获取资源的外部服务(例如:standalone,Mesos,Yarn )      |
| Worker Node     | 集群中任何可以运⾏应⽤代码的节点                             |
| Executor        | 是在⼀个worker node上为某应⽤启动的⼀个进程，该进程负责运⾏任务，并且负责将数据存在内存或者磁盘上。每个应⽤都有各⾃独⽴的executors |
| Task            | 被送到某个executor上的⼯作单元                               |
| Job             | 包含很多任务的并⾏计算，可以看做和Spark的action对应          |
| Stage           | ⼀个Job会被拆分很多组任务，每组任务被称为Stage(就像Mapreduce分map任务和reduce任务⼀样) |

## 核心组件

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1626567818507-be305ad4-100a-4363-9e80-f495860aae0e.png)

## RDD图

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1626567854247-ea014b42-b17e-40a3-9de0-047020d7923f.png)

# Spark安装

## 环境准备

```plain
jdk
hdfs
scala
```

## Scala安装

```plain
和安装jdk一样
https://www.scala-lang.org/download/scala2.html
# 下载，解压，配置classpath
```

## Spark集群安装

```plain
# 1. 下载安装包，并且解压
https://archive.apache.org/dist/spark/

# 2. 配置主节点
cp spark-env.sh.template spark-env.sh

export SCALA_HOME=/opt/scala-2.11.12
export JAVA_HOME=/opt/jdk1.8.0_131
export HADOOP_HOME=/opt/hadoop-2.4.1
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop

SPARK_MASTER_HOST=192.168.1.41
SPARK_LOCAL_DIRS=/opt/spark-2.0.1-bin-hadoop2.4
SPARK_DRIVER_MEMORY=500M

# 3. 配置从节点
cp slaves.template slaves

192.168.1.42
192.168.1.43

# 4. 复制安装包到其他节点

# 5. 启动集群
sbin/start-all.sh

# 6. 访问管理界面
http://192.168.1.41:8080/

# 进入bin目录scale开发模式
./spark-shell

scala> val lines = sc.textFile("/home/helloSpark")
lines: org.apache.spark.rdd.RDD[String] = /home/helloSpark MapPartitionsRDD[1] at textFile at <console>:24

scala> lines.count()
res0: Long = 3

scala> lines.first()
res1: String = hello world!


# 运行指令
/opt/spark-2.0.1-bin-hadoop2.4/bin/spark-submit \
--master spark://192.168.1.41:7077 \
--class com.xiao.spark.WorkCount \
--executor-memory 512m \
--total-executor-cores 2 \
/opt/spark-2.0.1-bin-hadoop2.4/jars/easy-springboot-spark-1.0-SNAPSHOT.jar \
192.168.1.41 \
9000 \
GoneWiththeWind.txt
```



# QA

- Spark目前⽀持哪⼏种语⾔的API？

python，java，scala

- RDD执⾏transformation和执⾏action的区别是什么？
- Spark为什么计算快？

基于内存，DAG-记录依赖，快速并行计算恢复，容错。 

- RDD cache默认的StorageLevel级别是什么？ 
- 说明narrow dependency 和 wide dependency的区别？ 从计算和容错两⽅⾯说明！