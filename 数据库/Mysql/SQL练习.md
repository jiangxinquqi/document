# SQL select语句完整的执行顺序

```plain
from-->where-->group by-->having-->计算所有的表达式-->order by-->select输出
```

# 自来水需求 

## 建表语句

```sql
drop table if exists t_account;
drop table if exists t_address;
drop table if exists t_area;
drop table if exists t_operator;
drop table if exists t_owner;
drop table if exists t_owner_type;
drop table if exists t_price_table;

CREATE TABLE IF NOT EXISTS t_owner_type(
	id int COMMENT '主键',
	name VARCHAR(30) not null COMMENT '类型名称',
	PRIMARY KEY(id)
)ENGINE=INNODB DEFAULT charset=utf8 COMMENT '业主类型表';


CREATE TABLE IF NOT EXISTS t_price_table(
	id int comment '主键',
	price DOUBLE(10,2) not null comment '价格',
	owner_type_id int not null comment '业主类型id',
	min_num DOUBLE(10,2) not null comment '区间数开始值',
	max_num DOUBLE(10,2) not null comment '区间数截至值',
	PRIMARY KEY(id)
)ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT '价格表';

CREATE TABLE IF NOT EXISTS t_area(
	id int COMMENT '主键',
	name varchar(20) NOT NULL COMMENT '区域名称',
	PRIMARY KEY(id)
)ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT '地区表';

CREATE TABLE if NOT EXISTS t_operator(
	id int COMMENT '主键',
	name VARCHAR(30) not null COMMENT '姓名',
	PRIMARY KEY(id)
)ENGINE=INNODB DEFAULT CHARSET=utf8 COMMENT '收费员表';

CREATE TABLE IF NOT EXISTS t_address(
	id int,
	name VARCHAR(30) not NULL COMMENT '地址名称',
	area_id INT not NULL COMMENT '区域id',
	operator_id INT not null COMMENT '操作员id',
	PRIMARY KEY(id)
)ENGINE=INNODB DEFAULT charset=utf8 COMMENT '地址表';

CREATE TABLE IF NOT EXISTS t_owner(
	id int,
	name VARCHAR(30) not NULL COMMENT '业主名称',
	address_id int not NULL COMMENT '地址id',
	house_number INT not null COMMENT '门牌号',
	water_meter VARCHAR(30) not null COMMENT '水表编号',
	add_date TIMESTAMP not null COMMENT '登记日期',
	owner_type_id int not null COMMENT '业主类型',
	PRIMARY KEY(id)
)ENGINE=INNODB DEFAULT charset=utf8 COMMENT '业主表';

CREATE TABLE if not EXISTS t_account(
	id int,
	owner_id INT not null COMMENT '业主编号',
	owner_type_id int not null COMMENT '业主类型',
	area_id int not NULL COMMENT '所在区域',
	year CHAR(4) not null COMMENT '账单年份',
	month CHAR(2) not null COMMENT '账单月份',
	num0 DOUBLE(20,2) COMMENT '上月累计数',
	num1 DOUBLE(20,2) COMMENT '本月累计数',
	use_num DOUBLE(20,2) COMMENT '本月使用数',
	meter_user_id int COMMENT '抄表员',
	meter_date TIMESTAMP COMMENT '抄表日期',
	money DOUBLE(10,2) COMMENT '应缴金额',
	is_fee CHAR(1) not NULL COMMENT '是否缴费',
	fee_date TIMESTAMP COMMENT '缴费日期',
	fee_user_id int COMMENT '收费员',
	PRIMARY KEY(id)
)ENGINE=INNODB DEFAULT charset=utf8 COMMENT '收费台账';

insert into t_owner_type values (1, '居民');
insert into t_owner_type values (2, '商业');
insert into t_owner_type values (3, '行政事业单位');

insert into t_price_table values (1, 1.50, 1, 0.00, 100.00);
insert into t_price_table values (2, 1.65, 1, 100.00, 300.00);
insert into t_price_table values (3, 2.10, 1, 300.00, 99999.00);
insert into t_price_table values (4, 2.50, 2, 0.00, 300.00);
insert into t_price_table values (5, 2.65, 2, 300.00, 600.00);
insert into t_price_table values (6, 3.10, 2, 600.00, 9999999.00);
insert into t_price_table values (7, 1.00, 3, 0.00, 9999900.00);

insert into t_area values (1, '武侯区');
insert into t_area values (2, '青羊区');
insert into t_area values (3, '金牛区');
insert into t_area values (4, '高新区');
insert into t_area values (5, '高新西区');
insert into t_area values (6, '锦江区');
insert into t_area values (7, '成华区');
insert into t_area values (8, '天府新区');
insert into t_area values (9, '双流区');
insert into t_area values (10, '天马区');

insert into t_operator values (1, '马小云');
insert into t_operator values (2, '马大腾');
insert into t_operator values (3, '李中复');
insert into t_operator values (4, '乐蒂');

insert into t_address values (1, '天鹅湖花园', 4, 1);
insert into t_address values (2, '锐力领峰', 4, 1);
insert into t_address values (3, '英郡', 4, 1);
insert into t_address values (4, '蜀都中心', 4, 1);
insert into t_address values (5, '南城都汇', 4, 1);
insert into t_address values (6, '富民新居', 9, 3);
insert into t_address values (7, '环球中心', 4, 2);
insert into t_address values (8, '孵化园', 4, 2);
insert into t_address values (9, '软件园', 4, 2);
insert into t_address values (10, '天府新谷', 4, 2);
insert into t_address values (11, '火车南站', 4, 2);

insert into t_owner values (1, '范冰', 1, '3-2-212', 'SK10488', '2003-01-30', 1);
insert into t_owner values (2, '王强', 1, '5-10-1012', 'SK10029', '2003-01-30', 1);
insert into t_owner values (3, '马腾', 1, '12-12-1203', 'SK10090', '2003-01-30', 1);
insert into t_owner values (4, '周建', 2, '2-5', 'SK20348', '2001-05-12', 1);
insert into t_owner values (5, '林杰', 3, '3-2', 'SK30209', '2010-02-04', 1);
insert into t_owner values (6, '周伦', 4, '1-9', 'SK50102', '2009-09-15', 1);
insert into t_owner values (7, '市第一人民医院', 11, '0', 'SK00020', '1998-12-30', 3);
insert into t_owner values (8, '凯德天府', 11, '0', 'SK91293', '2011-04-12', 2);
insert into t_owner values (9, '银泰城', 9, '0', 'SK921023', '2014-10-07', 2);
insert into t_owner values (10, 'T-Boys', 6, '9-1', 'SK40293', '2000-04-12', 1);

insert into t_account values(1, 7, 1, 4, '2016', '01', 2019273, 2119273, 10000, 2, '2016-02-01', 10000, '1','2016-02-05', 4);
insert into t_account values(2, 1, 1, 4, '2016', '01', 12012, 12100, 88, 1, '2016-02-01', 88*1.5, '1','2016-02-05', 4);
insert into t_account values(3, 1, 1, 4, '2016', '02', 12100, 12190, 90, 1, '2016-03-01', 90*1.5, '1','2016-03-05', 4);
insert into t_account values(4, 1, 1, 4, '2016', '03', 12190, 12200, 110, 1, '2016-04-01', 100*1.5+10*1.65, '1','2016-04-05', 4);
insert into t_account values(5, 1, 1, 4, '2016', '04', 12200, 12250, 50, 1, '2016-05-01', 50*1.5, '1','2016-05-05', 4);
insert into t_account values(6, 1, 1, 4, '2016', '05', 12250, 12400, 150, 1, '2016-06-01', 100*1.5+50*1.65, '1','2016-06-05', 4);
insert into t_account values(7, 1, 1, 4, '2016', '06', 12400, 12460, 60, 1, '2016-07-01', 60*1.5, '1','2016-07-05', 4);
insert into t_account values(8, 1, 1, 4, '2016', '07', 12460, 12540, 90, 1, '2016-08-01', 90*1.5, '1','2016-08-05', 4);
insert into t_account values(9, 1, 1, 4, '2016', '08', 12540, 12640, 100, 1, '2016-09-01', 100*1.5, '1','2016-09-05', 4);
insert into t_account values(10, 1, 1, 4, '2016', '09', 12640, 12729, 89, 1, '2016-10-01', 89*1.5, '1','2016-10-05', 4);
insert into t_account values(11, 1, 1, 4, '2016', '10', 12729, 12806, 67, 1, '2016-11-01', 67*1.5, '1','2016-11-05', 4);
insert into t_account values(12, 1, 1, 4, '2016', '11', 12806, 12862, 56, 1, '2016-12-01', 56*1.5, '1','2016-12-05', 4);
insert into t_account values(13, 1, 1, 4, '2016', '12', 12862, 12932, 70, 1, '2017-01-01', 70*1.5, '1','2017-01-05', 4);
insert into t_account values(14, 10, 1, 9, '2016', '01', 50120, 50408, 288, 1, '2016-02-01', 100*1.5+188*1.65, '1','2016-02-05', 4);
insert into t_account values(15, 10, 1, 9, '2016', '02', 50408, 50708, 300, 1, '2016-03-01', 100*1.5+200*1.65, '1','2016-03-05', 4);
insert into t_account values(16, 10, 1, 9, '2016', '03', 50708, 51028, 320, 1, '2016-04-01', 100*1.5+200*1.65+2.1*20, '1','2016-04-05', 4);
insert into t_account values(17, 10, 1, 9, '2016', '04', 51028, 51258, 230, 1, '2016-05-01', 100*1.5+130*1.65, '1','2016-05-05', 4);
insert into t_account values(18, 10, 1, 9, '2016', '05', 51258, 51508, 250, 1, '2016-06-01', 100*1.5+150*1.65, '1','2016-06-05', 4);
insert into t_account values(19, 10, 1, 9, '2016', '06', 51508, 51708, 200, 1, '2016-07-01', 100*1.5+100*1.65, '1','2016-07-05', 4);
insert into t_account values(20, 10, 1, 9, '2016', '07', 51708, 51964, 256, 1, '2016-08-01', 100*1.5+156*1.65, '1','2016-08-05', 4);
insert into t_account values(21, 10, 1, 9, '2016', '08', 51964, 52404, 340, 1, '2016-09-01', 100*1.5+200*1.65+2.1*40, '1','2016-09-05', 4);
insert into t_account values(22, 10, 1, 9, '2016', '09', 52404, 52683, 279, 1, '2016-10-01', 100*1.5+179*1.65, '1','2016-10-05', 4);
insert into t_account values(23, 10, 1, 9, '2016', '10', 52683, 52928, 245, 1, '2016-11-01', 100*1.5+145*1.65, '1','2016-11-05', 4);
insert into t_account values(24, 10, 1, 9, '2016', '11', 52928, 53138, 210, 1, '2016-12-01', 100*1.5+110*1.65, '1','2016-12-05', 4);
insert into t_account values(25, 10, 1, 9, '2016', '12', 53138, 53404, 266, 1, '2017-01-01', 100*1.5+166*1.65, '1','2017-01-05', 4);
insert into t_account values(26, 8, 2, 4, '2016', '01', 80192, 82393, 2201, 2, '2016-02-01', 300*2.5+300*2.65+3.1*1600, '1','2016-02-05', 4);
insert into t_account values(27, 12, 2, 4, '2016', '01', 80192, 82393, 2201, 2, '2016-02-01', 300*2.5+300*2.65+3.1*1600, '1','2016-02-05', 4);
insert into t_account values(28, 12, 2, 4, '2016', '01', 80192, 82393, 2201, 2, '2016-02-01', 300*2.5+300*2.65+3.1*1600, '1','2016-02-05',4);
insert into t_account values(29, 12, 4, 4, '2016', '01', 80192, 82393, 2201, 4, '2016-02-01', 300*2.5+300*2.65+3.1*1600, '1','2016-02-05',4);
```



