# df

df 显示每个文件名参数所在的文件系统的可用容量。如果未指定文件名，则显示当前装载的所有文件系统上的可用空间。默认情况下，空间以1K块显示，除非设置了环境变量POSIXLY_CORRECT，在这种情况下使用512字节块。

## 用法

```shell
df [OPTION]... [FILE]...
```

## 选项

显示每个 FILE 所在的文件系统的信息，默认显示所有文件系统。

+ -a, --all 包括伪文件系统、重复文件系统、不可访问的文件系统。
+ -B, --block-size=*SIZE* 打印前按照 *SIZE* 对文件系统容量进行缩放。例如，`-BM` 以1048576字节为单位打印容量。
+ -h, --human-readable 以易读的形式打印容量（1024的幂，例如1023M）。
+ -H, --si 以1000的幂打印容量（例如，1.1G）。
+ -i, --inodes 列出 inode 信息而不是块的使用信息。
+ -k 类似 --block-size=*1K*
+ -l, --local 仅列出本地的文件系统。
+ --no-sync 在获取使用信息之前不同步（默认行为）
+ --output[=*FIELD_LIST*] 使用 *FIELD_LIST* 定义的格式进行输出，或者在 *FIELD_LIST* 被忽略的情况下打印所有输出。
+ -P, --portability 使用 POSIX 输出格式。
+ --sync 在获取使用信息之前调用同步。
+ --total 删除所有对可用空间不重要的条目，并生成一个总计。
+ -t, --type=*TYPE* 仅列出 *TYPE* 类型的文件系统。
+ -T, --print-type 打印文件系统类型。
+ -x, --exclude-type=*TYPE* 列出除了 *TYPE* 以外类型的文件系统。
+ -v 忽略
+ --help 打印帮助信息并退出。
+ --version 打印版本信息并退出。

展示的容量值以 `--block-size` 或者环境变量中的 DF_BLOCK_SIZE、BLOCK_SIZE、BLOCKSIZE中的第一个可用的为单位。如果上述都不可用，单位默认被设置为1024字节（如果 POSIXLY_CORRECT 被设置则为512）。

参数 *SIZE* 是整数并且可选单位（例如：10K 是10*1024）。单位是K，M，G，T，P，E，Z，P（1024的幂） 或者KB，MB

，...（1000的幂）。也可以使用二进制前缀，例如：KiB=K，MiB=M等等。

FIELD_LIST 是一个逗号分隔的被包含的列的列表。合法的列表名包括：'source', 'fstype', 'itotal', 'iused','iavail', 'ipcent', 'size', 'used', 'avail', 'pcent', 'file' 和 'target' 。

## 官方文档

[df(1) — Linux manual page](https://man7.org/linux/man-pages/man1/df.1.html)