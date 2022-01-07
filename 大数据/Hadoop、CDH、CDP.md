# Hadoop

# CDH

**CDH：全称Cloudera’s Distribution Including Apache Hadoop。**



**Apache Hadoop 不足之处：**

```plain
版本管理混乱
部署过程繁琐、升级过程复杂
兼容性差
安全性低
```

**Cloudera's Distribution, including Apache Hadoop（CDH）：**

```plain
是Hadoop众多分支中的一种，由Cloudera维护，基于稳定版本的Apache Hadoop构建
提供了Hadoop的核心
可扩展存储
分布式计算
基于Web的用户界面
```

**CDH的优点：**

```plain
版本划分清晰
版本更新速度快
支持Kerberos安全认证
文档清晰
支持多种安装方式（Cloudera Manager方式）
```

**CDH简略架构：**

```plain
数据导入：sqoop
文件存储：HDFS
资源管理与调度：Yarn
数据计算：
	离线计算：MapReduce
  在线流计算：Spark
批处理：Hive
数据存储：Hbase
数据分析：Impala
机器学习：M/R
```

# CDP