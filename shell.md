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
|  =/==  |       检查两个操作数是否相等，是则返回 true       | `[ $a = $b ]` false |
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

### 单中括号与双中括号

~~~
单中括号 [ ] 
a. [ ] 两个符号左右都要有空格分隔 
b. 内部操作符与操作变量之间要有空格：如 [ "$a" = "$b" ] 
c. 字符串比较中，> < 需要写成\> \< 进行转义 
d. [ ] 中字符串或者${}变量尽量使用双引号扩住，以避免值未定义引用而出错 
e. [ ] 中可以使用 –a –o 进行逻辑运算 

双中括号 
a. [[ ]] 两个符号左右都要有空格分隔 
b. 内部操作符与操作变量之间要有空格：如 [[ "$a" = "$b" ]] 
c. 字符串比较中，可以直接使用 > < 无需转义 
d. [[ ]] 中字符串或者${}变量尽量使用双引号扩住，如未使用双引号会进行模式和元字符匹配 
e. [[ ]] 内部可以使用 && || 进行逻辑运算 
~~~

结论：尽量使用双中括号

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

shell 中有四种循环-

+ while
+ for
+ until
+ select

### while 循环

```shell
a=0

while [ $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

### for 循环

```shell
for var in 0 1 2 3 4 5 6 7 8 9
do
   echo $var
done
```

### until 循环

```shell
a=0

until [ ! $a -lt 10 ]
do
   echo $a
   a=`expr $a + 1`
done
```

输出

```shell
0
1
2
3
4
5
6
7
8
9
```

### select 循环

select 循环在 ksh 中引入并且已经适配 bash，但是在 sh 中是不可用的。

select 可以方便与用户的交互。

```shell
echo "What is your favourite OS?"
select name in "Linux" "Windows" "Mac OS" "UNIX" "Android"
do
    case $name in
        "Linux")
            echo "Linux是一个类UNIX操作系统，它开源免费，运行在各种服务器设备和嵌入式设备。"
            break
            ;;
        "Windows")
            echo "Windows是微软开发的个人电脑操作系统，它是闭源收费的。"
            break
            ;;
        "Mac OS")
            echo "Mac OS是苹果公司基于UNIX开发的一款图形界面操作系统，只能运行与苹果提供的硬件之上。"
            break
            ;;
        "UNIX")
            echo "UNIX是操作系统的开山鼻祖，现在已经逐渐退出历史舞台，只应用在特殊场合。"
            break
            ;;
        "Android")
            echo "Android是由Google开发的手机操作系统，目前已经占据了70%的市场份额。"
            break
            ;;
        *)
            echo "输入错误，请重新输入"
    esac
done
```

输出

```shell
What is your favourite OS?
1) Linux
2) Windows
3) Mac OS
4) UNIX
5) Android
#? 1
Linux是一个类UNIX操作系统，它开源免费，运行在各种服务器设备和嵌入式设备。
```

`#?`用来提示用户输入菜单编号。如果用户的输入是和合法的，`name` 会被赋值为对应的值，否则会被赋值为空；空输入会导致 select 重新打印选择列表；select 是死循环，只有 Ctrl+D 或者 break 才会退出循环。

### 循环控制 

break 与 continue 用来控制循环流程。

+ break 退出本层循环
+ break n 退出第 n 层嵌套循环
+ continue 退出当前循环的本次迭代
+ continue n 退出第 n 层嵌套循环的本次迭代

## 替换

 shell 在遇到包含一个或多个特殊字符的表达式时执行替换。 

下列是 echo 命令中的转义序列。

| 序号 | 序列 | 描述           |
| ---- | ---- | -------------- |
| 1    | \\\\ | 反斜杠         |
| 2    | \a   | 警告（BEL）    |
| 3    | \b   | 退格           |
| 4    | \c   | 禁止行尾的换行 |
| 5    | \f   | 换页           |
| 6    | \n   | 换行           |
| 7    | \r   | 回车           |
| 8    | \t   | 水平制表符     |
| 9    | \v   | 垂直制表符     |

