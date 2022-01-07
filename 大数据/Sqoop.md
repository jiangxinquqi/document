# 安装

```xml
# 上传解压软件包
#  添加sqoop到环境变量
# 将数据库连接驱动拷贝到$SQOOP_HOME/lib里面
```

# 使用

## 1. 将数据库中的数据导入到HDFS上

```bash
# 简单导入
sqoop import 
--connect jdbc:mysql://db01:3306/scott 
--username root
--password xiaojianjun
--table emp
--target-dir '/user/hive/warehouse/xiaojianjun.db/emp'
--fields-terminated-by '\t'
--columns 'empno, ename, job, mgr, hiredate, sal, comm, deptno'
 
# 指定输出路径、指定数据分隔符
sqoop import 
	--connect jdbc:mysql://192.168.1.10:3306/itcast 
  --username root --password 123  
  --table trade_detail 
  --target-dir '/sqoop/td' 
  --fields-terminated-by '\t'
# 指定Map数量 -m   
sqoop import 
	--connect jdbc:mysql://192.168.1.10:3306/itcast 
  --username root 
  --password 123  
  --table trade_detail 
  --target-dir '/sqoop/td1' 
  --fields-terminated-by '\t' 
  -m 2
# 增加where条件, 注意：条件必须用引号引起来
sqoop import 
	--connect jdbc:mysql://192.168.1.10:3306/itcast 
  --username root --password 123  
  --table trade_detail 
  --where 'id>3' 
  --target-dir '/sqoop/td2' 
  
# 增加query语句(使用 \ 将语句换行)
# 注意：
#		如果使用--query这个命令的时候，需要注意的是where后面的参数，AND $CONDITIONS这个参数必须加上
#		存在单引号与双引号的区别，如果--query后面使用的是双引号，那么需要在$CONDITIONS前加上\即\$CONDITIONS
#		如果设置map数量为1个时即-m 1，不用加上--split-by ${tablename.column}，否则需要加上
sqoop import 
	--connect jdbc:mysql://192.168.1.10:3306/itcast 
  --username root 
  --password 123 
  --query 'SELECT * FROM trade_detail where id > 2 AND $CONDITIONS' 
  --split-by trade_detail.id 
  --target-dir '/sqoop/td3'
  
```

## 2. 将HDFS上的数据导出到数据库中

```bash
sqoop export 
	--connect jdbc:mysql://192.168.8.120:3306/itcast 
  --username root 
  --password 123 
  --export-dir '/td3' 
  --table td_bak 
  -m 1 
  --fields-termianted-by '\t'
```

​	