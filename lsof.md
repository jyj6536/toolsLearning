# lsof

lsof - 列出打开的文件

## 基本作用

lsof 用来输出打开的文件信息。这些文家年包括普通文件，目录，块文件，字符文件，执行中的文本引用，库，流或者网络文件（Internet 套接字，NFS 文件或者 Unix 域套接字）。

## 常用参数

+ -a 如果 lsof 命令指定多个选项，这些选项之间默认是或的关系，-a 可以使这些选项变为与的关系。

+ -p s 特定进程打开的文件列表，其中 s 是一个逗号分隔的进程 pid 集合。例如， `123` 或者 `123,^456`，`^` 表示不包含。

+ fileName 查看打开特定文件的所有进程。

+ -l 禁止将用户 id 转换为登陆名，即输出中直接输出用户 id。

+ -c c 执行以字符 c 开头的命令的进程所打开的文件。可以通过多个 -c 选项指定多个命令。如果 c 以 `^` 开头则忽略命令名包含 c 的进程打开的文件。如果 c 以斜杠开头和结束，那么 c 会被解释为正则表达式。c 中的 shell 元字符必须被转义以避免被 shell 解释。结束的斜杠后可以跟随下列修饰符：

  + b 正则表达式是基本形式
  + i 忽略大小写
  + x 正则表达式是扩展形式，这是默认的

+ -u s 特定用户打开的文件列表，其中 s 是一个逗号分隔的用户名或者用户 id 集合。例如，`abe` 或者 `548,root`。

+ -i [i] 选择 Internet 地址匹配 i 中指定的地址的文件。可以通过多个 -i 指定至多100个地址。i 的标准格式：`[46][protocol][@hostname|hostaddr][:service|port]`。其中的个部分的含义如下：

  + 46 指定 IP 版本，不指定则后续 ip 地址适用于所有 IP 版本。
  + protocol 协议名，TCP 或 UDP。
  + hostname Internet 主机名。除非指定 IP 版本，否则所有与版本的与该主机名关联的文件都会被选择。
  + hostaddr 点分形式的 ipv4 地址或者放在中括号中的冒号分隔的 ipv6 地址。
  + service `/etc/services`中的服务名。
  + port 端口号

  一些例子：

  + 6 - 仅IPv6
  + TCP:25 - tcp的25端口
  + @1.2.3.4 - ipv4地址 `1.2.3.4`
  + @[3ffe:1ebc::1]:1234 - ipv6 地址 `3ffe:1ebc::1`，端口1234
  + UDP:who - udp who 服务端口
  + TCP@lsof.itap:513 - tcp 的513端口，主机是 lsof.itap
  + tcp@foo:1-10,smtp,99 - tcp的1-10端口，smtp服务以及99端口，主机名是 foo
  + tcp@bar:1-smtp - TCP 从1到smtp的端口，主机名是 bar
  + :time - tcp，udp 或 udp-lite 的 time 服务

## 例子

### 1.列出所有打开的文件（无参数）

```shell
$ lsof
```

输出

```shell
COMMAND     PID   TID TASKCMD                  USER   FD      TYPE             DEVICE    SIZE/OFF       NODE NAME
systemd       1                                root  cwd       DIR              259,3        4096          2 /
systemd       1                                root  rtd       DIR              259,3        4096          2 /
systemd       1                                root  txt       REG              259,3     1849992   27399850 /usr/lib/systemd/systemd
systemd       1                                root  mem       REG              259,3      149760   27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
systemd       1                                root  mem       REG              259,3       27072   27396271 /usr/lib/x86_64-linux-gnu/libcap-ng.so.0.0.0
systemd       1                                root  mem       REG              259,3      613064   27394179 /usr/lib/x86_64-linux-gnu/libpcre2-8.so.0.10.4
systemd       1                                root  mem       REG              259,3      170456   27394183 /usr/lib/x86_64-linux-gnu/liblzma.so.5.2.5
systemd       1                                root  mem       REG              259,3      841808   27400994 /usr/lib/x86_64-linux-gnu/libzstd.so.1.4.8

```

