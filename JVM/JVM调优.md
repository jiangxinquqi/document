# 参数详解

JVM参数主要分为以下三种（可以根据书写形式来区分）：

## 1、标准参数

标准参数，顾名思义，标准参数中包括功能以及输出的结果都是很稳定的，基本上不会随着JVM版本的变化而变化。

我们可以通过`java -help `命令来检索出所有标准参数。

```she
用法: java [-options] class [args...]
           (执行类)
   或  java [-options] -jar jarfile [args...]
           (执行 jar 文件)
其中选项包括:
    -d32          使用 32 位数据模型 (如果可用)
    -d64          使用 64 位数据模型 (如果可用)
    -server       选择 "server" VM
                  默认 VM 是 server.

    -cp <目录和 zip/jar 文件的类搜索路径>
    -classpath <目录和 zip/jar 文件的类搜索路径>
                  用 ; 分隔的目录, JAR 档案
                  和 ZIP 档案列表, 用于搜索类文件。
    -D<名称>=<值>
                  设置系统属性
    -verbose:[class|gc|jni]
                  启用详细输出
    -version      输出产品版本并退出
    -version:<值>
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  需要指定的版本才能运行
    -showversion  输出产品版本并继续
    -jre-restrict-search | -no-jre-restrict-search
                  警告: 此功能已过时, 将在
                  未来发行版中删除。
                  在版本搜索中包括/排除用户专用 JRE
    -? -help      输出此帮助消息
    -X            输出非标准选项的帮助
    -ea[:<packagename>...|:<classname>]
    -enableassertions[:<packagename>...|:<classname>]
                  按指定的粒度启用断言
    -da[:<packagename>...|:<classname>]
    -disableassertions[:<packagename>...|:<classname>]
                  禁用具有指定粒度的断言
    -esa | -enablesystemassertions
                  启用系统断言
    -dsa | -disablesystemassertions
                  禁用系统断言
    -agentlib:<libname>[=<选项>]
                  加载本机代理库 <libname>, 例如 -agentlib:hprof
                  另请参阅 -agentlib:jdwp=help 和 -agentlib:hprof=help
    -agentpath:<pathname>[=<选项>]
                  按完整路径名加载本机代理库
    -javaagent:<jarpath>[=<选项>]
                  加载 Java 编程语言代理, 请参阅 java.lang.instrument
    -splash:<imagepath>
                  使用指定的图像显示启动屏幕
```

## 2、X 参数

对应前面讲的标准化参数，这是非标准化参数。表示在将来的JVM版本中可能会发生改变，但是这类以 -X开始的参数变化的比较小。

我们可以通过 `java -X` 命令来检索所有-X 参数。

```shell
    -Xmixed           混合模式执行（默认）
    -Xint             仅解释模式执行
    -Xbootclasspath:<用 ; 分隔的目录和 zip/jar 文件>
                      设置引导类和资源的搜索路径
    -Xbootclasspath/a:<用 ; 分隔的目录和 zip/jar 文件>
                      附加在引导类路径末尾
    -Xbootclasspath/p:<用 ; 分隔的目录和 zip/jar 文件>
                      置于引导类路径之前
    -Xdiag            显示附加诊断消息
    -Xnoclassgc        禁用类垃圾收集
    -Xincgc           启用增量垃圾收集
    -Xloggc:<file>    将 GC 状态记录在文件中（带时间戳）
    -Xbatch           禁用后台编译
    -Xms<size>        设置初始 Java 堆大小
    -Xmx<size>        设置最大 Java 堆大小
    -Xss<size>        设置 Java 线程堆栈大小
    -Xprof            输出 cpu 分析数据
    -Xfuture          启用最严格的检查，预计会成为将来的默认值
    -Xrs              减少 Java/VM 对操作系统信号的使用（请参阅文档）
    -Xcheck:jni       对 JNI 函数执行其他检查
    -Xshare:off       不尝试使用共享类数据
    -Xshare:auto      在可能的情况下使用共享类数据（默认）
    -Xshare:on        要求使用共享类数据，否则将失败。
    -XshowSettings    显示所有设置并继续
    -XshowSettings:system
                      （仅限 Linux）显示系统或容器
                      配置并继续
    -XshowSettings:all
                      显示所有设置并继续
    -XshowSettings:vm 显示所有与 vm 相关的设置并继续
    -XshowSettings:properties
                      显示所有属性设置并继续
    -XshowSettings:locale
                      显示所有与区域设置相关的设置并继续

-X 选项是非标准选项。如有更改，恕不另行通知。
```

## 3、XX参数

这是我们日常开发中接触到最多的参数类型。这也是非标准化参数，相对来说不稳定，随着JVM版本的变化可能会发生变化，主要用于JVM调优和debug。

注意：这种参数是我们后续介绍JVM调优讲解最多的参数。

该参数的书写形式又分为两大类：

①、Boolean类型 格式：-XX:[+-]表示启用或者禁用name属性。

例子：-XX:+UseG1GC（表示启用G1垃圾收集器）

②、Key-Value类型 格式：-XX:=表示name的属性值为value。

