# nc

nc 命令是 Linux 系统中的一个网络工具，全称为 Netcat，它可以用于创建 TCP/UDP 连接、监听端口、传输文件等操作。

## 常用选项

+ -l 在指定的端口进行监听
+ -p 指定端口号
+ -u 使用 UDP
+ -v 显示详细调试信息
+ -w 设置超时时间

## 例子

1. 创建 TCP/UDP 连接

   ~~~shell
   #创建到 127.0.0.1:80 的 TCP 连接
   nc 127.0.0.1 80
   #创建到 127.0.0.1 1234 的 UDP 连接
   nc -u 127.0.0.1 1234
   ~~~

2. 监听端口

   ~~~~shell
   #监听 TCP(-u UDP) 127.0.0.1:8080
   nc -l 127.7.7.1 8080
   ~~~~

   
