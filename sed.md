# sed

sed 是 GNU/Linux 上最著名的文本处理工具之一。类似于许多其他的 GNU/Linux 工具，sed 是面向流的并且使用简单的编程语言。sed 可以通过数行代码解决复杂的文本处理任务。

## 概述

缩写 SED 代表 **Stream EDitor**，它是一个简单而强大的实用程序，可以解析文本并无缝转换。SED 由贝尔实验室的 Lee E. McMahon 开发，可以在所有主要的操作系统上运行。

McMahon编写了一个面向行的通用编辑器，并最终演变为了 SED。SED 借鉴了 ed 编辑器的语法和许多其他有用的特性。 诞生伊始，SED 已支持正则表达式。SED 从文件以及管道接收输入。此外，SED 也可以从标准输入流接收输入。

SED 由自由软件基金会（FSF）编写并维护，由 GNU/Linux 进行分发。因此它通常指的是 **GNU SED**。对于一个菜鸟用户，SED 的语法可能看起来很晦涩难懂。然而，一旦熟悉了 SED 的语法，用户可以通过数行 SED 脚本解决许多复杂的任务。

## 工作流

~~~flow
oprd=>operation: 从输入流中读取一行
opex=>operation: 在读取的行上执行 sed 命令
opshow=>operation: 在输出流上展示结果

oprd->opex->opshow
~~~

SED 遵循简单的工作流：读取，执行，显示。

+ **Read**：SED 从输入流读取一行（文件，管道或者标准输入）并且将其存储在被称为**模式缓冲区**的内部缓冲区之中。
+ **Execute**：所有的 SED 命令顺序应用于模式缓冲区。默认情况下除非指定了行地址，SED 命令会应用到所有行（全局）。
+ **Display**：发送（处理后的）内容到标准输出流。发送数据之后，模式缓冲区会被清空。

重复上述流程直到文件被读取完毕。

### 要点

+ 模式缓冲区是一个 SED 。
+ 默认地，所有的 SED 命令都应用于模式缓冲区，因此输入文件保持不变。GNU SED 提供了就地修改输入文件的方法。
+ 存在另一个专用的，内存中的，易失的存储区域被称为**保存缓冲区**。数据可以被保存在保存缓冲区中以备后续检索。在每次循环结束，模式缓冲区的内容被 SED 移除，但是保存缓冲区的内容在 SED 循环之间持续存在。然而，SED 命令不能在保存缓冲区上直接执行，因此 SED 允许在保存缓冲区和模式缓冲区之间进行数据移动。
+ 模式缓冲区和保存缓冲区均初始化为空。
+ 如果没有指定输入文件，那么 SED 从标准输入流接收输入（stdin）。
+ 如果没有提供地址范围，那么 SED 处理每一行。

### 例子

创建一个名为 **quote.txt** 的文本文件并写入一下内容：

~~~shell
There is only one thing that makes a dream impossible to achieve: the fear of failure. 
 - Paulo Coelho, The Alchemist
~~~

模拟 **cat** 命令。

~~~shell
$ sed '' quote.txt 
There is only one thing that makes a dream impossible to achieve: the fear of failure. 
 - Paulo Coelho, The Alchemist
~~~

## 基本语法

本节列出了 SED 支持的基本命令以及命令行语法。SED 可以通过一下两种方式调用：

~~~shell
sed [-n] [-e] 'command(s)' files 
sed [-n] -f scriptfile files
~~~

第一种方式在行内指定命令并且命令被单引号包围。第二种方式允许指定一个包含 SED 命令的脚本文件。然而，我们可以同时多次使用这两种方式。SED 提供各种命令行选项来控制其行为。

假定有一个文本文件 **books.txt**，内容如下：

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

现在，我们要删除第一、二、五行，使用 **-e** 选项。

~~~shell
$ sed -e '1d' -e '2d' -e '5d' books.txt 
~~~

执行之后的文件内容：

~~~shell
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
6) A Game of Thrones, George R. R. Martin, 864 
~~~

此外，还可以在文本文件中指定 SED 命令。

创建一个 **commands.txt** 并写入以下内容：

~~~shell
$ echo -e "1d\n2d\n5d" > commands.txt 
~~~

使用 **-f** 选项执行：

~~~shell
$ sed -f commands.txt books.txt
~~~