例子：-XX:MaxGCPauseMillis=500（表示设置GC的最大停顿时间是500ms）

## 4、参数详解

1、打印已经被用户或者当前虚拟机设置过的参数

```shell
java -XX:+PrintCommandLineFlags
```

2、最大堆和最小堆内存设置

-Xms512M：设置堆内存初始值为512M

-Xmx1024M：设置堆内存最大值为1024M

这里的ms是memory start的简称，mx是memory max的简称，分别代表最小堆容量和最大堆容量。但是别看这里是-X参数，其实这是-XX参数，等价于：

-XX:InitialHeapSize

-XX:MaxHeapSize

在通常情况下，服务器项目在运行过程中，堆空间会不断的收缩与扩张，势必会造成不必要的系统压力。所以在生产环境中，JVM的Xms和Xmx要设置成一样的，能够避免GC在调整堆大小带来的不必要的压力。

3、Dump异常快照以及以文件形式导出

-XX:+HeapDumpOnOutOfMemoryError

-XX:HeapDumpPath

堆内存出现OOM的概率是所有内存耗尽异常中最高的，出错时的堆内信息对解决问题非常有帮助，所以给JVM设置这个参数(-XX:+HeapDumpOnOutOfMemoryError)，让JVM遇到OOM异常时能输出堆内信息，并通过（-XX:+HeapDumpPath）参数设置堆内存溢出快照输出的文件地址，这对于特别是对相隔数月才出现的OOM异常尤为重要。

这两个参数通常配套使用：

```shell
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=./
```

4、发送OOM后，执行一个脚本

-XX:OnOutOfMemoryError

比如这样设置：

```shell
-XX:OnOutOfMemoryError="C:\Program Files\Java\jdk1.8.0_152\bin\jconsole.exe"
```

表示发生OOM后，运行jconsole.exe程序。这里可以不用加“”，因为jconsole.exe路径Program Files含有空格。

利用这个参数，我们可以在系统OOM后，自定义一个脚本，可以用来发送邮件告警信息，可以用来重启系统等等。

5、打印gc信息

①、打印GC简单信息

-verbose:gc

-XX:+PrintGC

一个是标准参数，一个是-XX参数，都是打印详细的gc信息。

②、打印详细GC信息

-XX:+PrintGCDetails

-XX:+PrintGCTimeStamps

6、指定GC日志以文件输出

-Xloggc:./gc.log

这个在参数用于将gc日志以文件的形式输出，更方便我们去查看日志，定位问题。

7、设置永久代大小

-XX:MaxPermSize=1280m

在JDK1.7以及以前的版本中，只有Hotspot 才有Perm区，称为永久代，它在启动时固定大小，很难进行调优。

在某些情况下，如果动态加载类过多，容易产生Perm区的 OOM。比如某个实际 Web 工程中，因为功能点较多，在运行过程中，要不断动态加载很多类，就会出现类似错误：

"Exception in thread 'dubbo client x.x.connect' java.lang.OutOfMemoryError:PermGenspace"

为了解决这个问题，就需要在项目启动时，设定运行参数-XX:MaxPermSize。

注意：在JDK1.8以后面的版本，使用元空间来代替永久代。在 JDK1.8以及后面的版本中，如果设定参数-XX:MaxPermSize，启动JVM不会报错，但是会提示：

Java Hotspot 64Bit Server VM warning：ignoring option MaxPermSize=1280m:support was removed in 8.0

![GC](assets/GC-1642057159107.jpg)

# 最佳实践

## java.lang.OutOfMemoryError: Metaspace 异常解决

java8 及以后的版本使用Metaspace来代替永久代，Metaspace是方法区在HotSpot中的实现，它与持久代最大区别在于，Metaspace并不在虚拟机内存中而是使用本地内存也就是在JDK8中,classe metadata(the virtual machines internal presentation of Java class),被存储在叫做Metaspace的native memory.
永久代（java 8 后被元空间Metaspace取代了）存放了以下信息：

1. 虚拟机加载的类信息
2. 常量池
3. 静态变量
4. 即时编译后的代码

出现问题原因

错误的主要原因, 是加载到内存中的 class 数量太多或者体积太大。
解决办法：    增加 Metaspace 的大小
```java
-XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=512m
```
查看元空间大小
```java
java -XX:+PrintFlagsInitial
```

代码演示

```java
import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;

import java.lang.reflect.Method;

public class MetaspaceDemo {
    static class OOM{}
    public static void main(String[] args) {
        int i = 0;//模拟计数多少次以后发生异常
        try {
            while (true){
                i++;
                Enhancer enhancer = new Enhancer();
                enhancer.setSuperclass(OOM.class);
                enhancer.setUseCache(false);
                enhancer.setCallback(new MethodInterceptor() {
                    @Override
                    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                        return methodProxy.invokeSuper(o,args);
                    }
                });
                enhancer.create();
            }
        } catch (Throwable e) {
            System.out.println("=================多少次后发生异常："+i);
            e.printStackTrace();
        }
    }
}
```

