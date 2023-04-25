# tee

从标准输入读入并写道标准输出和文件中。

## 语法

~~~shell
tee [OPTION]... [FILE]...
~~~

## 描述

将标准输入拷贝到每一个 FILE，同时输出到标准输出。

+ -a, --append：将输出追加到文件中而不是覆盖
+ -i, --ignore-interrupts：忽略 interrupt 信号
+ -p：诊断写入非管道的错误
+ --output-error[=MODE]：设置发生写错误时的行为，MODE 可以是一下的枚举值
  + warn：诊断写入任何输出的错误
  + warn-nopipe：诊断写入任何非管道输出的错误
  + exit：发生任何输出的写入错误时退出
  + exit-nopipe：发生任何非管道的写入错误时退出

