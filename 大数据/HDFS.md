#  简介

​	HDFS支持Quota功能，简单总结：Hadoop提供两种配额（quota）模式，name quota 和 space quota。

​	HDFS 允许管理员对目录下的子目录和文件个数(Name Quotas)，以及目录下数据存储大小(Space Quotas)进行配额限制。

名称配额和空间配额是独立运作的，但这两种配额的管理和实施是密切并行的。



> 名称配额(Name Quotas) 

​	名称配额是对目录树中的文件和目录名的数量的硬限制。

​	如果超出配额，则文件和目录创建失败。新创建的目录没有关联的配额，最大的配额是Long.Max_Value。

​	一个配额强制一个目录保持空白。(是的，一个目录会计入它自己的配额!)。

> 空间配额(Space Quotas) 限制的”磁盘”占用空间。可以通过此功能实现团队的存储使用管理。

​	空间配额是对目录树中的文件所使用的字节数的硬限制。

​	如果配额不允许写入整个块，则块分配失败。

​	一个块的每个副本都按配额计数。

​	新创建的目录没有关联的配额，最大的配额是Long.Max_Value。

​	零配额仍然允许创建文件，但是不能向文件添加任何块

名称配额和空间配额在fsimage中是持久化的。在启动时，如果fsimage立即违反了配额(可能fsimage被偷偷修改了)，则会对每一次违反都打印一个警告。设置或删除配额将创建一个日志条目。

> 存储类型配额

存储类型配额是对目录树中的文件使用特定存储类型(SSD、 DISK、 ARCHIVE)的硬限制。它在许多方面类似于存储空间配额，但提供了对集群存储空间使用的细粒度控制。要在目录上设置存储类型配额，必须在目录上配置存储策略，以便根据存储策略将文件存储在不同的存储类型中。有关更多信息，请参见 HDFS 存储策略文档。

存储类型配额可以与空间配额和名称配额结合起来，以有效地管理集群存储使用。比如说,

对于配置了存储策略的目录，管理员应该为资源约束存储类型(如 SSD)设置存储类型配额，为其他存储类型和总体空间配额设置限制较少的值或默认无限制。HDFS 将根据存储策略和总体空间配额从目标存储类型中扣除配额

对于未配置存储策略的目录，管理员不应配置存储类型配额。即使特定存储类型不可用(或可用，但没有正确配置存储类型信息) ，也可以配置存储类型配额。但是，在这种情况下，建议使用总体空间配额，因为存储类型信息对于存储类型配额强制不可用或不准确

DISK 上的存储类型配额的使用受到限制，除非 DISK 不是主要的存储介质。(例如以存档为主的集群)

# 管理员命令

所有的hdfs指令均可以尝试替换为hadoop指令

## set HDFS name quotas

设置目录下的文件个数限制。本身占用一个额度。

    $ hadoop dfsadmin -setQuota <max_number> <directory>

## set HDFS space quotas

这是对目录下所有文件的总大小的硬限制。空间配额也考虑到副本，即一个 1GB 3个副本的数据将消耗3 GB 的配额。

    $ hadoop dfsadmin -setSpaceQuota <max_size:1m> <directory>
    $ hadoop dfsadmin -setSpaceQuota -storageType <directory>

## To clear quotas

    hadoop dfsadmin -clrQuota <directory>
    hadoop dfsadmin -clrSpaceQuota <directory>
    hadoop dfsadmin -clrSpaceQuota -storageType <directory>
    
    # 新建文件
    hadoop dfs -mkdir /name_quota
    # 从HDFS中删除文件
    hadoop fs -rm /file_name
    # 从HDFS中删除文件夹
    hadoop fs -rm -r /folder_name



# 报告命令

报告配额值和当前使用的名称和字节数。

```shell
hadoop fs -count -q [-h][-v] [-t [comma-separated list of storagetypes]] <directory>
# 常用
hadoop fs -count -q -v -h <directory>
```

## **1. 命令选项**

​	q 选项可以报告每个目录的名称配额值、剩余的可用名称配额、空间配额值和剩余的可用空间配额。如果目录没有配额集，则报告的值为 none 和 inf。

​	h 选项以人类可读的格式显示大小。

​	v 选项显示一行标题。

​	t 选项显示每个存储类型配额集和每个目录剩余的可用配额。如果给出了-t 选项，则只显示指定类型的配额和剩余配额。否则，将显示支持配额的所有存储类型的配额和剩余配额。

