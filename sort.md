# sort

对文件中的行进行排序

## 语法

~~~shell
sort [OPTION]... [FILE]...
sort [OPTION]... --files0-from=F
~~~

## 选项

将所有 FILE 排序后的结果写入标准输出。

如果没有指定 FILE，或者 FILE 是 -，则读取标准输入。

+ -b, --ignore-leading-blanks：忽略前导的空白字符
+ -d, --dictionary-order：只考虑空格和字母数字字符
+ -f, --ignore-case：通过将小子字母当作大写字母对待来忽略大小写
+ -g, --general-numeric-sort：根据一般数值进行比较
+ -i, --ignore-nonprinting：仅考虑可打印字符
+ -M, --month-sort：按照以下规则比较——(unknown) < 'JAN' < ... < 'DEC'
+ -h, --human-numeric-sort：比较人类可读的数字（例如，2k、1g）
+ -n, --numeric-sort：根据字符串数值进行比较
+ -R, --random-sort：随机化，但根据相同的关键字分组
+ --random-source=FILE：从 FILE 中获取随机字节
+ -r, --reverse：反转比较结果
+ --sort=WORD：根据 WORD 排序—— -g，-h，-M，-n，-R，-V
+ -V, --version-sort：对包含数字的字符串进行排序

其他选项

+ --batch-size=NMERGE：一次最多合并 NMERGE 输入
+ -c, --check, --check=diagnose-first：对于要排序的输入进行检查，而不排序
+ -C, --check=quiet, --check=silent：类似于 -c，但是不报告第一次出现的错误行
+ --compress-program=PROG：通过 PROG 压缩临时文件；通过 PROG -d 解压他们
+ --debug：注释用于排序的行的部分，并将可以用法输出到 stderr
+ --files0-from=F：从文件F中以 NUL 结尾的名称指定的文件中读取输入；如果F为-，则从标准输入中读取名称
+ -k, --key=KEYDEF：根据 key 排序，KEYDEF 给出了位置和类型
+ -m, --merge：合并已排序的文件，不再进行排序
+ -o, --output=FILE：将输出写入 FILE 而不是标准输出
+ -s, --stable：通过禁用最后重排序比较使排序稳定化
+ -S, --buffer-size=SIZE：使用 SIZE 作为主缓冲区的大小
+ -t, --field-separator=SEP：使用 SEP 作为分隔符
+ -T, --temporary-directory=DIR：使用 DIR 作为临时目录而不是 $TMPDIR 或者 /tmp；多个选项指定多个目录
+ --parallel=N：将并发执行的 sort 的数量指定为 N
+ -u, --unique： 排序后相同的行只显示一次
+ -z, --zero-terminated：使用 NUL 而不是换行符作为行分隔符

KEYDEF 是 `F[.C][OPTS][,F[.C][OPTS]] `表示开始和停止位置，其中 F 是字段编号，C 是字段中的字符位置；两者都是从 1 开始，停止位置默认为行尾。如果 -t 和 -b 均无效，则字段中的字符从前一个空格的开头开始计数。OPTS 是一个或多个单字母排序选项 [bdfgiMhnRrV]，它覆盖该关键字的全局排序选项。如果没有给出关键字，则使用整行作为关键字。使用 --debug 来诊断不正确的关键字使用。SIZE 可以加后缀：% b K M G T P E Z Y

## 例子

假设有以下测试数据 

~~~
#test.txt
a11:12:33
d12:22:31
b1:21:32

#test1.txt
e 1
d 11
c 1
b 2
2 4
~~~

### -r 降序排序

~~~shell
sort -r test.txt
~~~

输出

~~~
d12:22:31
b1:21:32
a11:12:33
~~~

### -t 与 -k 合用按照指定域进行排序

按照第二列进行排序

~~~~shell
sort -t ":" -k 2 test.txt
~~~~

输出

~~~
a11:12:33
b1:21:32
d12:22:31
~~~

按照第三列降序排序

~~~shell
sort -t ":" -k 3r test.txt
~~~

输出

~~~
a11:12:33
b1:21:32
d12:22:31
~~~

### -n 按照数值进行排序

~~~shell
sort -k 2n test1.txt
~~~

输出

~~~
c 1
e 1
b 2
2 4
d 11
~~~

### -u 去除重复

单独使用 -u 时按照整行进行去重，与 -k 合用则针对特定域进行去重。

按照第二个域进行去重

~~~shell
sort -u -k 2,2 test1.txt
~~~

输出

~~~
e 1
d 11
b 2
2 4
~~~

其中，-k m,n 表示从第k个域直到第n个域作为比较范围，不指定n则表示从第m个域直到最后一个域

### -k a.b,c.d

从第a个域的第b个字符直到第c个域的第d个字符进行比较

~~~shell
sort -t ":" -k 2.2,3.1 test.txt
~~~

输出

~~~
b1:21:32
a11:12:33
d12:22:31
~~~

### -V 字母数字混合排序

假设有以下数据

~~~
a13
a1
a3
a12
a23
a33
~~~

不使用 -V 进行排序得到如下结果：

~~~
a1
a12
a13
a23
a3
a33
~~~

使用-V进行排序

~~~
a1
a3
a12
a13
a23
a33
~~~

可以看到，文本中的数值部分按照数值进行排序。
