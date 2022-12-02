# SS

## NAME

ss - 另一个统计socket信息的工具

## 简介

ss [options] [ FILTER ]

## 描述

ss用来统计socket的统计数据。它允许展示类似于*netstat*的统计信息。相比于其他工具，ss可以展示更多的TCP和状态信息。

## 选项

当没有任何选项被指定时，ss展示已经建立连接的、非监听状态的socket（例如，TCP/UNIX/UDP）列表。

**-h,--help** 显示选项摘要。

**-V,--version** 输出版本信息。

**-H,--no-header** 不输出首行。

**-O,--oneline** 将每个socket的数据打印在单独行。

**-n,--numeric** 不要尝试解析服务名。显示准确的带宽值，而不是以人类可读的方式显示。

**-r,--resolve** 尝试解析数字地址/端口号。

**-a,--all** 同时展示监听和非监听（对于TCP来说这意味着已建立的连接）状态的socket。

**-l,--listening** 仅展示监听状态的socket（这些socket默认是被忽略的）。

**-o,--options** 展示定时器信息。对于TCP协议，输出格式是：

`timer:(<timer_name>,<expire_time>,<retrans>)`

**<timer_name>** ：

​		**on** 下列定时器之一：TCP retrans定时器，TCP early retrans定时器，tail loss probe定时器。

​		**keepalive** tcp保活定时器

​		**timewait** timewait定时器

​		**persist** 0窗口探测定时器

​		**unknown** 上述定时器之外的定时器

**<expire_time>**

​		定时器将在多长时间内超时

**<retrans\>**

​		重传次数

**-e,-extended** 展示详细的socket统计信息。输出格式是：

`uid:<uid_number> ino:<inode_number> sk:<cookie>`

**<uid_number>**

​		socket所属的uid

**<inode_number>**

​		socket在VFS中的inode号

**<cookie\>**

socket的uuid

**-m,--memory** 显示socket的内存使用。输出格式是：

```shell
skmem:(r<rmem_alloc>,rb<rcv_buf>,t<wmem_alloc>,tb<snd_buf>,
                            f<fwd_alloc>,w<wmem_queued>,o<opt_mem>,
                            bl<back_log>,d<sock_drop>)
```

**<rmem_alloc>**

​		为了接收packet而已经分配的内存。

**<rcv_buf>**

​		为了接收packet总共可以分配多少内存。

**<wmem_alloc>**

​		为了发送packet（已经交付网络层）而已经使用的的内存。

**<snd_buf>**

​		为了发送packet总共可以分配多少内存。

**<fwd_alloc>**

​		由socket分配的、作为缓存的内存，但是还没有用来接收/发送packet。如果需要内存来发送/接收packet，那么该缓存中的内存会首先被使用而不是分配额外内存。

**<wmem_queued>**

​		为了发送packet（尚未交付网络层）而已经分配的内存

**<ropt_mem>**

​		存储socket选项的内存，例如，TCP MD5签名的key。

**<back_log>**

​		用于sk back_log队列的内存。在进程上下文中，如果进程正在接收packet，同时一个新的packet被接收，那么它将被放入sk back_log队列，因此进程可以立即接收该packet。

**<sock_drop>**

​		在de-multiplexed到socket之前而被丢弃的packet的数量。

**-p,-processes** 显示使用socket的进程。

**-i,--info** 显示内部的TCP信息。下列域会出现：

+ ts 时间戳选项被设置则显示“ts”字符串
+ sack sack选项被设置则显示“sack”字符串
+ ecn 显式拥塞通告选项被设置则显示“ecn”字符串
+ ecnseen 如果在收到的packet中设置了saw ecn标志则显示“ecnseen”字符串
+ fastopen fastopen选项被设置则显示“fastopen”字符串
+ cong_alg 拥塞控制算法的名字，默认的拥塞控制算法是“cubic”
+ wscale:<snd_wscale>:<rcv_wscale> 如果窗口扩大选项被设置，该域显示发送扩大因子和接收扩大因子
+ rto:<icsk_rto> tcp重传定时器的值，单位是毫秒
+ backoff:<icsk_backoff> 用于指数退避重传，真实的重传超时时间是`icsk_rto << icsk_backoff`
+ rtt:<rtt\>/<rttvar\> rtt是平均往返时间，rttvar是rtt的平均偏差，单位为毫秒
+ ato:<ato\> ack超时时间，单位是毫秒，用于延迟ack模式
+ mss:<mss\> 最大报文段长度
+ cwnd:<cwnd\> 拥塞窗口大小
+ pmtu:<pmtu\> 路径MTU的值
+  ssthresh:<ssthresh\> tcp慢启动门限
+ bytes_acked:<bytes_acked> 已确认的字节数量
+ bytes_received:<bytes_received> 以接收的字节数量
+ segs_out:<segs_out> 已发送的段
+ segs_in:<segs_in> 已接收的段
+ send <send_bps>bps 发送比特率
+ lastsnd:<lastsnd\> 发送最后一个packet后经过了多少毫秒
+ lastrcv:<lastrcv\> 接收最后一个packet后经过了多少毫秒
+ lastack:<lastack\> 接收最后一个ack后经过了多少毫秒
+ pacing_rate <pacing_rate>bps/<max_pacing_rate>bps pacing rate和最大pacing rate
+ rcv_space:<rcv_space> TCP内部自动调整接收缓冲区的帮助变量
+ tcp-ulp-mptcp flags:[MmBbJjecv] and token:<rem_token(rem_id)/loc_token(loc_id)> seq:<sn\> and sfseq:<ssn\> ssnoff:<off\> maplen:<maplen\> MPTCP的subflow信息

