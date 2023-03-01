# shell

shell 脚本是一种计算机程序，被设计用于在下列 Unix/Linux 上运行：

+ The Bourne Shell
+ The C Shell
+ The Korn Shell
+ The GNU Bourne-Again Shell

shell脚本有几个必需的构造，这些构造告诉Shell环境应该做什么以及什么时候做。毕竟，shell是一种真正的编程语言，包含变量、控制结构等。

下面的脚本使用 **read** 命令从键盘读取输入，赋值给变量 PERSON，并最终在 STDOUT 上打印。

```shell
echo "What is your name?"
read PERSON
echo "Hello, $PERSON"
```

输出

```shell
What is your name?
Tom   
Hello, Tom
```

## shell 简介

shell 为用户提供了 Unix 系统的接口。它收集来自用户的输入并基于该输入执行程序。当程序完成执行时，它将显示该程序的输出。

shell 是一个环境，我们可以在其中运行命令、程序和 shell 脚本。shell 有不同的类型，就像操作系统有不同的风格一样。每种风格的 shell 都有自己的一组公认的命令和函数。

### shell 类型

在 Unix 中，有两种主要的 shell 类型 -

+ **Bourne shell** 如果正在使用该类型的 shell，那么 $ 会是默认提示符
+ **C shell** 如果正在使用该类型的 shell，那么 % 会是默认提示符

Bourne Shell 有下列子类 -

- Bourne shell (sh)
- Korn shell (ksh)
- Bourne Again shell (bash)
- POSIX shell (sh)

C Shell -

- C shell (csh)
- TENEX/TOPS C shell (tcsh)

原始的 Unix shell 是二十世纪七十年代中期由 Stephen R. Bourne 开发的，当时他正在位于新泽西的 AT&T 贝尔实验室工作。

Bourne shell 是 Unix 系统中出现的第一个 shell，因此被称为 “the shell”。

在大多数版本的 Unix 系统中，Bourne shell 通常被安装为 **/bin/sh**。因此，它是编写在不同版本的 Unix 上运行的脚本时的 shell 选择。

## shell 变量

变量是我们为其赋值的字符串。分配的值可以是数字、文本、文件名、设备或任何其他类型的数据。


变量只是指向实际数据的指针。shell允许我们创建、分配和删除变量。

### 变量名

shell 中的变量名可以由字母、数字、下划线组成。其中，第一个字符不能是数字。

下面是合法的变量名-

```shell
_ALI
TOKEN_A
VAR_1
VAR_2
```

### 变量定义

变量如下定义-

```shell
variable_name=variable_value
```

例如-

```shell
VAR1="Zara Ali"
VAR2=100
```

### 访问变量

shell 中有三种引用变量的方式-

+ $var
+ "$var"
+ ${var}

例如-

```shell
aaa=1
echo $aaa "$aaa" ${aaa}
#输出
1 1 1
```

### 只读变量

**readonly** 可以将一个变量标记为只读变量-

```shell
NAME="Zara Ali"
readonly NAME
NAME="Qadiri"
```

上面的脚本会产生下面的输出-

```shell
bash: NAME：只读变量
```

### 删除变量

使用 **unset** 来删除变量

```shell
unset variable_name
```

### 变量类型

当 shell 运行时，会出现三种主要的变量类型-

+ **本地变量** - 本地变量是仅出现在当前 shell 实例中的变量。它对于 shell 启动的程序是不可访问的。他们在命令提示下被设置。
+ **环境变量** - 环境变量对 shell 的任何子进程都是可访问的。某些程序需要环境变量以正常工作。 通常，shell 脚本只定义其运行的程序所需的环境变量。
+ **shell变量** -  shell 变量是一个由 shell 设置的特殊变量，shell 需要它才能正确运行。其中一些变量是环境变量，而另一些是局部变量。

## 特殊变量

在 shell 中，有一些 $ 开头的特殊变量。

| 变量 |                      含义                       |
| :--: | :---------------------------------------------: |
|  $0  |                当前脚本的文件名                 |
|  $n  | 传递给当前脚本的第 n 个参数（$1代表第一个参数） |
|  $#  |              传递给脚本的参数数量               |
|  $*  |        传递给脚本的参数列表（不包含 $0）        |
|  $@  |        传递给脚本的参数列表（不包含 $0）        |
|  $?  |              上一条命令的退出状态               |
|  $$  |                当前 shell 的 PID                |
|  $!  |              上一个后台命令的 PID               |

### $* 与 $@

不被双引号引用时，$* 和 $@ 的作用是相同的。被双引号引用时，$* 将所有参数作为一个整体，$@ 将所有参数分开处理。

以下面的脚本为例-

```shell
#test.sh
echo "\$*"
for i in $*
do
echo $i
done
echo "\"\$*\""
for i in "$*"
do
echo $i
done
echo "\$@"
for i in $@
do
echo $i
done
echo "\"\$@\""
for i in "$@"
do
echo $i
done
```

输入 `sh test.sh 111 222`

输出

```shell
$*
111
222
"$*"
111 222
$@
111
222
"$@"
111
222
```

### 退出状态

$? 代表之前命令的退出状态。

例如，执行下列脚本

```shell
exit 2
```

执行 `echo $?`

输出

```shell
2
```

## shell 数组

shell 中数组是一种关联数组，类似于其他语言中的字典-

~~~shell 
array[1]=1
array[a]="a"
array[-1]=-1
num=-1
#引用数组 ${array[index]}
echo ${array[1]} ${array[a]} ${array[-1]} ${array[$num]}
#输出
1 a -1
~~~

bash 可以通过下面的方式初始化数组-

