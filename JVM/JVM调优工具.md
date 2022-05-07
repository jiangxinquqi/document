#  jps

```shell
# jps 输出进程号 主类名
# jps -m 输出进程号 主类名
# jps -l 输出进程号 主类全限定名
# jps -v 输出进程号 主类名 JVM参数列表
```

# Visual  VM

- 配置 jstatd

```shell
1）配置安全策略
# vim $JAVA_HOME/jre/lib/security/java.policy

# 文件末尾追加
permission java.security.AllPermission; 

2）启动 jstatd（启动后会开启注册端口1099和一个随机的连接端口，注册端口也可通过-p参数指定）
# $JAVA_HOME/bin/jstatd -J-Djava.security.policy=all.policy -p 10003 &

3）设置防火墙 将jstatd监控的端口开放
# netstat -anp | grep *jstatd

4）远程连接
```

# jar

## 修改jar包内部资源

```shell
jar -xf xxx.jar # 解压缩jar文件
jar -cvfm0 xxx.jar META-INF/MANIFEST.MF ./ # 重新打包，解压后重新打包的时候需要删除原来的jar包。
```