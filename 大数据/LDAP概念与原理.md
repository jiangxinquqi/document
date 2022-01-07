目录服务就是按照树状存储信息的模式。目录服务的数据类型主要是字符型, 而不是关系数据库提供的整数、浮点数、日期、货币等类型。为了检索的需要添加了BIN（二进制数据）、CIS（忽略大小写）、CES（大小写敏感）、TEL（电话型）等语法（Syntax）



同样也不提供象关系数据库中普遍包含的大量的函数



目录有很强的查询（读）功能，适合于进行大量数据的检索



但目录一般只执行简单的更新（写）操作，不支持批量更新所需要的事务处理功能



它主要面向数据的查询服务（查询和修改操作比一般是大于10:1），不提供事务的回滚（rollback）机制.



目录具有广泛复制信息的能力，适合于多个目录服务器同步/更新.



LDAP是轻量目录访问协议(Lightweight Directory Access Protocol)的缩写



LDAP标准实际上是在X.500标准基础上产生的一个简化版本



LDAP的结构用树来表示，而不是用表格。正因为这样，就不能用SQL语句了



LDAP可以很快地得到查询结果，不过在写方面，就慢得多



LDAP提供了静态数据的快速查询方式



Client/server模型



Server 用于存储数据



Client提供操作目录信息树的工具



这些工具可以将数据库的内容以文本格式（LDAP 数据交换格式，LDIF）呈现在您的面前



LDAP目录数据结构



在LDAP中目录是按照树型结构组织——目录信息树（DIT）



DIT是一个主要进行读操作的数据库



DIT由条目（Entry）组成，条目相当于关系数据库中表的记录；



条目是具有分辨名DN（DistinguishedName）的属性-值对（Attribute-value,简称AV）的集合.



LDIF是LDAP数据库信息的一种文本格式。



1. LDAP简介



LDAP（轻量级目录访问协议，Lightweight Directory Access Protocol)是实现提供被称为目录服务的信息服务。目录服务是一种特殊的数据库系统，其专门针对读取，浏览和搜索操作进行了特定的优化。目录一般用来包含描述性的，基于属性的信息并支持精细复杂的过滤能力。目录一般不支持通用数据库针对大量更新操作操作需要的复杂的事务管理或回卷策略。而目录服务的更新则一般都非常简单。这种目录可以存储包括个人信息、web链结、jpeg图像等各种信息。为了访问存储在目录中的信息，就需要使用运行在TCP/IP 之上的访问协议—LDAP。



LDAP目录中的信息是是按照树型结构组织，具体信息存储在条目(entry)的数据结构中。条目相当于关系数据库中表的记录；条目是具有区别名DN （Distinguished Name）的属性（Attribute），DN是用来引用条目的，DN相当于关系数据库表中的关键字（Primary Key）。属性由类型（Type）和一个或多个值（Values）组成，相当于关系数据库中的字段（Field）由字段名和数据类型组成，只是为了方便检索的需要，LDAP中的Type可以有多个Value，而不是关系数据库中为降低数据的冗余性要求实现的各个域必须是不相关的。LDAP中条目的组织一般按照地理位置和组织关系进行组织，非常的直观。LDAP把数据存放在文件中，为提高效率可以使用基于索引的文件数据库，而不是关系数据库。类型的一个例子就是mail，其值将是一个电子邮件地址。



LDAP的信息是以树型结构存储的，在树根一般定义国家(c=CN)或域名(dc=com)，在其下则往往定义一个或多个组织 (organization)(o=Acme)或组织单元(organizational units) (ou=People)。一个组织单元可能包含诸如所有雇员、大楼内的所有打印机等信息。此外，LDAP支持对条目能够和必须支持哪些属性进行控制，这是有一个特殊的称为对象类别(objectClass)的属性来实现的。该属性的值决定了该条目必须遵循的一些规则，其规定了该条目能够及至少应该包含哪些属性。例如：inetorgPerson对象类需要支持sn(surname)和cn(common name)属性，但也可以包含可选的如邮件，电话号码等属性。



