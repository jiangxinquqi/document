# 简介

架构：HMaster-集群协调，负载均衡。

​    多个RegionServer: 负责读写表数据。最好和HDFS的datanode在一起，减少网络IO。



表结构：建表时只需要指定表名和列族

每一行是一条记录，每一行的字段数是任意的，每一个字段都要存储k-v

查询：指定：表名-行键-列族-字段名（列）

没有库的概念，只有表的概念。

插入数据后，hbase会对数据自动进行排序：

行之间按照行键进行字典排序。

列族内部，kv对之间按照key的字段排序。

# 环境准备

```bash
hadoop,zk
```

# 下载安装包，解压安装包

```bash
# 要把hadoop的hdfs-site.xml和core-site.xml 放到hbase/conf下
cp /opt/hadoop-2.4.1/etc/hadoop/{hdfs-site.xml,core-site.xml} /opt/hbase/conf
```

# 配置HBase

vim hbase-env.sh

```bash
export JAVA_HOME=/usr/java/jdk1.7.0_55
//告诉hbase使用外部的zk
export HBASE_MANAGES_ZK=false
```

vim hbase-site.xml

```xml
<configuration>
	  <!-- 指定hbase在HDFS上存储的路径 -->
    <property>
            <name>hbase.rootdir</name>
            <value>hdfs://192.168.1.41:9000/hbase</value>
    </property>
	  <!-- 指定hbase是分布式的 -->
    <property>
            <name>hbase.cluster.distributed</name>
            <value>true</value>
    </property>
	  <!-- 指定zk的地址，多个用“,”分割 -->
    <property>
            <name>hbase.zookeeper.quorum</name>
            <value>zk01:2181,zk02:2181,zk03:2181</value>
    </property>
    <!-- 指定web端口，如果是-1，禁止启动-->
    <property>
            <name>hbase.master.info.port</name>
            <value>60010</value>
   </property>
</configuration>
```

vim regionservers

```bash
hdp01
hdp02
hdp03
```

# 拷贝HBase到其他节点并同步时间

```bash
	scp -r /opt/hbase-0.96.2-hadoop2/ root@hdp01:/opt/
	scp -r /opt/hbase-0.96.2-hadoop2/ root@hdp02:/opt/
	scp -r /opt/hbase-0.96.2-hadoop2/ root@hdp03:/opt/
```

# 启动

```bash
start-hbase.sh
# 为保证集群的可靠性，要启动多个HMaster
hbase-daemon.sh start master

# 通过浏览器访问hbase管理页面
http://192.168.1.41:60010/master-status

#==================================================================== 
# hbase 停止regionserver
# 每个regionserver节点可以自由启动或停止，可以不随hbase整体一起。
# 停止后regionserver上的数据会被移到其他regionserver上，不影响hbase的使用。
#==================================================================== 

# 停止regionserver
/bin/hbase-daemon.sh stop regionserver RegionServer
 
# 启动regionserver
/bin/hbase-daemon.sh start regionserver RegionServer
 
# 重启regionserver
bin/graceful_stop.sh --restart --reload --debug nodename
```

# 简单语法

