# 架构初探

1. UI界面
2. sql解析引擎 - mysql数据库存储元数据

1. MR模板引擎
2. job提交引擎

# 配置Hive

```sql
1.修改配置文件 cp hive-default.xml.template hive-site.xml 
	修改hive-site.xml（删除所有内容，只留一个<property></property>）
	添加如下内容：
  
  <property>
    <name>hadoop.proxyuser.root.hosts</name>   #配置成*的意义，表示任意节点使用 hadoop 集群的代理用户 root 都能访问 hdfs 集群
    <value>*</value>
	</property>
	<property>
    <name>hadoop.proxyuser.root.groups</name> #表示代理用户的组所属
    <value>*</value>
	</property>
  
	<property>
	  <name>javax.jdo.option.ConnectionURL</name>
	  <value>jdbc:mysql://db01:3306/hive?createDatabaseIfNotExist=true</value>
	  <description>JDBC connect string for a JDBC metastore</description>
	</property>
	<property>
	  <name>javax.jdo.option.ConnectionDriverName</name>
	  <value>com.mysql.jdbc.Driver</value>
	  <description>Driver class name for a JDBC metastore</description>
	</property>
	<property>
	  <name>javax.jdo.option.ConnectionUserName</name>
	  <value>root</value>
	  <description>username to use against metastore database</description>
	</property>
	<property>
	  <name>javax.jdo.option.ConnectionPassword</name>
	  <value>xiaojianjun</value>
	  <description>password to use against metastore database</description>
	</property>
 
 2. 安装hive和mysq完成后，将mysql的连接jar包拷贝到$HIVE_HOME/lib目录下
 	如果出现没有权限的问题，在mysql授权(在安装mysql的机器上执行)
	mysql -uroot -p
	#(执行下面的语句  *.*:所有库下的所有表   %：任何IP地址或主机都可以连接)
	GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY '123' WITH GRANT OPTION;
	FLUSH PRIVILEGES;
```

# beeline连接

```sql
hive --service hiveserver2 &
beeline -u jdbc:hive2://192.168.1.41:10000
```

# 建表

```sql
在hive当中创建两张表
create table trade_detail (id bigint, account string, income double, expenses double, time string) row format delimited fields terminated by '\t';
create table user_info (id bigint, account string, name  string, age int) row format delimited fields terminated by '\t';

将mysq当中的数据直接导入到hive当中
sqoop import --connect jdbc:mysql://192.168.1.10:3306/itcast --username root --password 123 --table trade_detail --hive-import --hive-overwrite --hive-table trade_detail --fields-terminated-by '\t'
sqoop import --connect jdbc:mysql://192.168.1.10:3306/itcast --username root --password 123 --table user_info --hive-import --hive-overwrite --hive-table user_info --fields-terminated-by '\t'

创建一个result表保存前一个sql执行的结果
create table result row format delimited fields terminated by '\t' as select t2.account, t2.name, t1.income, t1.expenses, t1.surplus from user_info t2 join (select account, sum(income) as income, sum(expenses) as expenses, sum(income-expenses) as surplus from trade_detail group by account) t1 on (t1.account = t2.account);

create table user (id int, name string) row format delimited fields terminated by '\t'
将本地文件系统上的数据导入到HIVE当中
load data local inpath '/root/user.txt' into table user;

创建外部表
create external table stubak (id int, name string) row format delimited fields terminated by '\t' location '/stubak';
== 内部表与外部表的不同
==== 创建外部表需要添加 external 字段。而内部表不需要。
==== 删除外部表时，HDFS中的数据文件不会一起被删除。而删除内部表时，表数据及HDFS中的数据文件都会被删除。

创建分区表
普通表和分区表区别：有大量数据增加的需要建分区表
create table book (id bigint, name string) partitioned by (pubdate string) row format delimited fields terminated by '\t';

分区表加载数据
load data local inpath './book.txt' overwrite into table book partition (pubdate='2010-08-22');

-- 建表
create table t_user(id int,name string,age int) 
row format delimited
fields terminated by ",";

-- 创建数据文件
vim user.data

1,xiaoxiao,11
2,haah,22
3,fdsfd,33

-- 上传数据文件到hdfs 
方式一：
hadoop fs -put user.data /user/hive/warehouse/xiaojianjun.db/t_user
方式二:
load data local inpath /home/user.data into table t_user;

-- 查询hive库(类sql)
select * from t_user;

create database myhive; # 建库
show databases; # 查看某个数据库
use 数据库;      # 进入某个数据库
show tables;    # 展示所有表
desc 表名;            # 显示表结构
show partitions 表名; # 显示表名的分区
show create table table_name;   # 显示创建表的结构
```

# 