##  单表查询

```sql
-- （一）简单条件查询
-- 需求：查询水表编号为 30408 的业主记录
SELECT * FROM t_owner where water_meter = 'SK10090';
-- 需求：查询业主名称包含“范”的业主记录
SELECT * FROM t_owner where name LIKE '%范%';
-- 需求：查询业主名称包含“周”的并且门牌号包含 2 的业主记录
SELECT * FROM t_owner where name like '%周%' and house_number LIKE '%2%';
-- 需求：查询业主名称包含“周”的或者门牌号包含 2 的业主记录
SELECT * FROM t_owner WHERE name like '%周%' OR house_number LIKE '%2%';
-- 需求：查询业主名称包含“周”的或者门牌号包含 2 的业主记录，并且地址编号为 3 的记录。
SELECT * FROM t_owner WHERE (name like '%周%' OR house_number LIKE '%2%')
and address_id = 3;

-- 需求：查询台账记录中用水字数大于等于 10000，并且小于等于 20000 的记录
SELECT * FROM t_account WHERE use_num BETWEEN 10000 and 20000;

-- 需求：查询 T_PRICE_TABLE 表中 MAXNUM 为空的记录
SELECT * FROM t_price_table WHERE max_num is NULL;

-- 需求：查询 T_PRICETABLE 表中 MAXNUM 不为空的记录
SELECT * FROM t_price_table WHERE max_num is not null;

-- （二）去掉重复记录
-- 需求：查询业主表中的地址 ID,不重复显示
SELECT DISTINCT address_id FROM t_owner;

-- （三）排序查询 
-- 需求：对 T_ACCOUNT 表按使用量进行升序排序
SELECT * FROM t_account ORDER BY use_num;
-- 需求：对 T_ACCOUNT 表按使用量进行降序排序
SELECT * from t_account ORDER BY use_num desc;

-- （四）分页查询
-- 需求：查询t_account 1-5 条记录
SELECT * from t_account LIMIT 5;
SELECT * FROM t_account LIMIT 0,5;
-- 需求：查询t_account 6-10 条记录
SELECT * FROM t_account LIMIT 5,5;
-- 需求：查询t_account 6条记录以后的所有记录
SELECT * FROM t_account LIMIT 5;

-- （五）聚合统计
-- 需求：统计 2016 年所有用户的用水量总和
SELECT SUM(use_num) as '2016用水总量' FROM t_account WHERE year = '2016';
-- 需求：统计 2016 年所有用水量（字数）的平均值
SELECT AVG(use_num) as '2016平均用水量' FROM t_account WHERE YEAR = '2016';
-- 需求：统计 2016 年最高用水量（字数）
SELECT MAX(use_num) as '2016最大用水量' FROM t_account WHERE YEAR = '2016';
-- 需求：统计 2012 年最低用水量（字数）
SELECT MIN(use_num) as '2016最小用水量' FROM t_account WHERE YEAR = '2016';
-- 需求：统计业主类型 ID 为 1 的业主数量
SELECT COUNT(*) as '业主类型为1的业主数量' FROM t_owner WHERE owner_type_id = 1;

-- 需求：按区域分组统计水费合计数
SELECT area_id,SUM(money) FROM t_account GROUP BY area_id;
-- 需求：查询水费合计大于 16900 的区域及水费合计
SELECT area_id,SUM(money) as sum_money FROM t_account GROUP BY area_id HAVING sum_money > 16900 ;
```