执行之后的文件内容：

~~~shell
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
6) A Game of Thrones,George R. R. Martin, 864 
~~~

### 标准选项

SED 支持下列标准选项：

+ **-n**：默认输出模式缓冲区（仅输出处理后的结果）。例如，下面的 SED 命令不会输出任何内容：

  ~~~shell
  $ sed -n '' quote.txt 
  ~~~

+ **-e**：下一个参数是一条编辑命令。 这里，尖括号表示强制参数。通过使用这个选项，可以指定多个命令。例如，每行打印两次：

  ~~~shell
  $ sed -e '' -e 'p' quote.txt
  ~~~

  输出：

  ~~~shell
  There is only one thing that makes a dream impossible to achieve: the fear of failure. 
  There is only one thing that makes a dream impossible to achieve: the fear of failure. 
   - Paulo Coelho, The Alchemist
   - Paulo Coelho, The Alchemist
  ~~~

+ **-f**：下一个参数是一个包含编辑命令的文件。尖括号表示强制参数。下面的例子通过文件指定打印命令：

  ~~~shell
  echo "p" > commands
  sed -n -f commands quote.txt
  ~~~

### GNU 特定选项

下列选项是 GNU 特定的，可能不被其他的 SED 变体所支持。

+ **-n, --quiet, --silent**：与标准 **-n** 选项类似。

+ **-e script, --expression=script**：与标准 **-e** 选项类似。

+ **-f script-file, --file=script-file**：与标准 **-f** 选项类似。

+ **--follow-symlinks**：如果提供该选项，SED 会在原地编辑文件时跟随符号连接。

+ **-i[SUFFIX], --in-place[=SUFFIX]**：该选项用于原地编辑文件。如果提供 suffix，SED 会对原始文件进行备份，否则覆盖原始文件。

+ **-l N, --line-length=N**：设定每一行的长度为 N 个字符。例如：

  ~~~shell
  echo 'abcdefg' | sed -n -l 4 'l'
  abc\
  def\
  g$
  ~~~

+ **--posix**：禁用所有 GNU 扩展。

+ **-r, --regexp-extended**：允许使用扩展而不是基本正则表达式。

+ **-u, --unbuffered**：如果提供该选项，SED 从输入文件加载最少的数据并且更频繁地刷新输出缓冲区。如果在编辑 “tail -f”的输出时不想等待输出的话这是非常有用的。

+ **-z, --null-data**：默认地，SED通过换行符分割每一行。如果提供了 NULL-data 选项，SED 通过 NULL 字符对行进行分割。

## 分支

类似于其他程序语言，SED 也提供了循环和分支能力以控制执行流程。SED中的循环类似于goto语句。SED 可以跳转到被标签标记的行并且继续执行剩余命令。在 SED 中，可以像下面这样定义标签：

~~~shell
:label 
:start 
:end 
:up
~~~

要跳转到特定的标签，可以使用命令 **b** 后跟一个标签名。如果标签名被省略，那么 SED 跳转到 SED 文件的结尾。

以 books.txt 为例，如果某一行匹配模式 Paulo，则在该行的前面打印连字符（-），否则打印改行本身：

~~~shell
$ sed -n ' 
/Paulo/!b Print 
s/^/- / 
:Print 
p' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
- 3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
- 5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

可以将所有命令写在一行之中：

~~~shell
sed -n '/Paulo/!b Print; s/^/- /; :Print;p' books.txt
~~~

## 循环

可以使用 **t** 命令创建循环。只有在上一个替换命令成功时 **t** 命令才会跳转到指定标签。现在，我们在匹配 Paulo 的行前打印四个连字符（-）。

~~~shell
$ sed -n '  
:Loop 
/Paulo/s/^/-/ 
/----/!t Loop 
p' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
----3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
----5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

同样地，上述命令可以写在一行之中：

~~~shell
$ sed -n ':Loop;/Paulo/s/^/-/; /----/!t Loop; p' books.txt
~~~

## 地址范围

仍然以 books.txt 为例介绍 **print** 命令。

~~~shell
$ sed 'p' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
6) A Game of Thrones, George R. R. Martin, 864
~~~

