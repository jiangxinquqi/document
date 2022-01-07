# 背景

最近在做一个审计日志组件，简单来说就是把用户的操作日志存储到Solr索引库，然后提供一个查询接口给前段。旧有的日志是硬编码在业务逻辑中，且持久化的方式是mysql，所以这里面就涉及到一个存量数据迁移到solr的过程。

在进行联调的时候踩了一个关于时区的坑，所以就有了这篇文章。审计日志里面有一个字段用来描述用户的操作时间的。同一个操作，应用展现的是一个时间，solr存储的是另外一个时间，mysql里面是第三个时间。简单的看了一下三个时间差异，定位到是时区问题，所以展开了排查。



# 问题排查

整个过程中涉及到的时间转换过程主要是：

mysql -> 后端 -> solr -> 后端 -> 前端。



## 查看各服务的时区

1. 前端时区为 UTC +8。
2. solr索引库的时区设置为 UTC +8，但是存储的形式是以 UTC存储的，这里比前端晚了8个小时。

1. 查看应用服务器的时区，也是UTC +8，结果如下：

```shell
$ date -R  
Fri, 09 Apr 2021 08:23:50 +0800
```

1. 查看mysql的时区，结果如下：

```shell
$ show variables like'%time_zone';

+------------------+--------+
| Variable_name    | Value  |
+------------------+--------+
| system_time_zone |   CST  |
| time_zone        | SYSTEM |
+------------------+--------+
```

CST可视为美国、澳大利亚、古巴或中国的标准时间，这里有歧义，所以我又查询了mysql的系统时间。

1. 查询mysql的系统时间

```shell
select now();
```

发现与我本机的时间一样，说明mysql认为该CST为中国标准时间。

这就很很郁闷了，各服务的时区设置都是中国标准时间，为什么最终为出现三个不同的时间表示呢？



## 分析



先说结论，做了简单的测试，创建数据库连接，查看当前session的时区为CST，再select SYSDATE(); 

查看时间，与当前符合，说明这里的CST等于北京时间。

坑就坑在CST有歧义，mysql觉得CST是北京时间，java那边觉得是美国时间，有四个时区的缩写都可以叫CST。说到底还是mysql的坑，postgre那边就用的Asia/shanghai，没这个问题。下面是具体分析：



首先来看一下原因。升级到Boot 2.1之后，MySQL Connect/J版本也随之升级到8.0，会优先使用连接参数（`serverTimezone`）中指定的时区，如果没有指定，则再使用数据库配置的时区，参考下面的[官宣](https://link.zhihu.com/?target=https%3A//dev.mysql.com/doc/connector-j/8.0/en/connector-j-other-changes.html)（对应的源代码是`com.mysql.cj.protocol.a.NativeProtocol#configureTimezone()`）。由于我们之前数据库连接参数没有指定时区，并且数据库配置的是默认的`CST`时区（美国中部时区，即-6:00），所以读取出来的时间出现偏差。



Connector/J 8.0 always performs time offset adjustments on  date-time values, and the adjustments require one of the following to be  true:

- The MySQL server is configured with a  canonical time zone that is recognizable by Java (for example,  Europe/Paris, Etc/GMT-5, UTC, etc.)
- The server's time zone is overridden by setting the Connector/J connection property `serverTimezone` (for example, `serverTimezone=Europe/Paris`).



找到原因之后，解决办法就比较直白了，



方法一：数据库的连接参数添加`serverTimezone=Asia/Shanghai`或者`serverTimezone=GMT%2B8`。Boot 1.5下不需要添加此参数，但添加了也无妨。

方法二：修改MySQL数据库的time_zone配置，改为`+8:00`（默认是`SYSTEM`）。采用此方法，则不需要修改数据库连接参数。



方法二显然更优，一次修改，终生受益。但要注意，对于升级到Boot 2.1之后新生成的那批数据，如果包含时间类型的字段并且该字段值是应用指定的而不是数据库生成的（例如`DEFAULT CURRENT_TIMESTAMP`），那么需要手动修复（加上偏差的小时数）。



两个解决办法都很简单，有同学马上会问，为什么Boot 1.5下没有这个问题？为什么Boot 2.0下读取历史数据存在14个小时的偏差，而新生成的数据又是好的？要回答这两个问题，看官宣就不够了，需要读一下MySQL Connect/J的源代码。



谜题一，为什么Boot 1.5下没有这个问题？答案隐藏在`com.mysql.jdbc.ResultSetImpl`和`com.mysql.jdbc.ConnectionImpl`两个类的源代码中。



```plain
// 源代码：com.mysql.jdbc.ResultSetImpl
private TimeZone getDefaultTimeZone() {
        // useLegacyDatetimeCode默认为true，因此使用connection的默认时区
        return this.useLegacyDatetimeCode ? this.connection.getDefaultTimeZone() : this.serverTimeZoneTz;
    }
// 源代码：com.mysql.jdbc.ConnectionImpl
public ConnectionImpl(String hostToConnectTo, int portToConnectTo, Properties info, String databaseToConnectTo, String url) throws SQLException {
        // connection的默认时区使用的是JVM的默认时区，一般为操作系统的时区
        // We store this per-connection, due to static synchronization issues in Java's built-in TimeZone class...
        this.defaultTimeZone = TimeUtil.getDefaultTimeZone(getCacheDefaultTimezone());
}
```



Boot 1.5下，MySQL Connect/J默认使用操作系统的时区（Asia/Shanghai，即+8:00），而忽略连接参数或者数据库指定的时区，因此不管是读数据还是写数据都是使用统一的时区，因此不存在时间偏差。



谜题二，为什么Boot  2.0下读取历史数据存在14个小时的偏差，而新生成的数据又是好的？升级到Boot 2.0之后，MySQL  Connect/J改为使用数据库配置的CST时区，而历史数据是在Boot  1.5下的Asia/Shanghai时区生成的，因此读出来存在14（-6:00和+8:00之间）个小时的偏差。对于新生成的数据，由于同处在CST时区下，因此没有偏差。