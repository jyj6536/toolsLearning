# du

du 估计文件空间的使用情况。

## 用法

```shell
du [OPTION]... [FILE]...
du [OPTION]... --files0-from=F
```

## 选项

统计 FILEs 的设备使用情况（对于目录进行递归统计）。

+ -0, --null 输出的结尾使用 NUL 而不是换行符
+ -a, --all 统计所有文件而不仅仅是目录
+ --apparent-size 打印文件大小而不是设备使用情况；虽然文件大小通常较小，但由于文件中的稀疏文件中的空洞、内部碎片、间接块等原因，文件大小可能会更大。
+ -B, --block-size=*SIZE* 打印大小前按照 SIZE 进行缩放；例如，`-BM` 以 1,048,576 字节为单位进行打印；请参考下面的 SIZE 格式
+ -b, --bytes 等价于 `--apparent-size --block-size=1`
+ -c, --total 产生总量统计
+ -D, --dereference-args 仅针对命令行中列出的符号连接进行解引用
+ -d, --max-depth=*N* 指定递归深度；`--max-depth=0` 等价于 `--summarize`
+ --files0-from=*F* 统计文件 F 中指定的以 NUL 结尾的文件的设备使用情况；如果 F 是 -，则从标准输入读取文件名
+ -H 等价于 `--dereference-args (-D)`
+ -h, --human-readable 以人类可读的格式打印文件大小（例如，1K 234M 2G）
+ --inodes 打印 inode 使用信息而不是块的使用信息
+ -k 等价于 `--block-size=1K`
+ -L, --dereference 对所有符号连接进行解引用
+ -l, --count-links 对于硬连接的文件进行多次统计
+ -m 等价于 `--block-size=1M`
+ -P, --no-dereference 不要跟随任何符号连接（这是默认行为）
+ -S, --separate-dirs 不统计目录中的子目录
+ --si 类似与 `-h`，但是以1000而不是1024为因子
+ -s, --summarize 对于每一个参数仅显示一个摘要
+ -t, --threshold=*SIZE* 如果是正值则排除比 SIZE 小的条目，否则反之
+ --time 显示目录或其子目录中任何文件的上次修改时间
+ --time=*WORD* 显示 WORD 指定的事件而不是修改时间：atime，access，use，ctime 或者 status
+ --time-style=*STYLE* 以 STYLE 指定的格式显示时间：full-iso，long-iso，iso 或者 +FORMAT；FORMAT 的解释类似于 `date`
+ -X, --exclude-from=*FILE* 排除任何匹配 FILE 的文件
+ --exclude=*PATTERN* 排除任何匹配 PATTERN 的文件
+ -x, --one-file-system 跳过在不同文件系统上的目录
+ --help 显示帮助信息并退出
+ --version 打印版本信息并退出

展示的容量值以 `--block-size` 或者环境变量中的 DF_BLOCK_SIZE、BLOCK_SIZE、BLOCKSIZE中的第一个可用的为单位。如果上述都不可用，单位默认被设置为1024字节（如果 POSIXLY_CORRECT 被设置则为512）。

参数 *SIZE* 是整数并且可选单位（例如：10K 是10*1024）。单位是K，M，G，T，P，E，Z，P（1024的幂） 或者KB，MB

，...（1000的幂）。也可以使用二进制前缀，例如：KiB=K，MiB=M等等。

## PATTERNS

PATTERN 是一个 shell 模式（而不是正则表达式）。模式 `?` 匹配任何的单个字符，`*` 匹配任何字符串（0个，1个或多个字符）。例如，`*.o` 可以匹配任何文件名以 `.o` 结尾的文件。因此，命令

```shell
du --exclude='*.o'
```

会跳过任何以 `.o` 结尾的文件和子目录（包括文件 `.o` 本身）。