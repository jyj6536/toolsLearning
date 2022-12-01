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

**<retrans>**

​		重传次数

**-e,-extended** 展示详细的socket统计信息。输出格式是：

`uid:<uid_number> ino:<inode_number> sk:<cookie>`

**<uid_number>**

​		socket所属的uid

**<inode_number>**

​		socket在VFS中的inode号

**<cookie>**

socket的uuid

**-m,--memory** 显示socket的内存使用。输出格式是：

```shell
skmem:(r<rmem_alloc>,rb<rcv_buf>,t<wmem_alloc>,tb<snd_buf>,
                            f<fwd_alloc>,w<wmem_queued>,o<opt_mem>,
                            bl<back_log>,d<sock_drop>)
```