1. LDAP简称对应



o– organization（组织-公司）



ou – organization unit（组织单元-部门）



c - countryName（国家）



dc - domainComponent（域名）



sn – suer name（真实名称）



cn - common name（常用名称）



1. 目录设计



设计目录结构是LDAP最重要的方面之一。下面我们将通过一个简单的例子来说明如何设计合理的目录结构。该例子将通过Netscape地址薄来访文。假设有一个位于美国US(c=US)而且跨越多个州的名为Acme(o=Acme)的公司。Acme希望为所有的雇员实现一个小型的地址薄服务器。



我们从一个简单的组织DN开始：



dn: o=Acme, c=US



Acme所有的组织分类和属性将存储在该DN之下，这个DN在该存储在该服务器的目录是唯一的。Acme希望将其雇员的信息分为两类：管理者(ou= Managers)和普通雇员(ou=Employees),这种分类产生的相对区别名(RDN,relative distinguished names。表示相对于顶点DN)就shi ：



dn: ou=Managers, o=Acme, c=US



dn: ou=Employees, o=Acme, c=US



在下面我们将会看到分层结构的组成：顶点是US的Acme，下面是管理者组织单元和雇员组织单元。因此包括Managers和Employees的DN组成为：



dn: cn=Jason H. Smith, ou=Managers, o=Acme, c=US



dn: cn=Ray D. Jones, ou=Employees, o=Acme, c=US



dn: cn=Eric S. Woods, ou=Employees, o=Acme, c=US



为了引用Jason H. Smith的通用名(common name )条目，LDAP将采用cn=Jason H. Smith的RDN。然后将前面的父条目结合在一起就形成如下的树型结构：



cn=Jason H. Smith



-  ou=Managers 
-  o=Acme 

-  c=US 



-> dn: cn=Jason H. Smith,ou=Managers,o=Acme,c=US



现在已经定义好了目录结构，下一步就需要导入目录信息数据。目录信息数据将被存放在LDIF文件中，其是导入目录信息数据的默认存放文件。用户可以方便的编写Perl脚本来从例如/etc/passwd、NIS等系统文件中自动创建LDIF文件。



下面的实例保存目录信息数据为testdate.ldif文件，该文件的格式说明将可以在man ldif中得到。



在添加任何组织单元以前，必须首先定义Acme DN：



dn: o=Acme, c=US



objectClass: organization



这里o属性是必须的



o: Acme



下面是管理组单元的DN，在添加任何管理者信息以前，必须先定义该条目。



dn: ou=Managers, o=Acme, c=US



objectClass: organizationalUnit



这里ou属性是必须的。



ou: Managers



第一个管理者DN：



dn: cn=Jason H. Smith, ou=Managers, o=Acme, c=US



objectClass: inetOrgPerson



cn和sn都是必须的属性：



cn: Jason H. Smith



sn: Smith



但是还可以定义一些可选的属性：



telephoneNumber: 111-222-9999



mail: [headhauncho@acme.com](mailto:headhauncho@acme.com)



localityName:Houston



可以定义另外一个组织单元：



dn: ou=Employees, o=Acme, c=US



objectClass: organizationalUnit



ou: Employees



并添加雇员信息如下：



dn: cn=Ray D. Jones, ou=Employees, o=Acme, c=US



objectClass: inetOrgPerson



cn: Ray D. Jones



sn: Jones



telephoneNumber: 444-555-6767



mail: [jonesrd@acme.com](mailto:jonesrd@acme.com)



localityName:Houston



dn: cn=Eric S. Woods, ou=Employees, o=Acme, c=US



objectClass: inetOrgPerson



cn: Eric S. Woods



sn: Woods



telephoneNumber: 444-555-6768



mail: [woodses@acme.com](mailto:woodses@acme.com)



localityName:Houston