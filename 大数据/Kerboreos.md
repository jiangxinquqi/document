# 安装KDC

```shell
yum -y install openldap-clients krb5-workstation krb5-libs krb5-server krb5-auth-dialog
```

# 修改/etc/krb5.conf

```shell
vim /etc/krb5.conf

# Configuration snippets may be placed in this directory as well
includedir /etc/krb5.conf.d/

[logging]
 default = FILE:/var/log/krb5libs.log
 kdc = FILE:/var/log/krb5kdc.log
 admin_server = FILE:/var/log/kadmind.log

[libdefaults]
 dns_lookup_realm = false
 ticket_lifetime = 24h
 renew_lifetime = 7d
 forwardable = true
 rdns = false
# pkinit_anchors = FILE:/etc/pki/tls/certs/ca-bundle.crt
 default_realm = CLOUD.COM
# default_ccache_name = KEYRING:persistent:%{uid}

[realms]
 CLOUD.COM = {
   kdc = cdp01.cloud.com
   admin_server = cdp01.cloud.com
 }

[domain_realm]
 .cloud.com = CLOUD.COM
 cloud.com = CLOUD.COM
```

# 修改 /var/kerberos/krb5kdc/kadm5.acl

```shell
vim /var/kerberos/krb5kdc/kadm5.acl

*/admin@CLOUD.COM       *
```

# 修改/var/kerberos/krb5kdc/kdc.conf

```shell
vim /var/kerberos/krb5kdc/kdc.conf

[kdcdefaults]
 kdc_ports = 88
 kdc_tcp_ports = 88

[realms]
 CLOUD.COM = {
  #master_key_type = aes256-cts
  max_renewable_life = 7d 0h 0m 0s
  acl_file = /var/kerberos/krb5kdc/kadm5.acl
  dict_file = /usr/share/dict/words
  admin_keytab = /var/kerberos/krb5kdc/kadm5.keytab
  supported_enctypes = aes256-cts:normal aes128-cts:normal des3-hmac-sha1:normal arcfour-hmac:normal camellia256-cts:normal camellia128-cts:normal des-hmac-sha1:normal des-cbc-md5:normal des-cbc-crc:normal
 }
```

#  生成master服务器上的kdc database

```shell
kdb5_util create -s -r HADOOP.COM
# 保存路径为/var/kerberos/krb5kdc 如果需要重建数据库，将该目录下的principal相关的文件删除即可
在此过程中，我们会输入database的管理密码。这里设置的密码一定要记住，如果忘记了，就无法管理Kerberos server。
当Kerberos database创建好后，可以看到目录 /var/kerberos/krb5kdc 下生成了几个文件：
[root@masternode01 ~]# ls /var/kerberos/krb5kdc
hdfs.keytab  kadm5.acl  kdc.conf.m1  kdc.conf.m3  kdc.conf.orig  principal.kadm5       principal.ok
HTTP.keytab  kdc.conf   kdc.conf.m2  kdc.conf.m4  principal      principal.kadm5.lock
若要从新初始化数据库则需要删除　
rm -rf /var/kerberos/krb5kdc/principal*
之后在重新初始化数据即可　kdb5_util create -s -r CLOUD.COM
```

#  在maste服务器上创建admin用户  

```shell
在maste KDC上执行：
kadmin.local -q "addprinc admin/admin"
在KDC上我们需要编辑acl文件来设置权限，该acl文件的默认路径是 /var/kerberos/krb5kdc/kadm5.acl（也可以在文件kdc.conf中修改）。Kerberos的kadmind daemon会使用该文件来管理对Kerberos database的访问权限。对于那些可能会对pincipal产生影响的操作，acl文件也能控制哪些principal能操作哪些其他pricipals。
```

#  在master KDC启动Kerberos daemons  

```shell
[root@masternode01 jce]# service krb5kdc start
[root@masternode01 jce]# service kadmin start
设置开机自动启动：
[root@masternode01 jce]# chkconfig krb5kdc on
[root@masternode01 jce]# chkconfig kadmin on
```

# 其他主机安装client

```shell
yum install openldap-clients krb5-workstation krb5-libs
for i in {2,3,4} ;do scp /etc/krb5.conf root@cdp0$i:/etc/;done
```

# 测试管理员账号

```shell
kinit admin/admin@CLOUD.COM
```

# Cloudera Manager 安装Kerberos 

# 浏览器Web UI认证  

参考： https://mp.weixin.qq.com/s?__biz=MzI4OTY3MTUyNg==&mid=2247483853&idx=1&sn=442a8ba87c922857253a437affe42506&chksm=ec2ad1c4db5d58d2933ae5cde4ab1a7443c944e94aca85b51cbd8e9f4f3772162a39074da49d&scene=21#wechat_redirect

1. 安装Firefox浏览器
2. 安装MIT Kerberos Windows客户端

```shell
下载地址
https://web.mit.edu/kerberos/
```

3. 将KDC Server服务器上/ect/krb5.conf文件中的部分内容，拷贝到krb5.ini文件中 
4. 配置Firefox的高级选项，地址栏输入**about:config**

```shell
# 置需要访问的主机名, 多个域名以逗号分隔
network.negotiate-auth.trusted-uris = cdp01.cloud.com,cdp02.cloud.com
network.auth.use-sspi = false
```

5. 进行Kerberos认证，认证成功后打开对应的WebUI页面。

   ```
   find /var/run/cloudera-scm-agent -name "*-hdfs-NAMENODE"
   ```

   CDP keytab文件下载：CM节点的/var/run/cloudera-scm-agent /xxx-hdfs-NAMENODE/hdfs.keytab

6. 常用指令

   ```
   ## KDC Server
   sudo kadmin.local
   > addprinc test@CLOUD.COM # 新增用户
   > xst -norandkey -k test.keytab test@CLOUD.COM # 创建该账号的keytab文件，-norandkey必须加，不然会认证失败。
   > listprincs hdfs* # 查看匹配hdfs用户
   
   ## Client
   kdestroy # 销毁Ticket
   kinit admin/admin@CLOUD.COM # 账号密码认证
   kinit -kt test.keytab test@CLOUD.COM # keytab文件认证
   klist # 查看当前认证用户
   
   
   ```

   