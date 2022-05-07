# 环境准备

## 说明

| IP            | Hostname        | 角色           | 内存 | CPU  | 磁盘 |
| ------------- | --------------- | -------------- | ---- | ---- | ---- |
| 192.168.1.101 | cdp01.cloud.com | CM, 时间服务器 | 8G   | 2    | 100G |
| 192.168.1.102 | cdp02.cloud.com | 主节点         | 6G   | 2    | 100G |
| 192.168.1.103 | cdp03.cloud.com | 从节点         | 4G   | 2    | 100G |
| 192.168.1.104 | cdp04.cloud.com | 从节点         | 4g   | 2    | 100G |

## 配置host

```plain
cat >> /etc/hosts <<EOF
192.168.1.101 cdp01.cloud.com cdp01	
192.168.1.102 cdp02.cloud.com	cdp02
192.168.1.103 cdp03.cloud.com cdp03
192.168.1.104 cdp04.cloud.com cdp04
EOF
```

## 关闭 Selinux

```plain
setenforce 0
sed -i 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config
```

## 安装 ntp 服务

```plain
yum install -y ntp ntpdate

# 手动同步时间-省略
ssh cdp01 "ntpdate -u ntp.api.bz"
ssh cdp02 "ntpdate -u ntp.api.bz"
ssh cdp03 "ntpdate -u ntp.api.bz"
ssh cdp04 "ntpdate -u ntp.api.bz"

vim /etc/ntp.conf

# 01节点指向127.127.1.0
server 127.127.1.0
Fudge 127.127.1.0 stratum 10
# 其他节点指向01节点
server 192.168.1.101
Fudge 192.168.1.101 stratum 10

systemctl enable ntpd.service
systemctl start ntpd.service
systemctl status ntpd.service

# 查看时间同步
ntpq -p

remote：本机和上层ntp的ip或主机名，“+”表示优先，“*”表示次优先
refid：参考上一层ntp主机地址
st：stratum阶层
when：多少秒前曾经同步过时间
poll：下次更新在多少秒后
reach：已经向上层ntp服务器要求更新的次数
delay：网络延迟
offset：时间补偿
jitter：系统时间与bios时间差
```