## **2. 报告详解**

QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT     CONTENT_SIZE PATHNAME
10                 7             			1 G           		256 M            			1            	2             		 	256 M 		/quota

- QUOTA ：文件（包含文件和文件夹）数量配额
- REM_QUOTA  ：剩余文件（包含文件和文件夹）数量（该路径本身占用一个数量）
- SPACE_QUOTA ：磁盘配额
- REM_SPACE_QUOTA   ：剩余磁盘配额
- DIR_COUNT ：文件夹数量（包含自身）
- FILE_COUNT  ：文件数量
- CONTENT_SIZE ：磁盘占用大小
- PATHNAME：路径名称

# 示例

## Name Quota示例

- 首先测试name quota，进行如下操作：

    ```shell
    $hadoop dfs -mkdir /name_quota
    $hadoop dfsadmin -setQuota 5 /name_quota
    ```

- 准备hdfs路径，设置name quota，文件数量限制为5，

    ```shell
    $touch 1
    $hadoop fs -put 1 /name_quota/1
    $hadoop fs -put 1 /name_quota/2
    $hadoop fs -put 1 /name_quota/3
    $hadoop fs -put 1 /name_quota/4
    $hadoop fs -put 1 /name_quota/5
    
    copyFromLocal: The NameSpace quota (directories and files) of directory /name_quota is exceeded: quota=5 file count=6
    ```

创建一个临时文件“1“，并将它复制到指定hdfs路径下，发现，当复制到第5个时失败，提示显示达到quota限制，说明quota生效，同时，我们设置quota_max_num为5，但只放了4个文件就满了，说明这个数字限制是包含文件夹的。

实际上是可以理解的，在HDFS中，无论文件和文件夹的基本抽象是一样的，占用基本同样的元数据信息，name quota实际上是元数据信息的配额。

## Space Quota示例

> 创建测试路径并设置space quota

    $hadoop fs -mkdir /space_quota
    $hadoop dfsadmin -setSpaceQUota 50m /space_quota
    dd if=/dev/zero of=xiao.text bs=10MB count=1
    
    $hadoop fs -put xiao.text /space_quota/xiao_1

利用dd命令创建一个大小10m的文件，预期是在这个hdfs路径下可以存放5个10m的文件，第六个会超出，但实际情况是，第一个就失败了，报错如下：

    Caused by: org.apache.hadoop.ipc.RemoteException(org.apache.hadoop.hdfs.protocol.DSQuotaExceededException): The DiskSpace quota of /space_quota is exceeded: quota = 52428800 B = 50 MB but diskspace consumed = 134217728 B = 128 MB

看提示就理解，因为HDFS是以block为单位进行实际存储的，对于一个10m的文件，仍然会打成一个block存储，我的HDFS block size是128m，所以space quota配置50m，一个block也放不下，自然失败，因此进行一下改动，重新设置：

    $ hadoop dfsadmin -setSpaceQUota 1024m /space_quota
    $ dd if=/dev/zero of=xiao.text bs=128MB count=1
    
    $ hadoop fs -put xiao.text /space_quota/xiao_1
    $ hadoop fs -put xiao.text /space_quota/xiao_2
    $ hadoop fs -put xiao.text /space_quota/xiao_3
    $ hadoop fs -put xiao.text /space_quota/xiao_4
    $ hadoop fs -put xiao.text /space_quota/xiao_5
    $ hadoop fs -put xiao.text /space_quota/xiao_6
    $ hadoop fs -put xiao.text /space_quota/xiao_7
    $ hadoop fs -put xiao.text /space_quota/xiao_8
    $ hadoop fs -put xiao.text /space_quota/xiao_9

1024m的配额，可以放8个128m文件，在放第九个时，报错，核心信息如下：

    18/05/18 16:13:01 WARN hdfs.DFSClient: DataStreamer Exception
    org.apache.hadoop.hdfs.protocol.DSQuotaExceededException: The DiskSpace quota of /space_quota is exceeded: quota = 1073741824 B = 1 GB but diskspace consumed = 1207959552 B = 1.13 GB

符合预期，成功限制了规定路径下的存储空间配额。

另外，经过测，name quota和space quota可以同时作用于同一路径。

# 测试

> 查看块和副本： 

```shell
[hadoop@hadoop1 test]$ hadoop fs -stat "%o %r"  /quota/128m_2
134217728 3
```