## 连接查询

```sql
-- （一）多表内连接查询
-- （1）需求：查询显示业主编号，业主名称，业主类型名称
SELECT 
	a.id '业主编号',
	a.`name` '业主名称',
	b.`name` '业主类型名称'
FROM t_owner a INNER JOIN t_owner_type b 
ON a.owner_type_id = b.id;

-- （2）需求：查询显示业主编号，业主名称、地址和业主类型
SELECT 
	a.id '业主编号',
	a.`name` '业主名称',
	c.`name` '地址',
	b.`name` '业主类型'
FROM 
	t_owner a,
	t_owner_type b,
	t_address c
WHERE a.address_id = c.id
AND a.owner_type_id = b.id;

-- （3）需求：查询显示业主编号、业主名称、地址、所属区域、业主分类
SELECT 
	a.id '业主编号',
	a.`name` '业主名称',
	c.`name` '地址',
	d.`name` '所属区域',
	b.`name` '业主分类'
FROM 
	t_owner a,
	t_owner_type b,
	t_address c,
	t_area d
WHERE a.address_id = c.id
AND a.owner_type_id = b.id
AND c.area_id = d.id;
	
-- （4）需求：查询显示业主编号、业主名称、地址、所属区域、收费员、业主分类
SELECT 
	a.id '业主编号',
	a.`name` '业主名称',
	c.`name` '地址',
	d.`name` '所属区域',
	f.`name` '收费员',
	b.`name` '业主分类'
FROM 
	t_owner a,
	t_owner_type b,
	t_address c,
	t_area d,
	t_operator f
WHERE a.address_id = c.id
AND a.owner_type_id = b.id
AND c.area_id = d.id
AND c.operator_id = f.id;

-- （二）左外连接查询
-- 需求：查询业主的账务记录，显示业主编号、名称、年、月、金额。如果此业主没有账务记录也要列出姓名。
SELECT 
	a.id '业主编号',
	a.`name` '名称',
	b.`year` '年',
	b.`month` '月',
	b.money '金额'
FROM t_owner a LEFT JOIN  t_account b ON a.id = b.owner_id;

-- （三）右外连接查询
-- 需求：查询业主的账务记录，显示业主编号、名称、年、月、金额。如果账务记录没有对应的业主信息，也要列出记录
SELECT 
	a.id '业主编号',
	a.`name` '名称',
	b.`year` '年',
	b.`month` '月',
	b.money '金额'
FROM t_owner a RIGHT JOIN  t_account b ON a.id = b.owner_id;
SELECT * FROM t_account;
```



