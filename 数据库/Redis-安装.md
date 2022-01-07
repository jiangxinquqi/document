# 安装

```shell
# Reids是C语言编译的，所以需要c语言的编译环境，如果没有gcc需要在线安装。
[root@localhost  src]# yum install gcc-c++ 

# 下载安装包
[root@localhost  ~]# cd /opt
[root@localhost  src]# wget http://download.redis.io/releases/redis-3.0.0.tar.gz
[root@localhost  src]# tar -zxf redis-3.0.0.tar.gz 

# 安装与编译安装大多数其他软件不同的是，Redis 已经做过 ./configure，因此直接 make&& make install 即可：

[root@localhost  src]# cd redis-3.0.0 
[root@localhost  redis-3.0.0]# make 
[root@localhost  redis-3.0.0]# make install

# 安装可执行二进制文件到/usr/local/redis（默认为解压包下的src目录下）目录下，分别是
# （1）redis-benchmark —— 用于 Redis 性能测试
# （2）redis-check-aof —— 用于检查 aof 日志的工具
# （3）redis-check-dump —— 用于检查 rbd 日志的工具
# （4）redis-server —— Redis 服务器进程
# （5）redis-cli —— Redis 客户端进程
# （6）redis-sentinel —— 软链接文件，软链接到 redis-server
[root@localhost  redis-3.0.0]# mkdir /opt/redis
[root@localhost  redis-3.0.0]# make install PREFIX=/opt/redis 

# 复制配置文件
[root@xiaojianjun-redis  bin]# cp /opt/redis-3.0.0/redis.conf /opt/redis/bin

# 配置后台启动
[root@db01 bin]# vim /opt/redis/bin/redis.conf 
# 以配置文件启动
[root@db01 bin]# /opt/redis/redis-server /opt/redis/bin/redis.conf 
```

# 启动脚本

```shell
[root@db01 init.d]# vim /etc/rc.d/init.d/redis
[root@db01 init.d]# chmod +x redis
[root@db01 init.d]# chkconfig --add redis

#!/bin/bash
# chkconfig: 2345 90 10
# description: Redis is a persistent key-value database
PATH=/usr/local/bin:/sbin:/usr/bin:/bin   
REDISPORT=6379  
EXEC=/opt/redis/bin/redis-server   
REDIS_CLI=/opt/redis/bin/redis-cli
PIDFILE=/run/redis.pid  
CONF="/opt/redis/bin/redis.conf"  
AUTH="1234"  
case "$1" in   
        start)   
                if [ -f $PIDFILE ]   
                then   
                        echo "$PIDFILE exists, process is already running or crashed."  
                else  
                        echo "Starting Redis server..."  
                        $EXEC $CONF   &
                fi   
                if [ "$?"="0" ]   
                then   
                        echo "Redis is running..."  
                fi   
                ;;   
        stop)   
                if [ ! -f $PIDFILE ]   
                then   
                        echo "$PIDFILE exists, process is not running."  
                else  
                        PID=$(cat $PIDFILE)   
                        echo "Stopping..."  
                       $REDIS_CLI -p $REDISPORT  SHUTDOWN    
                        sleep 2  
                       while [ -x $PIDFILE ]   
                       do  
                                echo "Waiting for Redis to shutdown..."  
                               sleep 1  
                        done   
                        echo "Redis stopped"  
                fi   
                ;;   
        restart|force-reload)   
                ${0} stop   
                ${0} start   
                ;;   
        *)   
               echo "Usage: /etc/rc.d/init.d/redis {start|stop|restart|force-reload}" >&2  
                exit 1  
esac
```



# 