```sql
进入hbase命令行
./hbase shell

显示hbase中的表
list

创建user表，包含info、data两个列族
create 'user', 'info1', 'data1'
create 'user', {NAME => 'info', VERSIONS => '3'}

向user表中插入信息，row key为rk0001，列族info中添加name列标示符，值为zhangsan
put 'user', 'rk0001', 'info:name', 'zhangsan'

向user表中插入信息，row key为rk0001，列族info中添加gender列标示符，值为female
put 'user', 'rk0001', 'info:gender', 'female'

向user表中插入信息，row key为rk0001，列族info中添加age列标示符，值为20
put 'user', 'rk0001', 'info:age', 20

向user表中插入信息，row key为rk0001，列族data中添加pic列标示符，值为picture
put 'user', 'rk0001', 'data:pic', 'picture'

获取user表中row key为rk0001的所有信息
get 'user', 'rk0001'

获取user表中row key为rk0001，info列族的所有信息
get 'user', 'rk0001', 'info'

获取user表中row key为rk0001，info列族的name、age列标示符的信息
get 'user', 'rk0001', 'info:name', 'info:age'

获取user表中row key为rk0001，info、data列族的信息
get 'user', 'rk0001', 'info', 'data'
get 'user', 'rk0001', {COLUMN => ['info', 'data']}

get 'user', 'rk0001', {COLUMN => ['info:name', 'data:pic']}

获取user表中row key为rk0001，列族为info，版本号最新5个的信息
get 'user', 'rk0001', {COLUMN => 'info', VERSIONS => 2}
get 'user', 'rk0001', {COLUMN => 'info:name', VERSIONS => 5}
get 'user', 'rk0001', {COLUMN => 'info:name', VERSIONS => 5, TIMERANGE => [1392368783980, 1392380169184]}

获取user表中row key为rk0001，cell的值为zhangsan的信息
get 'people', 'rk0001', {FILTER => "ValueFilter(=, 'binary:图片')"}

获取user表中row key为rk0001，列标示符中含有a的信息
get 'people', 'rk0001', {FILTER => "(QualifierFilter(=,'substring:a'))"}

put 'user', 'rk0002', 'info:name', 'fanbingbing'
put 'user', 'rk0002', 'info:gender', 'female'
put 'user', 'rk0002', 'info:nationality', '中国'
get 'user', 'rk0002', {FILTER => "ValueFilter(=, 'binary:中国')"}


查询user表中的所有信息
scan 'user'

查询user表中列族为info的信息
scan 'user', {COLUMNS => 'info'}
scan 'user', {COLUMNS => 'info', RAW => true, VERSIONS => 5}
scan 'persion', {COLUMNS => 'info', RAW => true, VERSIONS => 3}
查询user表中列族为info和data的信息
scan 'user', {COLUMNS => ['info', 'data']}
scan 'user', {COLUMNS => ['info:name', 'data:pic']}


查询user表中列族为info、列标示符为name的信息
scan 'user', {COLUMNS => 'info:name'}

查询user表中列族为info、列标示符为name的信息,并且版本最新的5个
scan 'user', {COLUMNS => 'info:name', VERSIONS => 5}

查询user表中列族为info和data且列标示符中含有a字符的信息
scan 'user', {COLUMNS => ['info', 'data'], FILTER => "(QualifierFilter(=,'substring:a'))"}

查询user表中列族为info，rk范围是[rk0001, rk0003)的数据
scan 'people', {COLUMNS => 'info', STARTROW => 'rk0001', ENDROW => 'rk0003'}

查询user表中row key以rk字符开头的
scan 'user',{FILTER=>"PrefixFilter('rk')"}

查询user表中指定范围的数据
scan 'user', {TIMERANGE => [1392368783980, 1392380169184]}

删除数据
删除user表row key为rk0001，列标示符为info:name的数据
delete 'people', 'rk0001', 'info:name'
删除user表row key为rk0001，列标示符为info:name，timestamp为1392383705316的数据
delete 'user', 'rk0001', 'info:name', 1392383705316


清空user表中的数据
truncate 'people'


修改表结构
首先停用user表（新版本不用）
disable 'user'

添加两个列族f1和f2
alter 'people', NAME => 'f1'
alter 'user', NAME => 'f2'
启用表
enable 'user'


###disable 'user'(新版本不用)
删除一个列族：
alter 'user', NAME => 'f1', METHOD => 'delete' 或 alter 'user', 'delete' => 'f1'

添加列族f1同时删除列族f2
alter 'user', {NAME => 'f1'}, {NAME => 'f2', METHOD => 'delete'}

将user表的f1列族版本号改为5
alter 'people', NAME => 'info', VERSIONS => 5
启用表
enable 'user'


删除表
disable 'user'
drop 'user'


get 'person', 'rk0001', {FILTER => "ValueFilter(=, 'binary:中国')"}
get 'person', 'rk0001', {FILTER => "(QualifierFilter(=,'substring:a'))"}
scan 'person', {COLUMNS => 'info:name'}
scan 'person', {COLUMNS => ['info', 'data'], FILTER => "(QualifierFilter(=,'substring:a'))"}
scan 'person', {COLUMNS => 'info', STARTROW => 'rk0001', ENDROW => 'rk0003'}

scan 'person', {COLUMNS => 'info', STARTROW => '20140201', ENDROW => '20140301'}
scan 'person', {COLUMNS => 'info:name', TIMERANGE => [1395978233636, 1395987769587]}
delete 'person', 'rk0001', 'info:name'

alter 'person', NAME => 'ffff'
alter 'person', NAME => 'info', VERSIONS => 10


get 'user', 'rk0002', {COLUMN => ['info:name', 'data:pic']}
```