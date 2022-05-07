# 安装

## 一、解压

单机启动直接就可以bin/solr start

## 二、配置集群

```shell
# 1. 配置classpath
vim /etc/profile

export SOLR_HOME=/opt/solr-8.4.1
export PATH=$PATH:$JAVA_HOME/bin:$SOLR_HOME/bin

# 2. 重载配置
source /etc/profile

# 3. 配置solr启动信息
vim $SOLR_HOME/solr.in.sh

ZK_HOST="zk01.cloud.com:2181,zk02.cloud.com:2181,zk03.cloud.com:2181/solr"
ZK_CLIENT_TIMEOUT="15000"
SOLR_TIMEZONE="UTC+8"

# 4. 新建zk znode
solr zk mkroot /solr -z zk01.cloud.com:2181,zk02.cloud.com:2181,zk03.cloud.com:2181
solr zk ls /solr -z zk01.cloud.com:2181,zk02.cloud.com:2181,zk03.cloud.com:2181

# 5. 上传solr配置文件
solr zk upconfig -n default_01 -d /home/xiaojianjun/solrconfig/default_01  

# 6. 创建 collection
solr create -c default_01 -n default_01 -shards 2 -replicationFactor 1 -force 

# 7. 启动
solr start -p 8983 -cloud -force

# 查询solr集群运行状态
pdsh -R ssh -w cmsdn00[1-3] "source /etc/profile; /opt/solr-8.4.1/bin/solr status"
pdsh -R ssh -w cmsdn00[1-3] "source /etc/profile; /opt/solr-8.4.1/bin/solr stop -all"
pdsh -R ssh -w cmsdn00[1-3] "source /etc/profile; /opt/solr-8.4.1/bin/solr start -p 8984 -cloud -force"

# 下载solr配置
solr zk downconfig -n seele_index[配置名称] -d /home/xiaojianjun/seele_index[本地目录]
# 修改配置
# 上传solr配置
solr zk upconfig -n default_01 -d /home/xiaojianjun/solrconfig/default_01
# solr-admin：reload
```

## 三、配置开机启动

```shell
[root@zk02 ~]# vim /etc/rc.d/init.d/solr

#!/bin/bash
# chkconfig:2345 20 90
# description:solr
# processname:solr
export JAVA_HOME=/opt/jdk1.8.0_131
case $1 in
          start) su root /opt/solr-8.4.1/bin/solr start -c -force;;
          stop) su root /opt/solr-8.4.1/bin/solr -all stop;;
          status) su root /opt/solr-8.4.1/bin/solr status;;
          *)  echo "require start|stop|status";;
esac

[root@zk02 ~]# chmod +x solr
[root@zk02 ~]# chkconfig --add solr

[root@zk02 ~]# service solr start
[root@zk02 ~]# service solr status
[root@zk02 ~]# service solr stop
```

# IK分词器

```bash
# 1. 下载源码 https://github.com/magese/ik-analyzer-solr
# 2. 打包 ik-analyzer-solr-8.4.0.jar
# 3. 上传 ik-analyzer-solr-8.4.0.jar 到所有solr节点 $SOLR_HOME/server/solr-webapp/webapp/WEB-INF/lib
# 4. 上传词典到$SOLR_HOME/server/solr-webapp/webapp/WEB-INF/classes
	ik-analyzer-solr/src/main/resources/
		① IKAnalyzer.cfg.xml (IK默认的配置文件，用于配置自带的扩展词典及停用词典)
		② ext.dic (默认的扩展词典)
		③ stopword.dic (默认的停词词典)
# 5. ik.conf及dynamicdic.txt放入solr配置文件夹中，与solr的managed-schema文件同目录中。
	ik-analyzer-solr/src/main/resources/
  	ik.conf
    dynamicdic.txt
# 6. 配置Solr的managed-schema，添加ik分词器，示例如下；
<!-- ik分词器 -->
<fieldType name="text_ik" class="solr.TextField">
  <analyzer type="index">
      <tokenizer class="org.wltea.analyzer.lucene.IKTokenizerFactory" useSmart="false" conf="ik.conf"/>
      <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
  <analyzer type="query">
      <tokenizer class="org.wltea.analyzer.lucene.IKTokenizerFactory" useSmart="true" conf="ik.conf"/>
      <filter class="solr.LowerCaseFilterFactory"/>
  </analyzer>
</fieldType>
# 6. 重启solr
```

# 指令

## 基本指令

```shell
# 查看状态，包括zk，节点信息，健康状态
solr status
```

## zk

```bash
# 帮助文档
solr zk -help
# 从zk集群下载solr配置文件到本地
solr zk downconfig -n seele_index[配置名称] -d /home/xiaojianjun/seele_index[本地目录]
# 从本地上传配置文件到zk集群
solr zk upconfig -n default_01[配置名称] -d /home/xiaojianjun/solrconfig/default_01[本地目录]
```

## create

```bash
# 创建collection 
solr create -c default_01[collection名称] -n default_01[配置名称] -shards 2 -replicationFactor 1 -force
```

# 