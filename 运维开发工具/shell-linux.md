# 文件传输

## scp

功能说明：文件传输

常用指令

```shell
scp root@localhost:/opt/file [本地路径]		复制远程服务器文件 ——> 本地
scp file root@192.168.226.121:/opt			 复制本地文件 ——> 远程服务器
```

# 文件管理

## mkdir

```shell
mkdir -p /home/xiao # -p 递归创建文件夹
```

## cat

常用指令

```shell
cat >> file.text <<EOF 内容  EOF	 # 往file.text追加内容
```

## find

功能说明：查找文件

常用指令

```shell
find path -name "fd"	# 查找path路径下的 "fd" 文件
```

## grep

功能说明：查找文件里面符合条件的字符串

常用指令

```shell
grep "password" /usr/mysql/text.text  查找该文件下符合 “password” 行数
```

## chmod

功能说明 change mode：变更文件或目录的权限。

```shell
	  在UNIX系统家族里，文件或目录权限的控制分别以读取，写入，执行3种一般权限来区分，另有3种特殊权限可供运用，再搭配拥有者与所属群组管理权限范围。您可以使用chmod指令去变更文件与目录的权限，设置方式采用文字或数字代号皆可。符号连接的权限无法变更，如果您对符号连接修改权限，其改变会作用在被连接的原始文件。
    
  文件属性：
  -：普通文件
  d：目录文件
  l：链接文件
  p：管道文件
  
  权限范围表示： 
　u：User，即文件或目录的拥有者。 
　g：Group，即文件或目录的所属群组。 
　o：Other，除了文件或目录拥有者或所属群组之外，其他用户皆属于这个范围。 
　a：All，即全部的用户，包含拥有者，所属群组以及其他用户。 
 
　权限对照表： 
　r：读取权限，数字代号为"4"。 
　w：写入权限，数字代号为"2"。 
　x：执行或切换权限，数字代号为"1"。 
　-：不具任何权限，数字代号为"0"。 
　s：特殊?b>功能说明：变更文件或目录的权限。
 
 	数字对照表：
  7：rwx  	-可读可写可执行
  6：rw-		-读写
  5：r-x		-读执行
  4：r--		-只读
  3：-wx		-写执行
  2：-w-		-只写
  1：--x		-只执行
  0：---		-无权限
```

常用指令

```shell
chmod -R 755 [文件或目录]	递归修改权限为用户全权限，用户组和其他用户为读执行。 一般权限应该设为这个
```

语法

```shell
chmod [-cfRv][--help][--version][<权限范围>+/-/=<权限设置...>][文件或目录...] 或 
chmod [-cfRv][--help][--version][数字代号][文件或目录...] 或 
chmod [-cfRv][--help][--reference=<参考文件或目录>][--version][文件或目录...] 
```

参数说明

```shell
-c或--changes 　效果类似"-v"参数，但仅回报更改的部分。 
-f或--quiet或--silent 　不显示错误信息。 
-R或--recursive 　递归处理，将指定目录下的所有文件及子目录一并处理。 
-v或--verbose 　显示指令执行过程。 
--help 　在线帮助。 
--reference=<参考文件或目录> 　把指定文件或目录的权限全部设成和参考文件或目录的权限相同 
--version 　显示版本信息。 
<权限范围>+<权限设置> 　开启权限范围的文件或目录的该项权限设置。 
<权限范围>-<权限设置> 　关闭权限范围的文件或目录的该项权限设置。 
<权限范围>=<权限设置> 　指定权限范围的文件或目录的该项权限设置。 
```

# 磁盘管理

## df

功能说明(disk free):  显示磁盘的相关信息

常用命令

```shell
df -h [文件目录]   递归显示该目录下所有文件大小
df -h -s [文件目录]  仅显示该目录的总大小
```

语法

```shell
df [-ahHiklmPT][--block-size=<区块大小>][-t <文件系统类型>][-x <文件系统类型>][--help][--no-sync][--sync][--version][文件或设备]
df -h [文件目录]  
```

参数说明

```shell
-a或--all   包含全部的文件系统。
--block-size=<区块大小>   以指定的区块大小来显示区块数目。
-h或--human-readable   以可读性较高的方式来显示信息。
-H或--si   与-h参数相同，但在计算时是以1000 Bytes为换算单位而非1024 Bytes。
-i或--inodes   显示inode的信息。
-k或--kilobytes   指定区块大小为1024字节。
-l或--local   仅显示本地端的文件系统。
-m或--megabytes   指定区块大小为1048576字节。
--no-sync   在取得磁盘使用信息前，不要执行sync指令，此为预设值。
-P或--portability   使用POSIX的输出格式。
--sync   在取得磁盘使用信息前，先执行sync指令。
-t<文件系统类型>或--type=<文件系统类型>   仅显示指定文件系统类型的磁盘信息。
-T或--print-type   显示文件系统的类型。
-x<文件系统类型>或--exclude-type=<文件系统类型>   不要显示指定文件系统类型的磁盘信息。
--help   显示帮助。
--version   显示版本信息。
[文件或设备]   指定磁盘设备。
```