反引号可以将命令的输出赋值给一个变量。

```shell
DATE=`date`
echo $DATE
#输出
2023年 03月 02日 星期四 19:42:33 CST
```

变量替换允许 shell 程序员根据变量状态控制变量的值。

| 序号 | 格式                | 描述                                                         |
| ---- | ------------------- | ------------------------------------------------------------ |
| 1    | **${var}**          | 以 var 的值进行替换。                                        |
| 2    | **${var:-word}**    | 如果 var 的值是 null 或者未定义，使用 *word* 替换 var。      |
| 3    | **${var:=word}**    | 如果 var 的值是 null 或者未定义，使用 *word* 替换 var。var 会被设置为 *word*。 |
| 4    | **${var:?message}** | 如果 var 的值是 null 或者未定义，message 会被输出到标准输出。用来检查 var 是否被正确设置。 |
| 5    | **${var:+word}**    | 如果 var 被设置，*word* 用来替换 var 的值。var 的值不会改变。 |

## 引用

Unix shell 它提供了一些元字符。

下列是常用的 shell 元字符-

```shell
* ? [ ] ' " \ $ ; & ( ) | ^ < > new-line space tab
```

| 序号 | 引用   | 含义                                                         |
| ---- | ------ | ------------------------------------------------------------ |
| 1    | 单引号 | 所有被单引号引用的字符失去他们的特殊含义                     |
| 2    | 双引号 | 大多数特殊字符失去他们的特殊含义，除了以下字符 $ ` \\$ 反引号 \\' \\" \\\\ |
| 3    | 反斜杠 | 任何紧跟反斜杠的字符失去其特殊含义                           |
| 4    | 反引号 | 反引号之间的字符会被当作命令执行                             |

## I/O 重定向

| 序号 | 命令        | 描述                       |
| ---- | ----------- | -------------------------- |
| 1    | cmd > file  | 输出重定向到 file          |
| 2    | cmd < file  | 从 file 读取输入           |
| 3    | cmd >> file | 输出追加到 file            |
| 4    | n > file    | 描述符 n 重定向到 file     |
| 5    | n >> file   | 描述符 n 追加到 file       |
| 6    | n > &m      | 将 n 合并到 m 输出         |
| 7    | n <& m      | 将 n 合并到 m 输入         |
| 8    | << tag      | 从标准输入读取直到遇到 tag |
| 9    | \|          | 管道                       |

丢弃输出

```shell
$ command > /dev/null
```

标准错误重定向到标准输出并丢弃

```shell
$ command > /dev/null 2>&1
```

将标准输出重定向到标准错误

```shell
$ echo message 1>&2
```

通常，描述符 0 代表标准输入，1 代表标准输出，2 代表标准错误。

## 函数

定义函数-

```shell
function_name () { 
   list of commands
}
# 定义函数
Hello () {
   echo "Hello World"
}
# 调用函数
Hello
```

传递参数-

可以给函数传递参数，以 $1 $2 为代表

```shell
Hello () {
   echo "Hello World $1 $2"
}

# 调用函数
Hello Zara Ali
```

返回值-

````shell
return code
````

通过 return 进行返回，$?获取返回值。

```shell
# Define your function here
Hello () {
   echo "Hello World $1 $2"
   return 10
}

# Invoke your function
Hello Zara Ali

# Capture value returnd by last command
ret=$?

echo "Return value is $ret"
```

嵌套调用-

shell 函数支持嵌套调用以及递归

```shell
hello (){
if [ $1 -gt 0 ]
then
hello `expr $1 - 1`
echo $1
return 0
fi
echo $1
}
hello 10
# 输出
0
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
```

## 参考文献 

[shell tutorial](https://www.tutorialspoint.com/unix/index.htm)

[Bash Reference Manual](https://www.gnu.org/software/bash/manual/bash.html)

[shell教程](http://c.biancheng.net/shell/base/)