##  子查询

```sql
-- （一）单行子查询
-- 需求：查询 2016 年 1 月用水量大于平均值的台账记录
SELECT * FROM t_account
WHERE year = '2016'
AND month = '01'
AND use_num > (SELECT AVG(use_num) FROM t_account WHERE `year` = '2016' AND `month` = '01');

-- （二）多行子查询
-- （1）需求：查询地址编号为 1 、3、4 的业主记录
SELECT * FROM t_owner WHERE address_id in(1,3,4);

-- （2）需求：查询地址含有“花园”的业主的信息
SELECT * FROM t_owner WHERE address_id in
(SELECT id FROM t_address WHERE `name` LIKE '%花园%');

-- （3）需求：查询地址不含有“花园”的业主的信息
SELECT * FROM t_owner WHERE address_id NOT in
(SELECT id FROM t_address WHERE `name` LIKE '%花园%');


-- 需求：查询 2016 年台账中，使用量大于 2016 年 3 月最大使用量的台账数据
SELECT * FROM t_account 
WHERE `year` = '2016' 
AND use_num > (SELECT MAX(use_num) FROM t_account WHERE `year` = '2016' AND `month` = '03');
```

## 集合运算

```sql
-- （一）什么是集合运算
-- 集合运算，集合运算就是将两个或者多个结果集组合成为一个结果集。集合运算
-- 包括：
-- UNION ALL(并集)，返回各个查询的所有记录，包括重复记录。
-- UNION(并集)，返回各个查询的所有记录，不包括重复记录。
-- INTERSECT(交集)，返回两个查询共有的记录。
-- MINUS(差集)，返回第一个查询检索出的记录减去第二个查询检索出的记录之后剩余的记录。

-- UNION ALL 不去掉重复记录
select * from t_owner where id<=7
UNION ALL
select * from t_owner where id>=5

-- UNION  不去掉重复记录
select * from t_owner where id<=7
UNION
select * from t_owner where id>=5
```