默认地，SED 打印模式缓冲区的内容。此外，我们在命令部分中明确包含了一个打印命令。因此每行打印了两次。**-n** 选项可以可以禁止模式缓冲区的默认打印。下面的命令说明了这一点。

~~~shell
$ sed -n 'p' books.txt 
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

可以通过限定地址范围以打印特定的行。

| 命令 |               含义               |
| :--: | :------------------------------: |
|  3   |              第三行              |
| 2,5  |           第二到第五行           |
|  $   |             最后一行             |
| 3,$  |         第三行到最后一行         |
| 2,+4 |        第二行及其后的四行        |
| 1~2  | 第一行开始每两行（1，3，5，...） |
| 2~2  | 第二行开始每两行（2，4，6，...） |

例如，打印奇数行：

~~~shell
$ sed -n '1~2 p' books.txt 
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
3) The Alchemist, Paulo Coelho, 197 
5) The Pilgrimage, Paulo Coelho, 288
~~~

## 模式范围

模式范围可以是一个简单文本或者复杂的正则表达式。比如，匹配作者 Paulo Coelho：

~~~shell
$ sed -n '/Paulo/ p' books.txt
~~~

输出

~~~shell
3) The Alchemist, Paulo Coelho, 197 
5) The Pilgrimage, Paulo Coelho, 288
~~~

可以将模式范围与地址范围组合使用：

~~~shell
$ sed -n '/Alchemist/, 5 p' books.txt
~~~

从匹配 Alchemist 的行一直打印到第五行：

~~~shell
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288
~~~

也可以使用逗号分隔两个模式范围，例如：

~~~shell
$sed -n '/Two/, /Pilgrimage/ p' books.txt 
~~~

输出：

~~~shell
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
~~~

## 基本命令

### 删除命令

**d** 表示删除指定行的命令，与范围命令结合可以删除特定的行。

例子：

删除第四行

~~~shell
$ sed '4d' books.txt 
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

删除 **Storm** 与 **Alchemist** 之间的行

~~~shell
$ sed '/Storm/,/Alchemist/d' books.txt
~~~

输出

~~~shell
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

### 写入命令

**w** 命令可以将特定的行写入文件。

例子：

~~~shell
$ sed -n '2~2 w junk.txt' books.txt
~~~

输出 junk.txt 的内容

~~~shell
2) The Two Towers, J. R. R. Tolkien, 352 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
6) A Game of Thrones, George R. R. Martin, 864
~~~

### 追加命令

**a** 命令在匹配的行之后进行追加操作。

例子：

在第四行之后追加

~~~shell
$ sed '4 a 7) Adultry, Paulo Coelho, 234' books.txt
~~~

追加后的 books.txt

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
7) Adultry, Paulo Coelho, 234 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

可以通过模式匹配进行追加，如果多次匹配则多次追加

~~~shell
$ sed '/The/ a 7) Adultry, Paulo Coelho, 234' books.txt 
~~~

追加后的 books.txt

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
7) Adultry, Paulo Coelho, 234 
3) The Alchemist, Paulo Coelho, 197 
7) Adultry, Paulo Coelho, 234 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
7) Adultry, Paulo Coelho, 234 
5) The Pilgrimage, Paulo Coelho, 288 
7) Adultry, Paulo Coelho, 234 
6) A Game of Thrones, George R. R. Martin, 864 
~~~

### 替换命令

**c** 命令使用文本替换现有行。

例子：

使用新文本替换第三行

~~~shell
$ sed '3 c 3) Adultry, Paulo Coelho, 324' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) Adultry, Paulo Coelho, 324
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

多行会被作为整体进行替换

~~~shell
$ sed '4, 6 c 4) Adultry, Paulo Coelho, 324' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) Adultry, Paulo Coelho, 324
~~~

### 插入命令

**i** 与 **a**，区别在于 **i** 是在指定的行之前插入。

例子：

在第四行之前插入一行

~~~shell
$ sed '4 i 7) Adultry, Paulo Coelho, 324' books.txt 
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
7) Adultry, Paulo Coelho, 324
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

在最后一行之前插入多行

~~~shell
$ sed '$ i 7) Adultry, Paulo Coelho, 324\
8) Eleven Minutes, Paulo Coelho, 304' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
7) Adultry, Paulo Coelho, 324
8) Eleven Minutes, Paulo Coelho, 304
6) A Game of Thrones, George R. R. Martin, 864
~~~

