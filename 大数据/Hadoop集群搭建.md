# 

![img](https://cdn.nlark.com/yuque/0/2021/jpeg/10386414/1628036049807-2e7b7886-30f6-42bf-b708-6e3bee71d757.jpeg)

# 环境准备

- JDK8
- 关闭防火墙

# Hadoop安装

```shell
第一步
# vim /usr/local/software/hadoop-2.4.1/etc/hadoop/hadoop-env.sh
export JAVA_HOME=/usr/local/software/jdk1.8.0_131

第二步
# vim /usr/local/software/hadoop-2.4.1/etc/hadoop/core-site.xml
    <configuration>
			<!-- 指定HDFS老大（namenode）的通信地址 -->
			<property>
					<name>fs.defaultFS</name>
					<value>hdfs://192.168.1.41:9000</value>
			</property>
			<!-- 指定hadoop运行时产生文件的存储路径 -->
			<property>
					<name>hadoop.tmp.dir</name>
					<value>/opt/hadoop-2.4.1/tmp</value>
			</property>
		</configuration>
 第三步
 # vim /usr/local/software/hadoop-2.4.1/etc/hadoop/hdfs-site.xml
   <configuration>
			<!-- 设置hdfs block副本数量 -->
			<property>
					<name>dfs.replication</name>
					<value>3</value>
			</property>
		</configuration>
 第四步
 # mv /usr/local/software/hadoop-2.4.1/etc/hadoop/mapred-site.xml.template /usr/local/software/hadoop-2.4.1/etc/hadoop/mapred-site.xml
 # vim /usr/local/software/hadoop-2.4.1/etc/hadoop/mapred-site.xml
    <configuration>
			<!-- 通知框架MR使用YARN -->
			<property>
					<name>mapreduce.framework.name</name>
					<value>yarn</value>
			</property>
		</configuration>
    
 第五步
 # vim /usr/local/software/hadoop-2.4.1/etc/hadoop/yarn-site.xml
   <configuration>
			<!-- reducer取数据的方式是mapreduce_shuffle -->
			<property>
				<name>yarn.nodemanager.aux-services</name>
				<value>mapreduce_shuffle</value>
			</property>
		</configuration>
 
 第六步 将hadoop添加到环境变量
 # vim /etc/profile
export HADOOP_HOME=/opt/hadoop-2.4.1
export PATH=$PATH:$JAVA_HOME/bin:$HADOOP_HOME/bin
# source /etc/profile

第七步 格式化HDFS（namenode）第一次使用时要格式化
# hadoop namenode -format

第八步 启动hadoop
	# /usr/local/software/hadoop-2.4.1/sbin/start-dfs.sh
  # /usr/local/software/hadoop-2.4.1/sbin/start-yarn.sh
  
第九步 验证是否启动成功
# jps
3094 NodeManager
2551 DataNode
12745 NameNode
12937 SecondaryNameNode
13226 Jps
2814 ResourceManager

http://192.168.1.41:50070（HDFS管理界面）
http://192.168.1.41:8088（MR管理界面）
```

# Hadoop集群

- 配置ssh免密登陆
- 虚拟机克隆

# Hadoop指令

## fs

```shell
# 创建目录 
hadoop fs -mkdir <hdfs路径>
# 查看所有目录-递归    
hadoop fs -lsr <hdfs路径>
# 查看文件 						 
hadoop fs -ls  <hdfs路径>
# 上传文件 
hadoop fs -put <本地file路径> <hdfs路径>
# 下载文件 
hadoop fs -get <hdfs路径>
```

## 管理

```shell
# 检查本地库 
hadoop checknative -a
# 查看文件的block信息 	
hadoop fsck <hdfs文件路径> -files -blocks
# 查看hdfs集群信息  
hdfs dfsadmin -report
```