## 行列转换

```sql
-- 需求：按月份统计 2016 年各个地区的水费
SELECT (SELECT `name` FROM t_area WHERE id = area_id) '地区',
SUM(CASE WHEN `month` = '01' THEN money ELSE 0 END) '01月',
SUM(CASE WHEN `month` = '02' THEN money ELSE 0 END) '02月',
SUM(CASE WHEN `month` = '03' THEN money ELSE 0 END) '03月',
SUM(CASE WHEN `month` = '04' THEN money ELSE 0 END) '04月',
SUM(CASE WHEN `month` = '05' THEN money ELSE 0 END) '05月',
SUM(CASE WHEN `month` = '06' THEN money ELSE 0 END) '06月',
SUM(CASE WHEN `month` = '07' THEN money ELSE 0 END) '07月',
SUM(CASE WHEN `month` = '08' THEN money ELSE 0 END) '08月',
SUM(CASE WHEN `month` = '09' THEN money ELSE 0 END) '09月',
SUM(CASE WHEN `month` = '10' THEN money ELSE 0 END) '10月',
SUM(CASE WHEN `month` = '11' THEN money ELSE 0 END) '11月',
SUM(CASE WHEN `month` = '12' THEN money ELSE 0 END) '12月'
FROM t_account WHERE `year` = '2016' GROUP BY area_id;

-- 需求：按季度统计 2016 年各个地区的水费

SELECT (SELECT `name` FROM t_area WHERE id = area_id) '地区',
 SUM(CASE WHEN `month` in ('01','02','03') THEN money ELSE 0 END) '第一季度',
 SUM(CASE WHEN `month` in ('04','05','06') THEN money ELSE 0 END) '第二季度',
 SUM(CASE WHEN `month` in ('07','08','09') THEN money ELSE 0 END) '第三季度',
 SUM(CASE WHEN `month` in ('10','11','12') THEN money ELSE 0 end) '第三季度'
FROM t_account WHERE `year` = '2016' GROUP BY area_id;
```

## 综合案例