**--tos** 显示ToS和优先级信息。下列域会出现：

+ tos IPv4服务类型字节
+ tclass IPv6 流类别字节
+ class_id class id 由net_cls cgrou设置。如果class是0那么该属性显示由SO_PRIORITY设置的优先级

**--cgroup** 显示cgroup信息。下列域会出现：

+ cgroup v2 路径名。该路径名关联与层次结构中的挂载点。

**-K,--kill** 尝试强制关闭socket。该选项展示成功关闭的socket并默默跳过不支持关闭的socket。仅支持IPv4或IPv6 socket。

**-s,--summary** 打印摘要统计信息。此选项不解析从各种源获取摘要的套接字列表。当套接字数量太大以至于解析/proc/net/tcp非常困难时，它非常有用。

**-E,--events** 持续展示socket的破坏。

 **-Z,--context** 类似于-p选项但是同时展示进程安全上下文。

对于 [netlink(7)](https://man7.org/linux/man-pages/man7/netlink.7.html) socket，初始化上下文展示如下：

1. 如果存在合法的pid则显示进程上下文。
2. 如果目的是 kernel(pid=0) 则显示 kernel 初始化上下文。
3. 如果 kernel 或者用户分配了一个唯一的id，将上下文显示为“unavailable”。这通常意味着活跃的 netlink socket 不止一个。

**-z,--contexts** 类似于-Z选项但同时显示 socket 上下文。套接字上下文取自关联的inode，而不是内核所持有的实际套接字上下文。套接字通常标有创建进程的上下文，但是所显示的上下文将反映应用的任何策略角色、类型和/或范围转换规则，因此是一个有用的参考。

**-N NSNAME,--net=NSNAME** 切换到指定的命名空间。

**-b,--bpf** 显示socket经典BPF过滤器（只有管理员能获得这些信息）。

**-4,--ipv4** 仅展示IP版本4 socket（-f inet4的别名）。

**-6,--ipv6** 仅展示IP版本6 socket（-f inet6的别名）。

**-0,--packet** 展示PACKET socket（-f link的别名）。

**-t,--tcp** 展示TCP socket。

**-u,--udp** 展示UDPsocket。

**-d,--dccp** 展示DCCP socket。

**-w,--raw** 展示RAW socket。

**-x,--unix** 展示Unid域 socket。

**-S,--sctp** 展示SCTP socket。

**--vsock** 展示vsock socket（-f vsock的别名）。

**--xdp** 展示XDP socket（-f xdp的别名）。

**--inet-sockopt** 展示inet socket 选项。

**-f FAMILY,--family=FAMILY** 展示特定类型地址族的socket。当前支持下列地址族：unix, inet, inet6, link, netlink, vsock, xdp。

**-A QUERY,--query=QUERY,--socket=QUERY** 要输出的socket列表，以逗号分隔。支持下列标识符：all, inet, tcp, udp, raw, unix, packet, netlink, unix_dgram, unix_stream, unix_seqpacket, packet_raw, packet_dgram, dccp, sctp, vsock_stream, vsock_dgram, xdp。列表中的任何项目都可以选择以感叹号（！）作为前缀以避免在套接字表中被打印。

**-D FILE,--diag=FILE** 不要展示任何信息，仅仅将应用过滤器后的 TCP socket的原始信息输出到FILE。如果FILE **-** 则使用标准输出。

**-F FILE,--filter=FILE** 从FILE中读取filter信息。FILE的每一行都被解释为单个命令行选项。如果FILE是 **-** 则使用标准输入。

**FILTER := [ state STATE-FILTER ] [ EXPRESSION ]** 有关过滤器的详细信息，请查看官方文档。

## STATE-FILTER

STATE-FILTER 允许构造任意的状态集合以进行匹配。它的语法是关键字state和exclude的序列，后跟状态标识符。

可用的标识符如下：

所有的标准TCP状态：established, syn-sent, syn-recv, fin-wait-1, fin-wait-2, time-wait, closed, close-wait, last-ack, listening and closing。

all - 所有状态

connected - 处理 listening 和 closed 外的状态

synchronized 除了syn-sent以外的所有 connected 状态

bucket - 作为 minisocket 进行维护的状态，例如 time-wait 和 syn-recv

big - 与 bucket 相反

## 表达式

表达式允许基于指定的标准进行过滤。

表达式由一系列由布尔运算符组合而成的谓词组成。按照优先级从低到高，可用的操作符是`or(or | or ||)`，`and(or & or &&)`，`not(or !)`。如果相邻的谓词之间没有操作符，一个隐式的 **and** 会被插入。子表达式可以通过括号分组。

支持下列谓词：

**{dst|src} [=] HOST**

​		测试源或目的是否匹配 HOST。详细信息请查看 HOST 语法。

**{dport|sport} [OP] [FAMILY:]:PORT**

​		将源或目的端口与 PORT 进行比较。OP 可以是"<", "<=", "=", "!=", ">=" and ">"中的任何一个。遵循通常的算术规则。FAMILY 和 PORT在下面的 HOST 语法中被描述。

**dev [=|!=] DEVICE**

​		基于连接使用的设备进行匹配。DEVICE 可以是设备名或接口的索引。

**fwmark [=|!=] MASK**

​		基于连接的 fwmark 进行匹配。可以是一个特定的mark值或者后跟“/”的mark值和一个指示使用那些比特进行比较的位掩码。例如，“fwmark = 0x01/0x03“将匹配 最低两位有效位为0x01的 fwmark。

**cgroup [=|!=] PATH**

​		根据连接是否属于给定的cgroup 路径进行匹配。

**autobound**

​		根据源地址的端口或路径是否为自动分配的（而不是强制指定的）进行匹配。

大部分操作符都有别名。如果没有操作符被指定那么假定指定的操作符是”=“。下面的分组中，每一组内的操作符都是等价的。

+ = == eq
+ != ne neq
+ \> gt
+ < lt
+ \>= ge geq
+ <= le leq
+ ! not
+ | || or
+ & && and

## HOST 语法

通用的 host 语法是 `[FAMILY:]ADDRESS[:PORT]`

FAMILY 必须是-f 选项支持的地址族。如果没有指定会被默认指定为-f选项的地址族，如果-f也没有指定，则会被假定为 inet 或 inet6。注意，所有表达式中的 host 条件应该是一致的或者仅仅是 inet 和 inet6。如果存在其他地址族组合的情况，那么结果可能是不符合预期的。

ADDRESS 和 PORT 的格式依赖于所使用的地址族。”*“可以作为地址或端口的通配符。每个地址族的详细信息如下：

**unix** ADDRESS是一个文件名替换模式（参考[fnmatch(3)](https://man7.org/linux/man-pages/man3/fnmatch.3.html)），它将与unix socket地址进行忽略大小写的匹配。路径和抽象名都是支持的。Unix地址不支持端口，也不能使用"*"作为通配符。

**link** ADDRESS要匹配的、大小写不敏感的以太网协议名。PORT是ip link中输出的期望的设备名和设备索引。

**netlink** ADDRESS是netlink 地址族的描述符。可用的值来自/etc/iproute2/nl_protos。PORT是socket的port id，通常与所属的进行id是一致的。值”kernel“可以用来代表内核（port id 是0）。

**vsock** ADDRESS是一个代表CID地址的整数，PORT是端口。

**inet and inet6** ADDRESS是ip地址（根据地址族决定是v4还是v6）或一个可解析为指定版本ip地址的DNS主机名。一个ipv6地址必须被中括号包裹以消除与端口号分隔符的歧义。地址还可以具有CIDR表示法（反斜杠后跟比特表示的前缀长度）中给出的前缀长度。PORT 可以是数字表示的 socket 端口号，或者端口匹配的服务名。

## 用例

ss -t -a

​		展示所有TCP socket。

ss -t -a -Z

​		展示所有的TCP socket及其关联的SELinux安全上下文。

ss -u -a

​		展示所有的UDP socket。

ss -o state established '( dport = :ssh or sport = :ssh )'

​		展示所有已建立的ssh连接。

ss -x src /tmp/.X11-unix/*

​		展示所有连接到X server 的本地进程。

ss -o state fin-wait-1 '( sport = :http or sport = :https )' dst 193.233.7/24

​		列出所有到apache网络193.233.7/24的状态是 FIN-WAIT-1 的tcp socket并查看他们的定时器。

ss -a -A 'all,!tcp'

​		列出除了tcp以外的所有socket表中的所有socket。

## 官方文档

[ss(8) — Linux manual page](https://man7.org/linux/man-pages/man8/ss.8.html)