# 安装jdk

- 第一步，卸载系统自带OpenJKD

```
# 查看目前系统的jdk
rpm -qa|grep jdk 
java-1.6.0-openjdk-1.6.0.35-1.13.7.1.el6_6.x86_64 
java-1.7.0-openjdk-1.7.0.79-2.5.5.4.el6.x86_64 
# 卸载系统自带OpenJKD
yum -y remove java-1.6.0-openjdk-1.6.0.35-1.13.7.1.el6_6.x86_64 
yum -y remove java-1.7.0-openjdk-1.7.0.79-2.5.5.4.el6.x86_64
# 本地上传解压安装包
tar -zxvf jdk1.8.0_131.tar.gz
# 配置环境变量
vim /etc/profile

export JAVA_HOME=/opt/jdk1.8.0_131
export PATH=$PATH:$JAVA_HOME/bin
export CLASSPATH=.:$CLASSPATH

# 重载环境变量
source /etc/profile

# 验证是否成功
java -version
```

# 关闭防火墙

```
# 查看防火墙状态
systemctl status firewalld.service 
# 关闭防火墙
systemctl stop firewalld.service 
# 设置永久关闭防火墙
systemctl disable firewalld.service  
```

# 配置ssh免登陆

```
# 生成秘钥
cd ~/.ssh/ & ssh-keygen -t rsa
# windows免登陆远程linux服务器
cat ./.ssh/id_rsa.pub | ssh root@base "umask 077; test -d .ssh || mkdir .ssh ; cat >> .ssh/authorized_keys || exit 1" 
# 配置秘钥
ssh-copy-id [target-hostname]
```

# 虚拟机克隆

- 复制快照文件
- 修改hostname

```
hostnamectl set-hostname [hostname]
```

- 修改网卡

```
# 修改配置文件
vim /etc/sysconfig/network-scripts/ifcfg-ens33

TYPE="Ethernet"
DEFROUTE="yes"
IPV4_FAILURE_FATAL="no"
IPV6INIT="yes"
IPV6_AUTOCONF="yes"
IPV6_DEFROUTE="yes"
IPV6_FAILURE_FATAL="no"
IPV6_ADDR_GEN_MODE="stable-privacy"
NAME="ens33"
DEVICE="ens33"
ONBOOT="yes"
PEERDNS="yes"
PEERROUTES="yes"
IPV6_PEERDNS="yes"
IPV6_PEERROUTES="yes"
IPV6_PRIVACY="no"
# 修改为静态方式
BOOTPROTO="static"
# 启动之后重新设置UUID
NETMASK="255.255.255.0"
GATEWAY="192.168.1.1"
DNS1=8.8.8.8
UUID="564d92e3-4fee-4319-db82-45e946f41c43"
IPADDR="192.168.1.21"

# 重载网卡
systemctl restart network
```

# 开机启动脚本

```
[root@zk02 ~]# vim /etc/rc.d/init.d/tomcat

#!/bin/bash
# chkconfig:2345 20 90
# description:tomcat
# processname:tomcat
export JAVA_HOME=/opt/jdk1.8.0_131
case $1 in
          start) su root /opt/apache-tomcat-8.5.4/bin/startup.sh;;
          stop) su root /opt/apache-tomcat-8.5.4/bin/shutdown.sh;;
          status) su root ps -ef|grep apache-tomcat-8.5.4;;
          *)  echo "require start|stop|status";;
esac

[root@zk02 ~]# chmod +x tomcat
[root@zk02 ~]# chkconfig --add tomcat
[root@zk02 ~]# service tomcat start
[root@zk02 ~]# service tomcat status
[root@zk02 ~]# service tomcat stop
```