> 可知，块为128M，3副本的集群

设10个nameQuota和1024M的spaceQuota，注意：这里的spaceQuota是物理空间！申请的是物理空间！ 

```shell
hadoop fs -mkdir /quota
dd if=/dev/zero of=128m bs=1M  count=128

# 设置quota 10 和 1024M
hadoop dfsadmin -setQuota 10 /quota
hadoop dfsadmin -setSpaceQuota 1024m /quota

# 初始查看
hadoop fs -count -q -v -h /quota
QUOTA  REM_QUOTA  SPACE_QUOTA  REM_SPACE_QUOTA   DIR_COUNT   FILE_COUNT  CONTENT_SIZE PATHNAME
none      inf         1 G             1 G            1            0               0 	/quota

# 上传一个128M块
hdfs dfs -copyFromLocal 128m /quota/128m_1

# 查看，可知nameQuota占2个，原因是一个目录，一个文件。
# spaceQuota居然剩余640M，原因是三副本，128*3=384M。1024-384M=640M
hadoop fs -count -q -v -h /quota
       QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT       CONTENT_SIZE PATHNAME
          10               8             1 G           640 M            1            1              128 M /quota

# 再上传一个128M的文件
hdfs dfs -copyFromLocal 128m /quota/128m_2

# 再查看，可知nameQuota占用两文件一目录，还剩7个。
# spaceQuota占用128*2*3=768M，还剩256M
[hadoop@hadoop1 test]$ hadoop fs -count -q -v -h /quota
       QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT       CONTENT_SIZE PATHNAME
          10               7             1 G           256 M            1            2              256 M /quota

# 上传第三个文件时报错
[hadoop@hadoop1 test]$ hdfs dfs -copyFromLocal 128m /quota/128m_3
copyFromLocal: The DiskSpace quota of /quota is exceeded: quota = 1073741824 B = 1 GB but diskspace consumed = 1207959552 B = 1.13 GB
```

可知，申请1024M的空间，只能放128M的文件2个，因为两个128M的逻辑空间有256M，物理空间已经是768M了！ 

## 1.小文件测试

当上传10M后，实际占用128M还是10M呢？答案是10M。 

```shel
[hadoop@hadoop1 test]$ hadoop fs -copyFromLocal 10m /quota/10m_1
[hadoop@hadoop1 test]$ hadoop fs -copyFromLocal 10m /quota/10m_1
# 1024M，上传10M后，占用30M
```

[hadoop@hadoop1 test]$ hadoop fs -count -q -v -h /quota
       QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT       CONTENT_SIZE PATHNAME
          10               8             1 G           994 M            1            1               10 M /quota

## 2.设小块

### 测试1：设置50M的quota

```shell
hadoop fs -mkdir /quota50
# 设50M
hadoop dfsadmin -setSpaceQuota 50m /quota50
# 查看quota
[hadoop@hadoop1 test]$ hadoop fs -count -q -v -h /quota50
       QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT       CONTENT_SIZE PATHNAME
        none             inf            50 M            50 M            1            0                  0 /quota50
# 上传10M，上传不了
[hadoop@hadoop1 test]$ hadoop fs -copyFromLocal 10m /quota50/10m_1
copyFromLocal: The DiskSpace quota of /quota50 is exceeded: quota = 52428800 B = 50 MB but diskspace consumed = 402653184 B = 384 MB
```

可以看到，即使设置50M的quota，上传一个文件，也是不能上传的。10M的文件也要占用128的空间，需要3个块，所以需要`128*3M=384M`的空间。 

### 测试2：我们设置200M的quota

```shell
hadoop fs -mkdir /quota200
hadoop dfsadmin -setSpaceQuota 200m /quota200

# 查看quota
[hadoop@hadoop1 test]$ hadoop fs -count -q -v -h /quota200
       QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT       CONTENT_SIZE PATHNAME
        none             inf           200 M           200 M            1            0                  0 /quota200
```

200M是128M+72M。我们上传一个10M的，仍然上传不了。仍然需要384M的空间。 

    [hadoop@hadoop1 test]$ hadoop fs -copyFromLocal 10m /quota200/10m_1
    copyFromLocal: The DiskSpace quota of /quota200 is exceeded: quota = 209715200 B = 200 MB but diskspace consumed = 402653184 B = 384 MB

### 测试3：那么，我们建400M的quota，能否传两个10M呢？不能！