## du

功能说明(disk usage):  显示目录或文件的大小

常用指令

```shell
du -h [文件目录]   递归显示该目录下所有文件大小
du -hs [文件目录]  仅显示该目录的总大小
```

语法

```shell
du [-abcDhHklmsSx][-L <符号连接>][-X <文件>][--block-size][--exclude=<目录或文件>][--max-depth=<目录层数>][--help][--version][目录或文件]
```

参数说明

```shell
-a或-all   显示目录中个别文件的大小。
-b或-bytes   显示目录或文件大小时，以byte为单位。
-c或--total   除了显示个别目录或文件的大小外，同时也显示所有目录或文件的总和。
-D或--dereference-args   显示指定符号连接的源文件大小。
-h或--human-readable   以K，M，G为单位，提高信息的可读性。
-H或--si   与-h参数相同，但是K，M，G是以1000为换算单位。
-k或--kilobytes   以1024 bytes为单位。
-l或--count-links   重复计算硬件连接的文件。
-L<符号连接>或--dereference<符号连接>   显示选项中所指定符号连接的源文件大小。
-m或--megabytes   以1MB为单位。
-s或--summarize   仅显示总计。
-S或--separate-dirs   显示个别目录的大小时，并不含其子目录的大小。
-x或--one-file-xystem   以一开始处理时的文件系统为准，若遇上其它不同的文件系统目录则略过。
-X<文件>或--exclude-from=<文件>   在<文件>指定目录或文件。
--exclude=<目录或文件>   略过指定的目录或文件。
--max-depth=<目录层数>   超过指定层数的目录后，予以忽略。
--help   显示帮助。
--version   显示版本信息。
```

# 网络通讯

## firewall-cmd

常用指令

```shell
firewall-cmd --zone=public --add-port=5672/tcp --permanent   # 开放5672端口
firewall-cmd --zone=public --remove-port=5672/tcp --permanent  #关闭5672端口
firewall-cmd --reload   # 配置立即生效
firewall-cmd --zone=public --list-ports # 查看防火墙所有开放的端口
firewall-cmd --query-port=8080/tcp  # 查看8080端口是否开放

systemctl [start|stop|restart] firewalld.service  #启动|关闭|重新启动 防火墙
```

## netstat

功能说明：显示网络状态

常用指令

```shell
netstat -tnpl  查看 tcp ip socket 进程
netstat -tnpl|grep [端口号]  查看该socket的监听状态
```

语法

```shell
netstat [-acCeFghilMnNoprstuvVwx][-A<网络类型>][--ip]
```

参数说明

```shell
-a或--all   显示所有连线中的Socket。
-A<网络类型>或--<网络类型>   列出该网络类型连线中的相关地址。
-c或--continuous   持续列出网络状态。
-C或--cache   显示路由器配置的快取信息。
-e或--extend   显示网络其他相关信息。
-F或--fib   显示FIB。
-g或--groups   显示多重广播功能群组组员名单。
-h或--help   在线帮助。
-i或--interfaces   显示网络界面信息表单。
-l或--listening   显示监控中的服务器的Socket。
-M或--masquerade   显示伪装的网络连线。
-n或--numeric   直接使用IP地址，而不通过域名服务器。
-N或--netlink或--symbolic   显示网络硬件外围设备的符号连接名称。
-o或--timers   显示计时器。
-p或--programs   显示正在使用Socket的程序识别码和程序名称。
-r或--route   显示Routing Table。
-s或--statistice   显示网络工作信息统计表。
-t或--tcp   显示TCP传输协议的连线状况。
-u或--udp   显示UDP传输协议的连线状况。
-v或--verbose   显示指令执行过程。
-V或--version   显示版本信息。
-w或--raw   显示RAW传输协议的连线状况。
-x或--unix   此参数的效果和指定"-A unix"参数相同。
--ip或--inet   此参数的效果和指定"-A inet"参数相同。
```

# 系统管理

## date

功能说明： 显示或者设置系统时间

常用指令

```shell
date -R   查看时区
date -u		查看UTC
date  		查看时间
```

语法