### 转换命令

**y** 命令可以对字符进行转换。

~~~shell
[address1[,address2]]y/list-1/list-2/
~~~

注意，转换是基于 **list 1** 中与 **list 2** 对应位置的字符进行的并且这两个列表必须是明确的字符列表。不支持正则表达式和字符类。此外，**list 1** 和 **list 2** 的长度必须保持一致。

例子：

~~~shell
$ echo "1 5 15 20" | sed 'y/151520/IVXVXX/'
~~~

输出

~~~shell
I V IV XX
~~~

### l命令

**l** 命令用来显示不可见的字符。例如，制表符用 \t 替代，行结束符用 $ 替代。

例子：

~~~shell
sed 's/ /\t/g' books.txt|sed -n 'l'
~~~

输出

~~~shell
1)\tA\tStorm\tof\tSwords,\tGeorge\tR.\tR.\tMartin,\t1216\t$
2)\tThe\tTwo\tTowers,\tJ.\tR.\tR.\tTolkien,\t352\t$
3)\tThe\tAlchemist,\tPaulo\tCoelho,\t197\t$
4)\tThe\tFellowship\tof\tthe\tRing,\tJ.\tR.\tR.\tTolkien,\t432\t$
5)\tThe\tPilgrimage,\tPaulo\tCoelho,\t288\t$
6)\tA\tGame\tof\tThrones,\tGeorge\tR.\tR.\tMartin,\t864$
~~~

**l** 后可以接受一个参数 n，指示 SED 在一定数量的字符后执行换行。

~~~shell
$ sed -n 'l 25' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, Ge\
orge R. R. Martin, 1216 $
2) The Two Towers, J. R.\
 R. Tolkien, 352 $
3) The Alchemist, Paulo \
Coelho, 197 $
4) The Fellowship of the\
 Ring, J. R. R. Tolkien,\
 432 $
5) The Pilgrimage, Paulo\
 Coelho, 288 $
6) A Game of Thrones, Ge\
orge R. R. Martin, 864$
~~~

该特性是 GNU 扩展在其他的 SED 变体上可能无法正常工作。0代表只有在遇到换行符时才截断。

### 退出命令

**q** 命令指示 SED 退出当前的执行流。

注意，**q** 命令不接受范围地址，仅支持单个地址。默认地，SED 遵循读取，执行，重复的工作流；但是当遇到退出命令时，SED 简单地退出当前执行过程。

例子：

~~~shell
$ sed '3 q' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197
~~~

退出命令可以接受一个参数 value 作为退出值

~~~shell
$ sed '/The Alchemist/ q 100' books.txt
~~~

获取退出状态

~~~shell
$ echo $?
~~~

输出

~~~shell
100
~~~

### 读取命令

当满足特定条件时，我们可以指示 SED 读取文件内容并展示。读取命令是 **r**。

例子：

~~~shell
$ echo "This is junk text." > junk.txt
$ sed '3 r junk.txt' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
This is junk text.
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

对于 GNU SED，读取命令可以接受范围地址

~~~shell
$ sed '3, 5 r junk.txt' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
This is junk text.
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
This is junk text.
5) The Pilgrimage, Paulo Coelho, 288 
This is junk text.
6) A Game of Thrones, George R. R. Martin, 864
~~~

### 执行命令

可以使用执行命令从 SED 中执行外部命令。执行命令由 **e** 表示。

例子：

~~~shell
$ sed '3 e date' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
Fri Feb  3 11:58:16 AM UTC 2023
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

当 **e** 的参数 command 没有指定时，SED 将模式缓冲区的内容作为命令

~~~shell
$ echo -e "date\nuname"|sed 'e'
~~~

输出

~~~shell
Fri Feb  3 12:00:49 PM UTC 2023
Linux
~~~

### 杂项命令

默认地，SED 在单个行上进行操作，然而 SED 也可以在多行上进行操作。多行命令由大写字母表示。例如，与 **n** 命令不同，**N** 不会清空并打印模式缓冲空间。相反，它会在当前的模式缓冲区的末尾添加一个换行符（\n），从输入文件中读取下一行并追加到当前的模式缓冲区，执行剩余的 SED 命令以继续 SED 的标准流。

**N** 命令的语法如下

