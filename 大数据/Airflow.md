# 安装Python

```shell
# 下载安装保
wget https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz
tar -zxvf Python-3.6.5.tgz
mkdir /opt/python3

# 安装依赖
yum install gcc -y
yum install zlib* -y 
yum install openssl-devel -y 
yum install readline-devel.*

# 编译
cd Python-3.6.5
./configure --prefix=/opt/python3 --with-ssl --enable-optimizations
make && make install

# 设置环境变量
rm -rf /usr/bin/python
ln -s /opt/python3/bin/python3.6 /usr/bin/python
```

## 中文编码问题

```shell
Python中默认的编码格式是 ASCII 格式，在没修改编码格式时无法正确打印汉字，所以在读取中文时会报错。
解决方法为只要在文件开头加入 # -*- coding: UTF-8 -*- 或者 # coding=utf-8 就行了 
```

# 安装Airflow

```shell
# 升级pip
pip3 install --upgrade pip
# 依赖
pip3 install apache-airflow[mysql]

export AIRFLOW_HOME=/opt/airflow
pip install apache-airflow

# 安装依赖
yum install -y mysql-devel gcc gcc-devel gcc-c++ python-devel libffi-devel openssl-devel libsasl2-dev
pip3 install mysqlclient pymysql mysql -i  https://mirrors.aliyun.com/pypi/simple

# 修改cfg文件
[core]
executor=LocalExecutor
sql_alchemy_conn = mysql://user:password@IP:3306/airflow

# 初始化元数据
airflow db init

# 数据库执行脚本
create database airflow CHARACTER SET = utf8mb4;
set explicit_defaults_for_timestamp = 1;

# 创建用户
airflow users create \
    --username root \
    
    --firstname xiaojianjun \
    --lastname xiaojianjun \
    --role Admin \
    --email spiderman@superhero.org

# 配置path路径
export PATH=${PATH}:/opt/python3/bin/

# 启动
airflow webserver --port 8080
airflow scheduler

# 后台启动

nohup airflow worker>>$AIRFLOW_HOME/logs/airflow-worker.log 2>&1 &
nohup airflow scheduler>>$AIRFLOW_HOME/logs/airflow-scheduler.log 2>&1 &
nohup airflow webserver>>$AIRFLOW_HOME/logs/airflow-webserver.log 2>&1 &
```