其中，每列的参数含义如下：

+ COMMAND 命令名称
+ PID 进程ID
+ TID 线程ID
+ USER uid 或者登陆名
+ FD 文件描述符

  + cwd 当前工作目录
  + rtd 根目录
  + txt 程序文件（代码和数据）
  + mem 内存映射文件
+ TYPE 文件类型
  + DIR 目录
  + REG 普通文件
  + CHR 字符文件
  + FIFO 先入先出管道
+ DEVICE 设备号
+ SIZE/OFF 文件大小/偏移量
+ NODE 文件节点
+ NAME 文件挂载点和文件系统

### 2.列出特定用户打开的文件

```shell
$lsof -u root
```

输出

```shell
COMMAND     PID USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
systemd       1 root  cwd       DIR              259,3     4096          2 /
systemd       1 root  rtd       DIR              259,3     4096          2 /
systemd       1 root  txt       REG              259,3  1849992   27399850 /usr/lib/systemd/systemd
systemd       1 root  mem       REG              259,3   149760   27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
systemd       1 root  mem       REG              259,3    27072   27396271 /usr/lib/x86_64-linux-gnu/libcap-ng.so.0.0.0
systemd       1 root  mem       REG              259,3   613064   27394179 /usr/lib/x86_64-linux-gnu/libpcre2-8.so.0.10.4
systemd       1 root  mem       REG              259,3   170456   27394183 /usr/lib/x86_64-linux-gnu/liblzma.so.5.2.5
systemd       1 root  mem       REG              259,3   841808   27400994 /usr/lib/x86_64-linux-gnu/libzstd.so.1.4.8
systemd       1 root  mem       REG              259,3  4447536   27398391 /usr/lib/x86_64-linux-gnu/libcrypto.so.3
```

### 3.找到在特定端口运行的进程

找到在 tcp 的53端口运行的进程。

```shell
$lsof  -i TCP:53
```

输出

```shell
COMMAND   PID            USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
systemd-r 841 systemd-resolve   14u  IPv4  19092      0t0  TCP localhost:domain (LISTEN)
```

### 4.输出 COMMAND 中包含 sshd 的命令

```shell
$lsof -c sshd
```

输出

```shell
COMMAND     PID USER   FD   TYPE             DEVICE SIZE/OFF     NODE NAME
sshd        607 root  cwd    DIR              252,1     4096        2 /
sshd        607 root  rtd    DIR              252,1     4096        2 /
sshd        607 root  txt    REG              252,1   876328    20564 /usr/sbin/sshd
sshd        607 root  mem    REG              252,1    51856    26167 /usr/lib/x86_64-linux-gnu/libnss_files-2.31.so
sshd        607 root  mem    REG              252,1   137584     6017 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.28.0
sshd        607 root  mem    REG              252,1   101352    26173 /usr/lib/x86_64-linux-gnu/libresolv-2.31.so
sshd        607 root  mem    REG              252,1    22600    14554 /usr/lib/x86_64-linux-gnu/libkeyutils.so.1.8
sshd        607 root  mem    REG              252,1    56096    11697 /usr/lib/x86_64-linux-gnu/libkrb5support.so.0.1
sshd        607 root  mem    REG              252,1   191040    11659 /usr/lib/x86_64-linux-gnu/libk5crypto.so.3.1
sshd        607 root  mem    REG              252,1   157224    26172 /usr/lib/x86_64-linux-gnu/libpthread-2.31.so
sshd        607 root  mem    REG              252,1  1168056     1596 /usr/lib/x86_64-linux-gnu/libgcrypt.so.20.2.5
sshd        607 root  mem    REG              252,1   129248     1593 /usr/lib/x86_64-linux-gnu/liblz4.so.1.9.2
```

### 5.查看1号进程打开的文件

```shell
$lsof -p 1
```

输出