```sql
-- 1.收费日报单（总）统计某日的收费，按区域分组汇总
SELECT 
	(SELECT name FROM t_area WHERE id = area_id) '地区', 
	(SUM(use_num)/1000) '用水量（吨）' ,
	SUM(money) '金额' 
FROM t_account WHERE DATE_FORMAT(fee_date,'%Y-%m-%d') = '2016-02-05' GROUP BY area_id;

-- 2.收费日报单（收费员） 统计某收费员某日的收费，按区域分组汇总
SELECT 
	(SELECT `name` FROM t_area WHERE id = area_id) '区域',
	(SELECT `name` FROM t_operator WHERE id = fee_user_id) '收费员',
	SUM(use_num)/1000 '用水量（吨）',
	SUM(money) '收费'
FROM t_account
WHERE DATE_FORMAT(fee_date,'%Y-%m-%d') = '2016-02-05' 
AND fee_user_id = 4
GROUP BY area_id;

-- 3.收费月报表（总） 统计某年某月的收费记录，按区域分组汇总
SELECT 
	(SELECT `name` FROM t_area WHERE id = area_id) '区域',
	SUM(use_num)/1000 '用水量（吨）',
	SUM(money) '金额'
FROM t_account 
WHERE DATE_FORMAT(fee_date,'%Y-%m') = '2016-02'
GROUP BY area_id;

-- 4.收费月报表（收费员） 统计某收费员某年某月的收费记录，按区域分组汇总

SELECT 
	(SELECT `name` FROM t_area WHERE id = area_id) '区域',
	SUM(use_num)/1000 '用水量（吨）',
	SUM(money) '金额'
FROM t_account 
WHERE DATE_FORMAT(fee_date,'%Y-%m') = '2016-02' AND fee_user_id = 4
GROUP BY area_id;

-- 5.收费年报表（分区域统计）统计某年收费情况，按区域分组汇总
SELECT 
	(SELECT `name` FROM t_area WHERE id = area_id) '区域',
	SUM(use_num)/1000 '用水量(吨)',
	SUM(money) '金额'
FROM t_account
WHERE DATE_FORMAT(fee_date,'%Y') = '2016'
GROUP BY area_id;

-- 6.收费年报表（分月份统计） 统计某年收费情况，按月份分组汇总

SELECT 
	 CONCAT(DATE_FORMAT(fee_date,'%m'),'月') '月份',
	 SUM(use_num)/1000 '用水量（吨）',
	 SUM(money) '金额'
FROM t_account
WHERE DATE_FORMAT(fee_date,'%Y') = '2016'
GROUP BY DATE_FORMAT(fee_date,'%m')
ORDER BY DATE_FORMAT(fee_date,'%m');

-- 7.收费年报表（分月份统计） 统计某年收费情况，按月份分组汇总

SELECT '用水量（吨）' as '统计项',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '01' THEN use_num/1000 ELSE 0 END) '1 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '02' THEN use_num/1000 ELSE 0 END) '2 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '03' THEN use_num/1000 ELSE 0 END) '3 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '04' THEN use_num/1000 ELSE 0 END) '4 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '05' THEN use_num/1000 ELSE 0 END) '5 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '06' THEN use_num/1000 ELSE 0 END) '6 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '07' THEN use_num/1000 ELSE 0 END) '7 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '08' THEN use_num/1000 ELSE 0 END) '8 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '09' THEN use_num/1000 ELSE 0 END) '9 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '10' THEN use_num/1000 ELSE 0 END) '10月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '11' THEN use_num/1000 ELSE 0 END) '11月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '12' THEN use_num/1000 ELSE 0 END) '12月'
FROM t_account
WHERE DATE_FORMAT(fee_date,'%Y') = '2016'
UNION ALL
SELECT '金额' as '统计项',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '01' THEN money ELSE 0 END) '1 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '02' THEN money ELSE 0 END) '2 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '03' THEN money ELSE 0 END) '3 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '04' THEN money ELSE 0 END) '4 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '05' THEN money ELSE 0 END) '5 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '06' THEN money ELSE 0 END) '6 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '07' THEN money ELSE 0 END) '7 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '08' THEN money ELSE 0 END) '8 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '09' THEN money ELSE 0 END) '9 月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '10' THEN money ELSE 0 END) '10月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '11' THEN money ELSE 0 END) '11月',
	SUM(CASE WHEN DATE_FORMAT(fee_date,'%m') = '12' THEN money ELSE 0 END) '12月'
FROM t_account
WHERE DATE_FORMAT(fee_date,'%Y') = '2016';

-- 8.统计用水量，收费金额（分类型统计） 
-- 根据业主类型分别统计每种居民的用水量（整数，四舍五入）及收费金额 ，如果该类型在台账表中无数据也需要列出值为 0 的记录 
SELECT 
	ot.`name` '用户类型',
	IFNULL(FORMAT(SUM(ac.use_num)/1000,0),0) '用水量（吨）',
	IFNULL(SUM(ac.money),0) '金额'
FROM t_owner_type ot LEFT JOIN t_account ac ON  ot.id = ac.owner_type_id
GROUP BY ac.owner_type_id;

SELECT * FROM t_owner_type ot LEFT JOIN t_account ac ON  ot.id = ac.owner_type_id GROUP BY ac.owner_type_id; 

-- 9.统计每个区域的业主户数，并列出合计
SELECT 
	ar.`name` '区域',
	COUNT(ar.id) '用户数量'
FROM t_area ar, t_address ad ,t_owner ow 
WHERE ar.id = ad.area_id and ow.address_id = ad.id
GROUP BY ar.id
UNION ALL
SELECT '合计', COUNT(*) FROM t_owner; 

-- 10.统计每个区域的业主户数，如果该区域没有业主户数也要列出 0
SELECT 
 ar.`name` '区域',
 COUNT(t.owner_id) '业主户数'
FROM t_area ar LEFT JOIN 
	(SELECT ow.address_id 'address_id' ,ow.id 'owner_id',ad.area_id 'area_id' FROM t_owner ow, t_address ad WHERE ow.address_id = ad.id) t
ON ar.id = t.area_id
GROUP BY ar.id
UNION ALL
SELECT '合計',COUNT(*) FROM t_owner;
```