~~~shell
[address1[,address2]]N
~~~

假如有以下输入

~~~shell
 A Storm of Swords 
  George R. R. Martin
 The Two Towers 
  J. R. R. Tolkien
 The Alchemist 
  Paulo Coelho
 The Fellowship of the Ring 
  J. R. R. Tolkien
 The Pilgrimage 
  Paulo Coelho
 A Game of Thrones 
  George R. R. Martin
~~~

使用 SED 进行处理

~~~shell
$ sed 'N; s/\n/, /g'
~~~

输出

~~~shell
 A Storm of Swords ,  George R. R. Martin
 The Two Towers ,  J. R. R. Tolkien
 The Alchemist ,  Paulo Coelho
 The Fellowship of the Ring ,  J. R. R. Tolkien
 The Pilgrimage ,  Paulo Coelho
 A Game of Thrones ,  George R. R. Martin
~~~

类似于 **p** 命令，我们有 **P** 命令打印有 **N** 创建的多行的模式空间中的第一部分（直到嵌入的换行符）。

**P** 命令的语法如下

~~~shell
[address1[,address2]]P
~~~

仍然以上面的例子为例

~~~shell
$ sed -n 'N;P'
~~~

输出

~~~shell
 A Storm of Swords 
 The Two Towers 
 The Alchemist 
 The Fellowship of the Ring 
 The Pilgrimage 
 A Game of Thrones 
~~~

SED 提供了 **v** 命令以进行版本检查。如果提供的版本比安装的版本高，那么命令会失败。该选项是 GNU 特定的，对于其他 SED 变体可能无法正常工作。

**v** 命令的语法如下

~~~shell
[address1[,address2]]v
~~~

首先，查看 SED 版本

```shell
$sed --version
```

输出

```shell
sed (GNU sed) 4.8
由 Debian 打包
Copyright (C) 2020 Free Software Foundation, Inc.
许可证 GPLv3+：GNU 通用公共许可证第 3 版或更新版本<https://gnu.org/licenses/gpl.html>。
本软件是自由软件：您可以自由修改和重新发布它。
在法律范围内没有其他保证。

由 Jay Fenlason、Tom Lord、Ken Pizzini、
Paolo Bonzini、Jim Meyering 和 Assaf Gordon 编写。

本 sed 程序构建时含有 SELinux 支持。
此系统已禁用 SELinux。

GNU sed 主页：<https://www.gnu.org/software/sed/>。
使用 GNU 软件的一般性帮助：<https://www.gnu.org/gethelp/>。
请将错误报告发送至：<bug-sed@gnu.org>。

```

指定更高的版本

~~~shell
$ sed 'v 4.8' books.txt
~~~

输出

~~~shell
sed: -e 表达式 #1, 字符 7: 需要更高版本的sed↵
~~~

## 特殊字符

SED 提供了作为命令的两个特殊字符。

### = 命令

**=** 命令打印行号。

语法

~~~shell
[/pattern/]= 
[address1[,address2]]=
~~~

例子

~~~shell
$ sed '=' books.txt 
~~~

输出

~~~shell
1
1) A Storm of Swords, George R. R. Martin, 1216 
2
2) The Two Towers, J. R. R. Tolkien, 352 
3
3) The Alchemist, Paulo Coelho, 197 
4
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5
5) The Pilgrimage, Paulo Coelho, 288 
6
6) A Game of Thrones, George R. R. Martin, 864
~~~

### & 命令

无论何时，当一个模式匹配成功时，**&** 保存匹配成功的模式。**&** 通常与替换命令一起使用。

例子

~~~shell
$ sed 's/[[:digit:]]/Book number &/' books.txt
~~~

输出

~~~shell
Book number 1) A Storm of Swords, George R. R. Martin, 1216 
Book number 2) The Two Towers, J. R. R. Tolkien, 352 
Book number 3) The Alchemist, Paulo Coelho, 197 
Book number 4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
Book number 5) The Pilgrimage, Paulo Coelho, 288 
Book number 6) A Game of Thrones, George R. R. Martin, 864
~~~

## 字符串

### 替换命令

语法

~~~shell
[address1[,address2]]s/pattern/replacement/[flags]
~~~

将 books.txt 中的逗号转换为竖线

~~~shell
$ sed 's/,/ | /' books.txt
~~~

