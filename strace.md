# strace

`strace` 是一个强大的诊断工具，用于监视和调试运行中的程序。它通过跟踪系统调用（system calls）和信号（signals）来提供有关程序执行的信息。这对于调试程序的行为、识别性能瓶颈或了解程序与操作系统之间的交互非常有用。

## 常用参数

+ -tt 在每行输出的前面，显示毫秒级别的时间
+ -T 显示每次系统调用所花费的时间
+ -v 对于某些相关调用，把完整的环境变量，文件stat结构等打出来。
+ -f 跟踪目标进程，以及目标进程创建的所有子进程
+ -e 控制要跟踪的事件和跟踪行为,比如指定要跟踪的系统调用名称
+ -o 把 strace 的输出单独写到指定的文件
+ -s 当系统调用的某个参数是字符串时，最多输出指定长度的内容，默认是32个字节
+ -p 指定要跟踪的进程 pid，要同时跟踪多个 pid， 重复多次 -p 选项即可
+ -c 对系统调用进行分类统计

其中，-e 选项可以跟定特定类别的系统调用 `-e trace=syscall_set` 、`--trace=syscall_set`，`syscall_set` 常用的取值如下

+ %file 文件相关系统调用
+ %process 进程生命周期相关系统调用
+ %network/%net 网络相关系统调用
+ %signal 信号相关系统调用
+ %desc  文件描述符相关系统调用（包括 epoll 等）
+ %memory 内存映射相关系统调用

也可以指定特定的系统调用名跟踪特定的系统调用，比如 `strace --trace=write` 之跟踪 `write` 系统调用
