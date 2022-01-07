# 安装

官网下载地址：https://zookeeper.apache.org/releases.html

```shell
$ 需要jdk环境
$ wget https://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.4.14/zookeeper-3.4.14.tar.gz
$ tar -zxvf zookeeper-3.4.14.tar.gz
$ cd zookeeper-3.4.14
$ cd conf/
$ cp zoo_sample.cfg zoo.cfg
$ cd ..
$ cd bin/
$ sh zkServer.sh start
```

# 设置开机启动

```shell
[root@zk02 ~]# vim /etc/rc.d/init.d/zk

#!/bin/bash
# chkconfig:2345 20 90
# description:zookeeper
# processname:zookeeper
export JAVA_HOME=/opt/jdk1.8.0_131
case $1 in
          start) su root /opt/zookeeper-3.4.6/bin/zkServer.sh start;;
          stop) su root /opt/zookeeper-3.4.6/bin/zkServer.sh stop;;
          status) su root /opt/zookeeper-3.4.6/bin/zkServer.sh status;;
          restart) su root /opt/zookeeper-3.4.6/bin/zkServer.sh restart;;
          *)  echo "require start|stop|status|restart"  ;;
esac

[root@zk02 ~]# chmod +x zk
[root@zk02 ~]# chkconfig --add zk

[root@zk02 ~]# service zk start
[root@zk02 ~]# service zk status
[root@zk02 ~]# service zk stop
[root@zk02 ~]# service zk restart
```

# 集群配置

```shell
# vim /opt/zookeeper-3.4.6/conf/zoo.cfg

server.1=192.168.1.21:2888:3888
server.2=192.168.1.22:2888:3888
server.3=192.168.1.23:2888:3888

# 根据 id 和对应的地址分别配置 myid
vim /tmp/zookeeper/myid
1
2
3

# 重启zookeeper
```