# Scott雇员系统

## DDL

```sql
EMP（雇员表）
    NO        字段      类型              描述
    1       EMPNO      NUMBER(4)       雇员编号
    2       ENAME     VARCHAR2(10)  表示雇员姓名
    3      JOB        VARCHAR2(9)    表示工作职位
    4      MGR        NUMBER(4)      表示一个雇员的领导编号
    5       HIREDATE   DATE           表示雇佣日期
    6      SAL        NUMBER(7,2)    表示月薪，工资
    7      COMM      NUMBER(7,2)    表示奖金或佣金
    8       DEPTNO     NUMBER(2)     表示部门编号

DEPT（部门表）
    NO          字段      类型                描述
    1          DEPTNO    NUMBER(2)        部门编号
    2          DNAME     VARCHAR2(14)     部门名称
    3          LOC        VARCHAR2(13)     部门位置

BONUS（奖金表）
    NO         字段         类型                描述
    1         ENAME      VARCHAR2(10)        雇员姓名
    2        JOB         VARCHAR2(9)         雇员工作
    3        SAL         NUMBER             雇员工资
    4        COMM       NUMBER              雇员奖金

SALGRADE（工资等级表）
    NO     字段            类型                 描述
    1      GRADE         NUMBER             等级名称
    2      LOSAL          NUMBER          此等级的最低工资
    3     HISAL           NUMBER          此等级的最高工资
	
create table DEPT(
	deptno INT(2) not null,
	dname VARCHAR(14),
	loc VARCHAR(13)
) engine=InnoDB charset=utf8;
alter table DEPT add constraint PK_DEPT primary key (DEPTNO);

create table EMP(
	empno INT(4) not null,
	ename VARCHAR(10),
	job VARCHAR(9),
	mgr INT(4),
	hiredate DATE,
	sal decimal(7,2),
	comm decimal(7,2),
	deptno INT(2)
) engine=InnoDB charset=utf8;
alter table EMP add constraint PK_EMP primary key (EMPNO);
alter table EMP add constraint FK_DEPTNO foreign key (DEPTNO) references DEPT (DEPTNO);

create table SALGRADE(
	grade INT,
	losal INT,
	hisal INT
) engine=InnoDB charset=utf8;

create table BONUS(
	ename VARCHAR(10),
	job VARCHAR(9),
	sal INT,
	comm INT
) engine=InnoDB charset=utf8 ;

insert into DEPT(DEPTNO, DNAME, LOC) values ('10', 'ACCOUNTING', 'NEW YORK');
insert into DEPT(DEPTNO, DNAME, LOC) values ('20', 'RESEARCH', 'DALLAS');
insert into DEPT(DEPTNO, DNAME, LOC) values ('30', 'SALES', 'CHICAGO');
insert into DEPT(DEPTNO, DNAME, LOC) values ('40', 'OPERATIONS', 'BOSTON');

insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7369', 'SMITH', 'CLERK', '7902','1980-12-17', '800', null, '20');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7499', 'ALLEN', 'SALESMAN', '7698', '1981-02-20', '1600', '300', '30');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7521', 'WARD', 'SALESMAN', '7698', '1981-02-22', '1250', '500', '30');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7566', 'JONES', 'MANAGER', '7839', '1981-04-02', '2975', null, '20');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7654', 'MARTIN', 'SALESMAN', '7698', '1981-09-28', '1250', '1400', '30');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7698', 'BLAKE', 'MANAGER', '7839', '1981-05-01', '2850', null, '30');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7782', 'CLARK', 'MANAGER', '7839', '1981-06-09', '2450', null, '10');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7788', 'SCOTT', 'ANALYST', '7566', '1987-06-13', '3000', null, '20');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7839', 'KING', 'PRESIDENT', null, '1981-11-17', '5000', null, '10');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7844', 'TURNER', 'SALESMAN', '7698', '1981-09-08', '1500', '0', '30');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7876', 'ADAMS', 'CLERK', '7788', '1987-06-13', '1100', null, '20');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7900', 'JAMES', 'CLERK', '7698', '1981-12-03', '950', null, '30');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7902', 'FORD', 'ANALYST', '7566', '1981-12-03', '3000', null, '20');
insert into EMP(EMPNO, ENAME, JOB, MGR, HIREDATE, SAL, COMM, DEPTNO) values ('7934', 'MILLER', 'CLERK', '7782', '1982-01-23', '1300', null, '10');

insert into SALGRADE(GRADE, LOSAL, HISAL) values ('1', '700', '1200');
insert into SALGRADE(GRADE, LOSAL, HISAL) values ('2', '1201', '1400');
insert into SALGRADE(GRADE, LOSAL, HISAL) values ('3', '1401', '2000');
insert into SALGRADE(GRADE, LOSAL, HISAL) values ('4', '2001', '3000');
insert into SALGRADE(GRADE, LOSAL, HISAL) values ('5', '3001', '9999');
```