```shell
date [-d <字符串>][-u][+%H%I%K%l%M%P%r%s%S%T%X%Z%a%A%b%B%c%d%D%j%m%U%w%x%y%Y%n%t] 或date [-s <字符串>][-u][MMDDhhmmCCYYss] 或 date [--help][--version]
```

参数说明

```shell
%H 　小时(以00-23来表示)。 
%I 　小时(以01-12来表示)。 
%K 　小时(以0-23来表示)。 
%l 　小时(以0-12来表示)。 
%M 　分钟(以00-59来表示)。 
%P 　AM或PM。 
%r 　时间(含时分秒，小时以12小时AM/PM来表示)。 
%s 　总秒数。起算时间为1970-01-01 00:00:00 UTC。 
%S 　秒(以本地的惯用法来表示)。 
%T 　时间(含时分秒，小时以24小时制来表示)。 
%X 　时间(以本地的惯用法来表示)。 
%Z 　市区。 
%a 　星期的缩写。 
%A 　星期的完整名称。 
%b 　月份英文名的缩写。 
%B 　月份的完整英文名称。 
%c 　日期与时间。只输入date指令也会显示同样的结果。 
%d 　日期(以01-31来表示)。 
%D 　日期(含年月日)。 
%j 　该年中的第几天。 
%m 　月份(以01-12来表示)。 
%U 　该年中的周数。 
%w 　该周的天数，0代表周日，1代表周一，异词类推。 
%x 　日期(以本地的惯用法来表示)。 
%y 　年份(以00-99来表示)。 
%Y 　年份(以四位数来表示)。 
%n 　在显示时，插入新的一行。 
%t 　在显示时，插入tab。 
MM 　月份(必要)。 
DD 　日期(必要)。 
hh 　小时(必要)。 
mm 　分钟(必要)。 
CC 　年份的前两位数(选择性)。 
YY 　年份的后两位数(选择性)。 
ss 　秒(选择性)。 
-d<字符串> 　显示字符串所指的日期与时间。字符串前后必须加上双引号。 
-s<字符串> 　根据字符串来设置日期与时间。字符串前后必须加上双引号。 
-u 　显示GMT。 
--help 　在线帮助。 
--version 　显示版本信息。
```

## hwclock

功能说明hardware clock：显示与设定硬件时钟

```shell
在Linux中有硬件时钟与系统时钟等两种时钟。硬件时钟是指主机板上的时钟设备，也就是通常可在BIOS画面设定的时钟。系统时钟则是指kernel中的时钟。当Linux启动时，系统时钟会去读取硬件时钟的设定，之后系统时钟即独立运作。所有Linux相关指令与函数都是读取系统时钟的设定。
```

常用指令

```shell
hwclock	--show		和hwclock一样的效果，显示当前硬件时间
hwclock -w				根据系统时间设置硬件时间
```

语法

```shell
语　　法：hwclock [--adjust][--debug][--directisa][--hctosys][--show][--systohc][--test]
[--utc][--version][--set --date=<日期与时间>]
```

参数说明

```shell
--adjust 　hwclock每次更改硬件时钟时，都会记录在/etc/adjtime文件中。使用--adjust参数，可使hwclock根据先前的记录来估算硬件时钟的偏差，并用来校正目前的硬件时钟。 
--debug 　显示hwclock执行时详细的信息。 
--directisa 　hwclock预设从/dev/rtc设备来存取硬件时钟。若无法存取时，可用此参数直接以I/O指令来存取硬件时钟。 
--hctosys 　将系统时钟调整为与目前的硬件时钟一致。 
--set --date=<日期与时间> 　设定硬件时钟。 
--show 　显示硬件时钟的时间与日期。 
--systohc 　将硬件时钟调整为与目前的系统时钟一致。 
--test 　仅测试程序，而不会实际更改硬件时钟。 
--utc 　若要使用格林威治时间，请加入此参数，hwclock会执行转换的工作。 
--version 　显示版本信息。
```

## init

常用指令

```shell
init 0		关机
init 6 		重启
```

# 命令行快捷键

```shell
# 快速清除命令行
Ctrl + a & Ctrl + k
Ctrl + y - 复制

Ctrl + l - 清屏
Ctrl + a - 光标移到行首
Ctrl + e - 光标移到行尾
Ctrl + w - 清除光标之前一个单词
Ctrl + k - 清除光标到行尾的字符
Ctrl + t - 交换光标前两个字符

Ctrl + v - 输入控制字符 如Ctrl+v ,会输入^M
Ctrl + f - 光标后移一个字符
Ctrl + b - 光标前移一个字符
Ctrl + h - 删除光标前一个字符 
```