输出

~~~shell
1) A Storm of Swords |  George R. R. Martin, 1216 
2) The Two Towers |  J. R. R. Tolkien, 352 
3) The Alchemist |  Paulo Coelho, 197 
4) The Fellowship of the Ring |  J. R. R. Tolkien, 432 
5) The Pilgrimage |  Paulo Coelho, 288 
6) A Game of Thrones |  George R. R. Martin, 864
~~~

上面的例子只替换了第一个找到的逗号，这是 SED 的默认行为，可以通过全局标志（g）将所有逗号进行替换

~~~shell
$ sed 's/,/ | /g' books.txt
~~~

输出

~~~shell
1) A Storm of Swords |  George R. R. Martin |  1216 
2) The Two Towers |  J. R. R. Tolkien |  352 
3) The Alchemist |  Paulo Coelho |  197 
4) The Fellowship of the Ring |  J. R. R. Tolkien |  432 
5) The Pilgrimage |  Paulo Coelho |  288 
6) A Game of Thrones |  George R. R. Martin |  864
~~~

仅在模式匹配成功时进行替换

~~~shell
$ sed '/The Pilgrimage/ s/,/ | /g' books.txt 
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage |  Paulo Coelho |  288 
6) A Game of Thrones, George R. R. Martin, 864
~~~

此外，SED 可以只替换某个特定出现次数的模式。下面的例子替换第二次出现的逗号

~~~shell
$ sed 's/,/ | /2' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin |  1216 
2) The Two Towers, J. R. R. Tolkien |  352 
3) The Alchemist, Paulo Coelho |  197 
4) The Fellowship of the Ring, J. R. R. Tolkien |  432 
5) The Pilgrimage, Paulo Coelho |  288 
6) A Game of Thrones, George R. R. Martin |  864
~~~

标志 **p** 可以打印发生替换操作的行

~~~shell
$ sed -n 's/Paulo Coelho/PAULO COELHO/p' books.txt
~~~

输出

~~~shell
3) The Alchemist, PAULO COELHO, 197 
5) The Pilgrimage, PAULO COELHO, 288
~~~

标志 **w** 可以将发生变化的行存储在指定文件中

~~~shell
$ sed -n 's/Paulo Coelho/PAULO COELHO/w junk.txt' books.txt
~~~

执行后生成 **junk.txt**

~~~shell
$ ls
books.txt  junk.txt
~~~

标志 **i** 可以忽略大小写

~~~shell
$ sed  -n 's/pAuLo CoElHo/PAULO COELHO/pi' books.txt
~~~

输出

~~~shell
3) The Alchemist, PAULO COELHO, 197 
5) The Pilgrimage, PAULO COELHO, 288
~~~

除了反斜杠以外，SED 支持其他分隔符

例子

~~~shell
$ echo "/bin/sed" | sed 's/\/bin\/sed/\/home\/jerry\/src\/sed\/sed-4.2.2\/sed/'
$ echo "/bin/sed" | sed 's|/bin/sed|/home/jerry/src/sed/sed-4.2.2/sed|'
$ echo "/bin/sed" | sed 's@/bin/sed@/home/jerry/src/sed/sed-4.2.2/sed@'
$ echo "/bin/sed" | sed 's^/bin/sed^/home/jerry/src/sed/sed-4.2.2/sed^'
~~~

输出

~~~shell
/home/jerry/src/sed/sed-4.2.2/sed
~~~

### 创建子串

SED 支持从匹配的文本中创建子串。

~~~shell
echo "Three One Two" | sed 's|\(\w\+\) \(\w\+\) \(\w\+\)|\2 \3 \1|'
~~~

其中，**\\w\\+** 匹配单个单词，括号进行了分组

输出

~~~shell
One,Two,Three
~~~

### 字符串替换标志（仅适用于 GNU SED）

+ **\\L** 如果替换字符串中指定了 **\\L**，那么 **\\L**之后的所有字符都会被当作小写字符。

  ~~~shell
  $ sed -n 's/Paulo/PA\LULO/p' books.txt、
  #输出
  3) The Alchemist, PAulo Coelho, 197 
  5) The Pilgrimage, PAulo Coelho, 288
  ~~~

