# tcpdump

tcpdump可以截获网络中的packet并进行分析。支持复杂的过滤功能。

## 选项

通过`tcpdump --help`可以查看tcpdump的命令格式。

```shell
tcpdump version 4.99.1
libpcap version 1.10.1 (with TPACKET_V3)
OpenSSL 3.0.2 15 Mar 2022
Usage: tcpdump [-AbdDefhHIJKlLnNOpqStuUvxX#] [ -B size ] [ -c count ] [--count]
		[ -C file_size ] [ -E algo:secret ] [ -F file ] [ -G seconds ]
		[ -i interface ] [ --immediate-mode ] [ -j tstamptype ]
		[ -M secret ] [ --number ] [ --print ] [ -Q in|out|inout ]
		[ -r file ] [ -s snaplen ] [ -T type ] [ --version ]
		[ -V file ] [ -w file ] [ -W filecount ] [ -y datalinktype ]
		[ --time-stamp-precision precision ] [ --micro ] [ --nano ]
		[ -z postrotate-command ] [ -Z user ] [ expression ]

```

+ -c ：捕获指定数量的packet后退出。
+ -D or --list-interfaces ：列出操作系统上可用的网络接口，在这些接口上tcpdump可以捕获packet。

~~~shell
$ tcpdump --list-interfaces
1.wlo1 [Up，Running，Wireless，Associated]
2.any (Pseudo-device that captures on all interfaces) [Up，Running]
3.lo [Up，Running，Loopback]
4.enp3s0 [Up，Disconnected]
5.bluetooth0 (Bluetooth adapter number 0) [Wireless，Association status unknown]
6.bluetooth-monitor (Bluetooth Linux Monitor) [Wireless]
7.nflog (Linux netfilter log (NFLOG) interface) [none]
8.nfqueue (Linux netfilter queue (NFQUEUE) interface) [none]
9.dbus-system (D-Bus system bus) [none]
10.dbus-session (D-Bus session bus) [none]
~~~

+ -i *interface* or --interface=*interface* ：指定要监听的网络接口。若未指定该选项，将从系统接口列表中搜寻编号最小的已配置好的接口（loopback接口除外）。**any**关键字表示所有可用的网络接口。

+ -B *buffer_size* or --buffer-size=*buffer_size* ：将操作系统的捕获缓冲区大小设置为*buffer_size*，单位是KiB。

+ -c *count* ：捕获到*count*个packet后退出。

+ -C *file_size* ：将原始packet写入文件之前，检查当前文件大小是否超过*file_size*，如果超过则关闭当前文件并打开一个新文件。文件名由**-w**标志指定，第一个文件之后的文件会添加一个从1 开始的数字后缀。*file_size*默认值是一百万字节。

  通过添加k/K，m/M或者g/G后缀可以将单位指定为KiB，MiB或者GiB。

+ -e ：在每一行输出链路首部。

+ -F *file* ：使用*file*作为过滤表达式的输入。命令行中给出的额外的表达式会被忽略。