~~~shell
array_name=(value1 ... valuen)
~~~

初始化后，数组的下标从0开始-

~~~shell
array=(1 2 3 4)
echo ${array[0]} ${array[1]} ${array[2]} ${array[3]}
#输出
1 2 3 4
~~~

可以通过 * 与 @ 获取数组中的所有值-

~~~shell
${array_name[*]}
${array_name[@]}
~~~

## 运算符

每种 shell 都支持多种操作符。以 Bourne shell 为例-

+ 算术运算符
+ 关系运算符
+ 布尔运算符
+ 字符串运算符
+ 文件测试运算符

Bourne shell 不支持任何的原生机制进行简单的算术运算，相反，它使用 awk 或者 expr 等外部命令。

下面的例子展示了如何对两个数字进行相加-

~~~shell
val=`expr 2 + 2`
echo "Total value : $val"
#输出
Total value : 4
~~~

需要注意，数字和运算符之间需要空格，`2+2`是错误的，正确的是`2 + 2`；完整的表达式需要被反引号包围。

### 算术运算符

假定两个变量 a 和 b 分别为 10 和 20。

| 运算符 |  描述  |            例子            |
| :----: | :----: | :------------------------: |
|   +    |  加法  |  `expr $a + $b` 结果是30   |
|   -    |  减法  |  `expr $a - $b`结果是-10   |
|   *    |  乘法  |  `expr $a * $b`结果是200   |
|   /    |  除法  |   `expr $b / $a`结果是2    |
|   %    |  取模  |   `expr $b % $a`结果是0    |
|   =    |  赋值  | `a = $b`会将 b 的值赋给 a  |
|   ==   |  等于  | `[ $a == $b ]`会返回 false |
|   !=   | 不等于 | `[ $a != $b ]`会返回 true  |

需要注意，条件表达式应该被中括号包围并且两边有空格，**[ $a == $b ]** 是正确的然而 **[$a==$b]** 是错误的。

所有的算术计算使用的都是长整型。

### 关系运算符

Bourne Shell 支持下列只对数值生效的关系运算符。

假定两个变量 a 和 b 分别为 10 和 20。

| 运算符 |   描述   |         例子          |
| :----: | :------: | :-------------------: |
|  -eq   |   等于   | `[ $a -eq $b ]` false |
|  -ne   |  不等于  | `[ $a -ne $b ]` true  |
|  -gt   |   大于   | `[ $a -gt $b ]` false |
|  -lt   |   小于   | `[ $a -lt $b ]` true  |
|  -ge   | 大于等于 | `[ $a -ge $b ]` false |
|  -le   | 小于等于 | `[ $a -le $b ]` true  |

同样地，与 == 以及 != 类似，条件表达式应该被中括号包围并且两边有空格，否则会出错。

### 字符串运算符

Bourne Shell 支持下列字符串运算符。

假定两个变量 a 和 b 分别是 "abc" 和 "efg"-

| 运算符 |                       描述                        |        例子         |
| :----: | :-----------------------------------------------: | :-----------------: |
|   =    |       检查两个操作数是否相等，是则返回 true       | `[ $a = $b ]` false |
|   !=   |      检查两个操作数是否相等，是则返回 false       | `[ $a != $b ]` true |
|   -z   | 检查给定的字符串操作数长度是否为零，为零返回 true |  `[ -z $a ]` false  |
|   -n   | 检查给定的字符串操作数长度是否非零，非零返回 true |  `[ -n $a ]` true   |
|  str   |      检查 str 是否是空字符串，是则返回 false      |    `[ $ ]` true     |

### 文件测试操作符

下列是测试文件各种属性的运算符。

| 运算符 |                 描述                 |
| :----: | :----------------------------------: |
|   -b   |             是否为块文件             |
|   -c   |            是否为字符文件            |
|   -d   |              是否为目录              |
|   -f   |            是否为普通文件            |
|   -g   |     是否设置 group ID（SGID）位      |
|   -k   |       是否设置粘滞（sticky）位       |
|   -p   |          是否是一个命名管道          |
|   -t   | 文件描述符是否打开并且与一个终端关联 |
|   -u   |      是否设置 user ID（SUID）位      |
|   -r   |               是否可读               |
|   -w   |               是否可写               |
|   -x   |              是否可执行              |
|   -s   |            大小是否大于0             |
|   -e   |               是否存在               |

## 分支

shell 支持两种分支语法 if else 以及 case

+ if fi
+ if else fi
+ if elif else fi
+ case esac

if 例子

```shell
# test.sh
a=$1
b=$2
echo a=$a,b=$b
if [ $a -gt $b ]
then
	echo "a > b"
elif [ $a -eq $b ]
then
	echo "a = b"
else
	echo "a < b"
fi
```

case 例子

语法

~~~shell
case word in
   pattern1)
      Statement(s) to be executed if pattern1 matches
      ;;
   pattern2)
      Statement(s) to be executed if pattern2 matches
      ;;
   pattern3)
      Statement(s) to be executed if pattern3 matches
      ;;
   *)
     Default condition to be executed
     ;;
esac
~~~

例子

~~~~shell
option="${1}" 
case ${option} in 
   -f) FILE="${2}" 
      echo "File name is $FILE"
      ;; 
   -d) DIR="${2}" 
      echo "Dir name is $DIR"
      ;; 
   *)  
      echo "`basename ${0}`:usage: [-f file] | [-d directory]" 
      exit 1 # Command to come out of the program with status 1
      ;; 
esac 
~~~~

## 循环

## 参考文献

[shell tutorial](https://www.tutorialspoint.com/unix/index.htm)