+ **\\u** 如果替换字符串中指定了 **\\u**，那么 **\\u** 之后的第一个字符会被当作大写字符。

  ~~~shell
  $ sed -n 's/Paulo/p\uaul\uo/p' books.txt
  #输出
  3) The Alchemist, pAulO Coelho, 197 
  5) The Pilgrimage, pAulO Coelho, 288
  ~~~

+ **\\U** 如果替换字符串中指定了 **\\U**，那么 **\\U** 之后的所有字符都会被当作大写字符。

  ~~~shell
  $ sed -n 's/Paulo/\Upaulo/p' books.txt 
  #输出
  3) The Alchemist, PAULO Coelho, 197 
  5) The Pilgrimage, PAULO Coelho, 288 
  ~~~

+ **\\E** 与 **\\L** 或 **\\U** 联合使用，停止它们的转换。

  ~~~shell
  $sed -n 's/Paulo Coelho/\Upaulo \Ecoelho/p' books.txt
  #输出
  3) The Alchemist, PAULO coelho, 197 
  5) The Pilgrimage, PAULO coelho, 288 
  $ sed -n 's/Paulo Coelho/\Upaulo coelho/p' books.txt
  #输出
  3) The Alchemist, PAULO COELHO, 197 
  5) The Pilgrimage, PAULO COELHO, 288 
  ~~~

## 模式管理

### n 命令

语法

```shell
[address1[,address2]]n
```

例子

```shell
$ sed 'n' books.txt 
```

输出

```shell
1) A Storm of Swords, George R. R. Martin, 1216 
2) The Two Towers, J. R. R. Tolkien, 352 
3) The Alchemist, Paulo Coelho, 197 
4) The Fellowship of the Ring, J. R. R. Tolkien, 432 
5) The Pilgrimage, Paulo Coelho, 288 
6) A Game of Thrones, George R. R. Martin, 864 
```

**n** 命令打印模式缓冲区的内容，清空模式缓冲区，将下一行放入模式缓冲区，并对其应用命令。

现在，假定 **n** 之前由三条 SED 命令，**n** 之后由两条 SED 命令。

~~~shell
Sed command #1 
Sed command #2 
Sed command #3 
n command 
Sed command #4 
Sed command #5
~~~

首先，SED 对模式缓冲区应用前三条命令，然后清空模式缓冲区并读取下一行，并且继续对模式缓冲区应用剩余的两条命令。

SED 命令不能直接作用于保存缓冲区的内容。因此，如果想对其进行操作，需要将其放入模式缓冲区。**x** 命令可以交换模式缓冲区和保存缓冲区的内容。

例子

~~~shell
sed -n 'x;n;x;p' books.txt
~~~

输出

~~~shell
1) A Storm of Swords, George R. R. Martin, 1216 
3) The Alchemist, Paulo Coelho, 197 
5) The Pilgrimage, Paulo Coelho, 288
~~~

上面的例子打印了所有的奇数行。

**h** 命令处理保存缓冲区，它将模式缓冲区的内容拷贝到保存缓冲区。

语法

```shell
[address1[,address2]]h 
```

以下面的文本为例进行说明

~~~shell
A Storm of Swords 
George R. R. Martin 
The Two Towers 
J. R. R. Tolkien 
The Alchemist 
Paulo Coelho 
The Fellowship of the Ring 
J. R. R. Tolkien 
The Pilgrimage 
Paulo Coelho 
A Game of Thrones 
George R. R. Martin 
~~~

例子

```shell
$ sed -n '/Paulo/!h; /Paulo/{x;p}'
```

输出

```shell
The Alchemist 
The Pilgrimage
```

首先，如果不匹配 Paulo，则将当前内容放入保存缓冲区；如果匹配 Paulo，则交换并打印。该命令打印了作者 Paulo 的所有作品。

与 **h** 不同，**H** 命令将换行符和模式缓冲区中的内容追加到保存缓冲区中。

语法

```shell
[address1[,address2]]H
```

仍然以上面的文本为例

~~~shell
sed -n '/Paulo/!h; /Paulo/{H;x};s/\n/,/p'
~~~

输出

~~~shell
The Alchemist ,Paulo Coelho 
The Pilgrimage ,Paulo Coelho
~~~

## 参考文献

[Sed Tutorial](https://www.tutorialspoint.com/sed/index.htm)