![img](https://cdn.nlark.com/yuque/0/2021/png/10386414/1633619202086-cdb55e78-1096-4e80-b0cc-29bfcf6ce7a4.png)

## 设置swap

```plain
echo 'vm.swappiness=10'>> /etc/sysctl.conf
sysctl vm.swappiness=10
```

## 关闭透明大页

```shell
# 执行
echo never > /sys/kernel/mm/transparent_hugepage/enabled
echo never > /sys/kernel/mm/transparent_hugepage/defrag

# 设置开机启动项
vim /etc/rc.local

if test -f /sys/kernel/mm/transparent_hugepage/enabled; then
        echo never > /sys/kernel/mm/transparent_hugepage/enabled
fi
if test -f /sys/kernel/mm/transparent_hugepage/defrag; then
        echo never > /sys/kernel/mm/transparent_hugepage/defrag
fi

# 保存退出之后：
chmod +x /etc/rc.local
```

## 设置 limits

```shell
cat >> /etc/security/limits.conf <<EOF
hdfs - nofile 32768
mapred - nofile 32768
hbase - nofile 32768
hdfs - noproc 32768
mapred - noproc 32768
hbase - noproc 32768
EOF
```

## 安装mysql驱动

```shell
mkdir -p /usr/share/java
scp mysql-connector-java-5.1.28.jar root@cdp01.cloud.com:/usr/share/java/mysql-connector-java.jar
scp mysql-connector-java-5.1.28.jar root@cdp02.cloud.com:/usr/share/java/mysql-connector-java.jar
scp mysql-connector-java-5.1.28.jar root@cdp03.cloud.com:/usr/share/java/mysql-connector-java.jar
```

## 安装JDK

```shell
# 安装JDK

# 启动scm时会去 /usr/java 下找jdk
mkdir -p /usr/java
ln -s /opt/jdk1.8.0_131  /usr/java/default
```

## 配置免密登陆（01节点）

```shell
ssh-keygen -t rsa
ssh-copy-id cdp01.cloud.com	
ssh-copy-id cdp02.cloud.com
ssh-copy-id cdp03.cloud.com
ssh-copy-id cdp04.cloud.com
```

## 安装mysql（01节点）

## 创建数据库 (01节点)

```shell
CREATE DATABASE scm DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON scm.* TO 'scm'@'%' IDENTIFIED BY 'xiaojianjun';
CREATE DATABASE amon DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON amon.* TO 'amon'@'%' IDENTIFIED BY 'xiaojianjun';
CREATE DATABASE rman DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON rman.* TO 'rman'@'%' IDENTIFIED BY 'xiaojianjun';
CREATE DATABASE hue DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hue.* TO 'hue'@'%' IDENTIFIED BY 'xiaojianjun';
CREATE DATABASE hive DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON hive.* TO 'hive'@'%' IDENTIFIED BY 'xiaojianjun';
CREATE DATABASE ranger DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON ranger.* TO 'rangeradmin'@'%' IDENTIFIED BY 'xiaojianjun';
CREATE DATABASE nav DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON nav.* TO 'nav'@'%' IDENTIFIED BY 'xiaojianjun';
CREATE DATABASE navms DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON navms.* TO 'navms'@'%' IDENTIFIED BY 'xiaojianjun';
CREATE DATABASE oozie DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL ON oozie.* TO 'oozie'@'%' IDENTIFIED BY 'xiaojianjun';
flush privileges;
```

# 制作本地源

## 配置本地源（01节点）

```shell
#安装工具包
yum install httpd createrepo -y
systemctl start httpd
systemctl enable httpd
systemctl status httpd

mkdir -p /var/www/html/cdp7
cd /var/www/html/cdp7

#下载CM7.4.4的安装包
wget https://archive.cloudera.com/cm7/7.4.4/redhat7/yum/RPM-GPG-KEY-cloudera
wget https://archive.cloudera.com/cm7/7.4.4/allkeys.asc
wget -nd -r -l1 --no-parent https://archive.cloudera.com/cm7/7.4.4/redhat7/yum/RPMS/x86_64/

#下载 Cloudera Runtime7.1.7的安装包
wget https://archive.cloudera.com/cdh7/7.1.7.0/parcels/manifest.json
wget https://archive.cloudera.com/cdh7/7.1.7.0/parcels/CDH-7.1.7-1.cdh7.1.7.p0.15945976-el7.parcel
wget https://archive.cloudera.com/cdh7/7.1.7.0/parcels/CDH-7.1.7-1.cdh7.1.7.p0.15945976-el7.parcel.sha256

createrepo .

#访问：http://cdp01.cloud.com/cdp7
```

## 制作Cloudera Manager的repo源(01节点)

```shell
vim cdp7.repo
[cdp7]
name=cdp7
baseurl=http://cdp01.cloud.com/cdp7
gpgcheck=0
enabled=1

scp cdp7.repo root@cdp01.cloud.com:/etc/yum.repos.d
scp cdp7.repo root@cdp02.cloud.com:/etc/yum.repos.d
scp cdp7.repo root@cdp03.cloud.com:/etc/yum.repos.d
scp cdp7.repo root@cdp04.cloud.com:/etc/yum.repos.d

# yum基本指令
yum clean all
yum makecache
yum list |grep cloudera
```

## 安装 Cloudera Manager Server（01节点）

```shell
# 通过yum安装Cloudera Manager Server
yum install cloudera-manager-server -y
```

# 镜像一下，方便回滚重装

#  安装CDH

```shell
# 创建CM库
/opt/cloudera/cm/schema/scm_prepare_database.sh mysql scm scm xiaojianjun

# cloudera-scm-server指令
systemctl start cloudera-scm-server
systemctl status cloudera-scm-server

# 查看cm server日志
tail -100f /var/log/cloudera-scm-server/cloudera-scm-server.log

# 登陆cm界面
http://cdp01.cloud.com:7180
```

## 问题一

```shell
2:postfix-2.10.1-6.el7.x86_64 has missing requires of libmysqlclient.so.18()(64bit)
2:postfix-2.10.1-6.el7.x86_64 has missing requires of libmysqlclient.so.18(libmysqlclient_18)(64bit)
# 重点关注：libmysqlclient.so.18()(64bit)
# 缺少Percona-XtraDB-Cluster-shared-55-5.5.37-25.10.756.el6.x86_64.rpm这个包
wget http://www.percona.com/redir/downloads/Percona-XtraDB-Cluster/5.5.37-25.10/RPM/rhel6/x86_64/Percona-XtraDB-Cluster-shared-55-5.5.37-25.10.756.el6.x86_64.rpm
rpm -ivh Percona-XtraDB-Cluster-shared-55-5.5.37-25.10.756.el6.x86_64.rpm
```

## 问题二

```shell
启动YQM时
CM Internal user credentials are not available as system environment variables. Please make sure that both CM_USER and CM_PASSWORD are present as environment variables for the application to work correctly
根据错误信息，在相应节点执行

vim /opt/cloudera/parcels/CDH-7.1.7-1.cdh7.1.7.p0.15945976/meta/cdh_env.sh

export CM_USER=admin
export CM_PASSWORD=admin
```

# 特性

## YARN开启动态资源池

```plain
1. 配置->类别->资源管理->yarn.scheduler.fair.allow-undeclared-pools,默认选项是开启的，需要关闭
2. 配置->ResourceManager Default Group->yarn.scheduler.fair.user-as-default-queue，该选项默认是开启，表示用户提交任务时，如果未指定池名称，就使用用户名作为默认的池名称，我用需要关闭该选项，让未指定此名称时，任务运行在default池中
3. 重启YARN服务
```

## 开启接口调用

```plain
vim /opt/cloudera/cm/webapp/WEB-INF/spring/mvc-config.xml

# 注释掉跨域拦截，重启CM
com.cloudera.server.web.cmf.csrf.CsrfRefererInterceptor
```