```shell
COMMAND PID USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
systemd   1 root  cwd       DIR              259,3     4096          2 /
systemd   1 root  rtd       DIR              259,3     4096          2 /
systemd   1 root  txt       REG              259,3  1849992   27399850 /usr/lib/systemd/systemd
systemd   1 root  mem       REG              259,3   149760   27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
systemd   1 root  mem       REG              259,3    27072   27396271 /usr/lib/x86_64-linux-gnu/libcap-ng.so.0.0.0
systemd   1 root  mem       REG              259,3   613064   27394179 /usr/lib/x86_64-linux-gnu/libpcre2-8.so.0.10.4
systemd   1 root  mem       REG              259,3   170456   27394183 /usr/lib/x86_64-linux-gnu/liblzma.so.5.2.5
systemd   1 root  mem       REG              259,3   841808   27400994 /usr/lib/x86_64-linux-gnu/libzstd.so.1.4.8
systemd   1 root  mem       REG              259,3  4447536   27398391 /usr/lib/x86_64-linux-gnu/libcrypto.so.3
systemd   1 root  mem       REG              259,3   125152   27394181 /usr/lib/x86_64-linux-gnu/liblz4.so.1.9.3
systemd   1 root  mem       REG              259,3    35440   27394629 /usr/lib/x86_64-linux-gnu/libip4tc.so.2.0.0
systemd   1 root  mem       REG              259,3  1296312   27400355 /usr/lib/x86_64-linux-gnu/libgcrypt.so.20.3.4
systemd   1 root  mem       REG              259,3   198664   27395713 /usr/lib/x86_64-linux-gnu/libcrypt.so.1.1.0
systemd   1 root  mem       REG              259,3    39024   27394251 /usr/lib/x86_64-linux-gnu/libcap.so.2.44
```

### 6.查看那些进程打开了文件 fileame

```shell
$lsof /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
```

输出

````shell
COMMAND     PID             USER  FD   TYPE DEVICE SIZE/OFF     NODE NAME
systemd       1             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
systemd-j   354             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
systemd-o   839      systemd-oom mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
systemd-r   841  systemd-resolve mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
systemd-t   842 systemd-timesync mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
accounts-   925             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
avahi-dae   929            avahi mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
bluetooth   930             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
dbus-daem   933       messagebus mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
NetworkMa   935             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
irqbalanc   942             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
networkd-   943             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
polkitd     944             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
power-pro   945             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
rsyslogd    946           syslog mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
systemd-l   952             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
thermald    955             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
udisksd     958             root mem    REG  259,3   149760 27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
````

### 7.使用 uid 代替用户名

````shell
$lsof -u root -l
````

输出

```shell
COMMAND     PID     USER   FD      TYPE             DEVICE SIZE/OFF       NODE NAME
systemd       1        0  cwd       DIR              259,3     4096          2 /
systemd       1        0  rtd       DIR              259,3     4096          2 /
systemd       1        0  txt       REG              259,3  1849992   27399850 /usr/lib/systemd/systemd
systemd       1        0  mem       REG              259,3   149760   27397490 /usr/lib/x86_64-linux-gnu/libgpg-error.so.0.32.1
systemd       1        0  mem       REG              259,3    27072   27396271 /usr/lib/x86_64-linux-gnu/libcap-ng.so.0.0.0
systemd       1        0  mem       REG              259,3   613064   27394179 /usr/lib/x86_64-linux-gnu/libpcre2-8.so.0.10.4
systemd       1        0  mem       REG              259,3   170456   27394183 /usr/lib/x86_64-linux-gnu/liblzma.so.5.2.5
systemd       1        0  mem       REG              259,3   841808   27400994 /usr/lib/x86_64-linux-gnu/libzstd.so.1.4.8
systemd       1        0  mem       REG              259,3  4447536   27398391 /usr/lib/x86_64-linux-gnu/libcrypto.so.3
```

## 官方文档

[lsof(8) — Linux manual page](https://man7.org/linux/man-pages/man8/lsof.8.html)