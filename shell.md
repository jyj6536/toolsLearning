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

## 参考文献

[shell tutorial](https://www.tutorialspoint.com/unix/index.htm)