+ -G *rotate_seconds* ：如果指定该选项，tcpdump会每*rotate_seconds*秒滚动由**-w**选项指定的捕获文件。文件将具有**-w**指定的名称，该名称包含**[strftime(3)](https://man7.org/linux/man-pages/man3/strftime.3.html)**定义的时间格式。如果时间格式没有指定，那么新文件会覆盖之前的文件。每当生成的文件名不唯一时，tcpdump将覆盖预先存在的数据；因此不建议提供比捕获周期更粗糙的时间格式。

~~~shell
# 每10秒生成一个新文件
# -Z确保在切换输出文件后依然以root用户运行
$sudo tcpdump -G 10 -w "%Y-%m-%d %H-%M-%S.pcap" -Z root
# 输出
$ll 2022*
-rw-r--r-- 1 root root 42410 11月 23 16:48 '2022-11-23 16-48-08.pcap'
-rw-r--r-- 1 root root  4670 11月 23 16:48 '2022-11-23 16-48-18.pcap'
-rw-r--r-- 1 root root   938 11月 23 16:48 '2022-11-23 16-48-29.pcap'
-rw-r--r-- 1 root root   580 11月 23 16:48 '2022-11-23 16-48-39.pcap'
~~~

+ --version ：输出tcpdump和libpcap版本信息并退出。

+ -n ：不要将地址转换为名字（例如主机地址、端口号等）。

+ -# or --number ：在每一行的开始打印packet序号。

+ -r *file* ：从*file*中读取packet。**-**代表标准输入。

+ -S or --absolute-tcp-sequence-numbers ：打印TCP序列号的绝对值而不是相对值。

+ -s *snaplen* or --snapshot-length=*snaplen* ：从每个packet抓取*snaplen*字节的数据，而不是默认的262144字节。0表示不截断，抓取完整的packet。

+ -t ：不打印时间戳。

+ -tt ：将时间戳打印为自1970年1月1日00:00:00 UTC以来的秒数和当前秒的小数部分。

+ -ttt ：打印本行与上一行之间的时间差。

+ -tttt ：在每行上打印时间戳，作为自午夜以来的小时、分钟、秒和秒的小数部分，前面是日期。

+ -ttttt ：打印当前行和第一行的时间差。

+ -v or -vv or -vvv ：打印packet的详细信息。

+ -V *file*  ：从*file*中读取文件名列表。**-**代表标准输入。

+ -w *file* ：将原始packet写入*file*。

+ -W *filecount* ：

  + 与**-C**联合使用会将创建的文件数量限制为指定数量，并从头开始覆盖文件，从而创建一个“滚动”缓冲区。此外，它会用足够多的前导 0 来命名文件以支持最大文件数，从而使它们能够正确排序。
  + 与**-G**联合使用将限制创建的文件数量，达到限制时以状态0退出。
  + 与**-C**和**-G**同时使用则该选项会被忽略，仅仅会影响文件名。
  
+ -x ：解析和打印时，除了打印每个packet的首部外，还以十六进制打印每个packet的数据（减去其链路首部）。将打印整个packet或 *snaplen* 字节中较小的一个。请注意，这是整个链路层packet，因此对于填充的链路层（例如以太网），当更高层packet比所需的填充短时，填充字节也将被打印。在当前实现中，如果packet被截断，此标志可能与 -xx 具有相同的效果。

+ -xx ：在解析和打印时，除了打印每个packet的首部外，还要以十六进制打印每个packet的数据，包括其链路首部。

+ -X ：在解析和打印时，除了打印每个packet的首部外，还可以以十六进制和ASCII打印每个packet的数据（减去其链路首部）。这对于分析新协议非常方便。在当前实现中，如果packet被截断，则此标志可能具有与-XX相同的效果。

+ -XX ：在解析和打印时，除了打印每个packet的首部外，还要以十六进制和ASCII格式打印每个packet的数据，包括其链路首部。

+ -Z *user* or --relinquish-privileges=*user* ：如果tcpdump以root身份运行，在打开捕获设备或输入保存文件之后，但在打开任何保存文件进行输出之前，将用户ID更改为*user*，将组ID更改为*user*的主要组。

  此行为也可以在编译时默认启用。

## 表达式

[表达式](https://www.tcpdump.org/manpages/pcap-filter.7.html)用来判断一个packet是否被捕获。

*filter*表达式可以由多部分组成，每一部分包含多个限定符。共有三种不同类型的限定符：

+ type ：*type*限定符可以是**host**，**net**，**port**或者**portrange**。例如：‘**host** foo’，‘**net** 128.3’，‘**port 20**’或者‘**portrange** 6000-6008’。**host**是默认的*type*限定符。
+ dir ：*dir*限定符指定传输方向。可能的方向限定符包括**src**，**dst**，**src or dst**，**src and dst**，**ra**，**ta**，**addr1**，**addr2**，**addr3**以及**addr4**。例如：**src** foo，**src or dst port** ftp-data。默认的*dir*限定符是**src or dst**。限定符**ra**，**ta**，**addr1**，**addr2**，**addr3**以及**addr4**仅适用于IEEE 802.11 Wireless LAN。
+ proto ：*proto*限定符匹配特定的协议。可能的协议包括**ether**，**fddi**，**tr**，**wlan**，**ip**，**ip6**，**arp**，**rarp**，**decnet**，**sctp**，**tcp**以及**udp**。例如：**ether src** foo，**arp net** 128.3，**tcp port** 21，**udp portrange** 7000-7009，**wlan addr2** 0:2:3:4:5:6。默认匹配*type*可匹配的所有协议。例如：**src** foo等同于**(ip or arp or rarp) src** foo，**net** bar等同于**(ip or arp or rarp) net** bar，**port** 53等同于**(tcp or udp or sctp) port** 53（请注意，这些示例使用无效语法来说明原理）。

表达式支持逻辑运算符，包括

+ && 或者 and
+ || 或者 or
+ ! 或者 not

## 例子

1. 捕获目的端口是443或8443端口的packet

~~~shell
$tcpdump dst port 443 or dst port 8443
# 或者简写为
$tcpdump dst port 443 or 8443
~~~

2. 捕获来自10.0.0.1的packet

~~~shell
$tcpdump src host 10.0.0.1
~~~

3. 捕获10.0.0.1和10.0.0.2以外的所有主机的通信

~~~shell
$tcpdump ip host 10.0.0.1 and !10.0.0.2
~~~

4. 基于协议进行过滤

~~~shell
# 过滤icmp协议
$tcpdump icmp
# 过滤ipv4
$tcpudmp "ip proto \tcp"
# 或者
$tcpdump ip proto 6
# 或者
$tcpdump "ip protochain \tcp"
# 过滤ipv6
$tcpdump "ip6 proto \tcp"
# 或者
$tcpdump ip proto 6
# 或者
$tcpdump "ip6 protochain \tcp"
~~~

5. 精确匹配

tcpdump可以根据包的内容进行匹配

+ proto[n] ：获取proto报文里的第n个字节
+ proto[n:c] ：获取proto报文里从n开始的连续c个字节

~~~shell
# 捕获icmp echo报文
$tcpdump "icmp[1]=0x00"
# 捕获icmp echo报文并且其id为0x11d7
$tcpdump "icmp[1]=0x00 && icmp[4:2]=0x11d7"
# 捕获http GET请求
# (tcp[12:1]&0xf0)>>2)获取tcp首部长度
# tcp[((tcp[12:1]&0xf0)>>2):4]截取tcp数据段起始的四个字节
# “GET ”的十六进制表示0x47455420
$tcpdump "tcp[((tcp[12:1]&0xf0)>>2):4]=0x47455420"
~~~

## 参考文献

1. [tcpdump官方文档](https://www.tcpdump.org/manpages/tcpdump.1.html)
2. [表达式](https://www.tcpdump.org/manpages/pcap-filter.7.html)
3. [肝了三天，万字长文教你玩转 tcpdump，从此抓包不用愁](https://baijiahao.baidu.com/s?id=1671144485218215170&wfr=spider&for=pc)