```shell
[hadoop@hadoop1 test]$ hadoop fs -mkdir /quota400
[hadoop@hadoop1 test]$ hadoop dfsadmin -setSpaceQuota 400m /quota400

# 查看quota
[hadoop@hadoop1 test]$ hadoop fs -count -q -v -h /quota400
       QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT       CONTENT_SIZE PATHNAME
        none             inf           400 M           400 M            1            0                  0 /quota400
        
# 上传10M
[hadoop@hadoop1 test]$ hadoop fs -copyFromLocal 10m /quota400/10m_1

# 查看quota
[hadoop@hadoop1 test]$ hadoop fs -count -q -v -h /quota400
 QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT       CONTENT_SIZE PATHNAME
        none             inf           400 M           370 M            1            1               10 M /quota400
```

看到第一次上传10M，占用了30M，剩余370M，第二次上传失败！

    [hadoop@hadoop1 test]$ hadoop fs -copyFromLocal 10m /quota400/10m_2
    copyFromLocal: The DiskSpace quota of /quota400 is exceeded: quota = 419430400 B = 400 MB but diskspace consumed = 434110464 B = 414 MB

结论：设置quota一定要设置块大小（128M）的倍数！

### 测试4：1024M的quota可以分8个block块，那么上传10M的文件，能上传多少个呢？

是不是一个10M占3个块，最多传2个10M（第三个就是9个块）呢，答案：不是的。
上传15个10M后，查看quota：

[hadoop@hadoop1 test]$ hadoop fs -count -q -v -h /quota
       QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT       CONTENT_SIZE PATHNAME
        none             inf             1 G           574 M            1           15              150 M /quota

上传22个文件后，查看quota：

[hadoop@hadoop1 test]$ hadoop fs -count -q -v -h /quota
       QUOTA       REM_QUOTA     SPACE_QUOTA REM_SPACE_QUOTA    DIR_COUNT   FILE_COUNT       CONTENT_SIZE PATHNAME
        none             inf             1 G           364 M            1           22              220 M /quota

上传第23个时，出错：

    [hadoop@hadoop1 test]$ hadoop fs -copyFromLocal 10m /quota/10m_23
    copyFromLocal: The DiskSpace quota of /quota is exceeded: quota = 1073741824 B = 1 GB but diskspace consumed = 1094713344 B = 1.02 GB

可以分析出，最后一个文件，必须要先留有128 x 3=384M的空间，但是实际占用是文件自己的大小。

设置quota最小为128 x 3 = 384M的空间，否则再小的文件也传不了。

## 总结

​	实际占用大小是文件自己的大小。

​	对于3副本，必须要要留有3个块的大小即128M*3=384M的空间，才能上传文件。

​	quota 和 space quota 的推荐设置为 quota = ( space quota / block size / block copy ) * 放大系数。

# Java 编程API

```java
private void test() throws Exception{
    // 初始化hdfs
    Configuration conf = new Configuration();
    conf.addResource(new FileInputStream("/xxxxx/hive-conf/core-site.xml"));
    conf.addResource(new FileInputStream("/xxxxx/hive-conf/hdfs-site.xml"));
    
    UserGroupInformation.setConfiguration(conf);
    UserGroupInformation.loginUserFromKeytab("hdfs_admin", "/xxxx/hdfs_admin.keytab");
    
    FileSystem fs = FileSystem.get(conf);
    
    Path path = new Path("/");
    // 查询指定的文件（或者文件夹）是否存在
    fs.exist(path);
    // 创建指定的文件（或者文件夹）
    fs.mkdir(path);
    
    // 获取dfsadmin
    HdfsAdmin dfsadmin = new HdfsAdmin(fs.getUri(), fs.getConf());
    
    // set quota
    dfsadmin.setQuota(path, 1000L);
    // set space quota, 单位 B
    dfsadmin.setSpaceQuota(path，1024);
    
    // 清除限制
    dfsadmin.clearQuota(path);
    dfsadmin.clearSpaceQuota(path);
    
    // 获取指定路径下配额聚合根
    QuotaUsage quota = fs.getQuotaUsage(path);
    // 查询空间配额大小
    long spaceQuota = quota.getSpaceQuota();
    // 查询已使用大小
    long used = quota.getSpaceConsumed();
    // 查询HDFS磁盘总容量
    DistributedFileSystem dfs = (DistributedFileSystem )fs;
    long capacity = dfs.getStatus().getCapacity();
    
}
```