## 综合案例

```plain
1.请从表 EMP 中查找工种是职员 CLERK 或经理 MANAGER 的雇员姓名、工资。
2.请在 EMP 表中查找部门号在 10－30 之间的雇员的姓名、部门号、工资、工作。
3.请从表 EMP 中查找姓名以 J 开头所有雇员的姓名、工资、职位。
4.请从表 EMP 中查找工资低于 2000 的雇员的姓名、工作、工资，并按工资降序排列。
5.请从表中查询工作是 CLERK 的所有人的姓名、工资、部门号、部门名称以及部门地址的信息。
6.在表 EMP 中查询所有工资高于 JONES 的所有雇员姓名、工作和工资。
7.列出没有对应部门表信息的所有雇员的姓名、工作以及部门号。
8.查找工资在 1000～3000 之间的雇员所在部门的所有人员信息
9.雇员中谁的工资最高。
10.查询所有雇员的姓名、SAL 与 COMM 之和。
11.查询所有 1981 年 7 月 1 日以前来的员工姓名、工资、所属部门的名字
12.查询各部门中 1981 年 1 月 1 日以后来的员工数
13.查询所有在CHICAGO工作的经理MANAGER和销售员SALESMAN的姓名、工资
14.查询列出来公司就职时间超过 24 年的员工名单
15.查询于 1981 年来公司所有员工的总收入（SAL 和 COMM）
16.查询显示每个雇员加入公司的准确时间，按××××年××月××日 时分秒显示。
17.查询公司中按年份月份统计各地的录用职工数量
18.查询列出各部门的部门名和部门经理名字
19.查询部门平均工资最高的部门名称和最低的部门名称
20.查询与雇员号为7521员工的最接近的在其后进入公司的员工姓名及其所在部门名

1. 创建视图 view_emp，显示雇员表中的 EMPNO ENAME JOB
2. 创建带约束的视图 view_emp30，显示部门编号为 30 的雇员信息。
3. 创建只读视图，显示部门表中的信息。
4. 创建物化视图（自动刷新），显示雇员编号、雇员名称、雇员职位和雇员部门。
5. 创建物化视图（手动刷新），查询列出各部门的部门名和部门经理名字。并编写手动刷新命令。
6. 编写序列 seq_1 ,从 100 开始 ，增长 10 ，最大值 1000，最小值 10 ，循环 。
7. 编写序列 seq_2 ,最大值 100 ，最小值 5，增长值 5 ，不循环。
8. 编写序列 SEQ_EMP, 起始值 8000，增长 1 ，不循环，不缓存。
9. 编写序列 SEQ_DEPT, 起始值 50，增长 10 ，不循环，缓存 30。
10. 根据雇员名称对雇员表建立索引
11. 根据部门编号和职位对雇员表建立索引
12. 在奖金表根据职位建立位图索引
13.为 EMP 表创建私有同义词
14.为 DEPT 表创建公有同义词
```

## 解答