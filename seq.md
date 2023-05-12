# seq

seq 命令用来打印数字序列。

## 语法

~~~
seq [OPTION]... LAST
seq [OPTION]... FIRST LAST
seq [OPTION]... FIRST INCREMENT LAST
~~~

从 FIRST 开始以 INCREMENT 的间隔打印数字直到 LAST

## 参数

+ -f, --format=FORMAT：使用 printf 风格的浮点数 FORMAT
+ -s, --separator=STRING：使用 STRING 分隔数字（默认 \n)
+ -w, --equal-width：通过添加前导0使输出定宽

## 例子

~~~Shell
#输出1到10（FIRST 和 INCREMENT 默认为1）
seq 10
#输出
1
2
3
4
5
6
7
8
9
10

#输出10到1
seq 10 -1 1
10
9
8
7
6
5
4
3
2
1

#输出2到10之间的偶数
seq 2 2 10
2
4
6
8
10

#以浮点数作为 INCREMENT
seq 1 1.1 10
1.0
2.1
3.2
4.3
5.4
6.5
7.6
8.7
9.8

#定宽输出
seq -w 10
01
02
03
04
05
06
07
08
09
10

#指定输出格式
 seq -f "%.4f" 10
1.0000
2.0000
3.0000
4.0000
5.0000
6.0000
7.0000
8.0000
9.0000
10.0000

#指定分隔符
 seq -s + 10
1+2+3+4+5+6+7+8+9+10
~~~

