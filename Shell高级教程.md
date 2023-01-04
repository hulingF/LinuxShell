# Shell高级教程

## 1.Linux Shell重定向

Linux Shell 重定向分为两种，一种输入重定向，一种是输出重定向；从字面上理解，输入输出重定向就是「改变输入与输出的方向」的意思。

那么，什么是输入输出方向呢？标准的输入输出方向又是什么呢？

一般情况下，我们都是从键盘读取用户输入的数据，然后再把数据拿到程序（C语言程序、Shell 脚本程序等）中使用；这就是标准的输入方向，也就是从键盘到程序。反过来说，程序中也会产生数据，这些数据一般都是直接呈现到显示器上，这就是标准的输出方向，也就是从程序到显示器。

我们可以把观点提炼一下，其实输入输出方向就是数据的流动方向：

- 输入方向就是数据从哪里流向程序。`数据默认从键盘流向程序，如果改变了它的方向，数据就从其它地方流入，这就是输入重定向。`

- 输出方向就是数据从程序流向哪里。`数据默认从程序流向显示器，如果改变了它的方向，数据就流向其它地方，这就是输出重定向。`

### 1.1硬件设备和文件描述符

计算机的硬件设备有很多，常见的输入设备有键盘、鼠标、麦克风、手写板等，输出设备有显示器、投影仪、打印机等。`不过，在 Linux 中，标准输入设备指的是键盘，标准输出设备指的是显示器。`

==Linux 中一切皆文件，包括标准输入设备（键盘）和标准输出设备（显示器）在内的所有计算机硬件都是文件。==

为了表示和区分已经打开的文件，Linux 会给每个文件分配一个 ID，这个 ID 就是一个整数，被称为`文件描述符（File Descriptor）`。

| 文件描述符 | 文件名 | 类型             | 硬件   |
| ---------- | ------ | ---------------- | ------ |
| 0          | stdin  | 标准输入文件     | 键盘   |
| 1          | stdout | 标准输出文件     | 显示器 |
| 2          | stderr | 标准错误输出文件 | 显示器 |

> Linux 程序在执行任何形式的 I/O 操作时，都是在读取或者写入一个文件描述符。一个文件描述符只是一个和打开的文件相关联的整数，它的背后可能是一个硬盘上的普通文件、FIFO、管道、终端、键盘、显示器，甚至是一个网络连接。

`stdin`、`stdout`、`stderr` 默认都是打开的，在重定向的过程中，`0`、`1`、`2 `这三个文件描述符可以直接使用。

### 1.2Linux Shell 输出重定向

`输出重定向是指命令的结果不再输出到显示器上，而是输出到其它地方，一般是文件中。`这样做的最大好处就是把命令的结果保存起来，当我们需要的时候可以随时查询。Bash 支持的输出重定向符号如下表所示。

![image-20230102195257213](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102195257213.png)

在输出重定向中，`>`代表的是覆盖，`>>`代表的是追加。

**注意**

输出重定向的完整写法其实是`fd>file`或者`fd>>file`，其中 fd 表示文件描述符，如果不写，默认为 1，也就是标准输出文件。

当文件描述符为 1 时，一般都省略不写，如上表所示；当然，如果你愿意，也可以将`command >file`写作`command 1>file`，但这样做是多此一举。

当文件描述符为大于 1 的值时，比如 2，就必须写上。

需要重点说明的是，`fd`和`>`之间不能有空格，否则 Shell 会解析失败；`>`和`file`之间的空格可有可无。为了保持一致，我习惯在`>`两边都不加空格。

下面的语句是一个反面教材：

```shell
echo "c.biancheng.net" 1 >log.txt
```

注意`1`和`>`之间的空格。echo 命令的输出结果是`c.biancheng.net`，我们的初衷是将输出结果重定向到 log.txt，但是当你打开 log.txt 文件后，发现文件的内容为`c.biancheng.net 1`，这就是多余的空格导致的解析错误。也就是说，Shell 将该条语句理解成了下面的形式：

```shell
echo "c.biancheng.net" 1 1>log.txt
```

**输出重定向举例**

【实例1】将 echo 命令的输出结果以追加的方式写入到 demo.txt 文件中。

```shell
#!/bin/bash
for str in "C语言中文网" "http://c.biancheng.net/" "成立7年了" "日IP数万"
do
    echo $str >>demo.txt  #将输入结果以追加的方式重定向到文件
done
```

运行以上脚本，使用`cat demo.txt`查看文件内容，显示如下：

```shell
C语言中文网
http://c.biancheng.net/
成立7年了
日IP数万
```

【实例2】将`ls -l`命令的输出结果重定向到文件中。

```shell
[c.biancheng.net]$ ls -l  #先预览一下输出结果
总用量 16
drwxr-xr-x. 2 root     root      21 7月   1 2016 abc
-rw-r--r--. 1 mozhiyan mozhiyan 399 3月  11 17:12 demo.sh
-rw-rw-r--. 1 mozhiyan mozhiyan  67 3月  22 17:16 demo.txt
-rw-rw-r--. 1 mozhiyan mozhiyan 278 3月  16 17:17 main.c
-rwxr-xr-x. 1 mozhiyan mozhiyan 187 3月  22 17:16 test.sh
[c.biancheng.net]$ ls -l >demo.txt  #重定向
[c.biancheng.net]$ cat demo.txt  #查看文件内容
总用量 12
drwxr-xr-x. 2 root     root      21 7月   1 2016 abc
-rw-r--r--. 1 mozhiyan mozhiyan 399 3月  11 17:12 demo.sh
-rw-rw-r--. 1 mozhiyan mozhiyan   0 3月  22 17:21 demo.txt
-rw-rw-r--. 1 mozhiyan mozhiyan 278 3月  16 17:17 main.c
-rwxr-xr-x. 1 mozhiyan mozhiyan 187 3月  22 17:16 test.sh
```

**错误输出重定向举例**

命令正确执行是没有错误信息的，我们必须刻意地让命令执行出错，如下所示：

```shell
[c.biancheng.net]$ ls java  #先预览一下错误信息
ls: 无法访问java: 没有那个文件或目录
[c.biancheng.net]$ ls java 2>err.log  #重定向
[c.biancheng.net]$ cat err.log  #查看文件
ls: 无法访问java: 没有那个文件或目录
```

**正确输出和错误信息同时保存**

【实例1】把正确结果和错误信息都保存到一个文件中，例如：

```shell
[c.biancheng.net]$ ls -l >out.log 2>&1
[c.biancheng.net]$ ls java >>out.log 2>&1
[c.biancheng.net]$ cat out.log
总用量 12
drwxr-xr-x. 2 root     root      21 7月   1 2016 abc
-rw-r--r--. 1 mozhiyan mozhiyan 399 3月  11 17:12 demo.sh
-rw-rw-r--. 1 mozhiyan mozhiyan 278 3月  16 17:17 main.c
-rw-rw-r--. 1 mozhiyan mozhiyan   0 3月  22 17:39 out.log
-rwxr-xr-x. 1 mozhiyan mozhiyan 187 3月  22 17:16 test.sh
ls: 无法访问java: 没有那个文件或目录
```

out.log 的最后一行是错误信息，其它行都是正确的输出结果。

【实例2】上面的实例将正确结果和错误信息都写入同一个文件中，这样会导致视觉上的混乱，不利于以后的检索，所以我建议把正确结果和错误信息分开保存到不同的文件中，也即写成下面的形式：

```shell
ls -l >>out.log 2>>err.log
```

这样一来，正确的输出结果会写入到` out.log`，而错误的信息则会写入到 `err.log`。

**/dev/null 文件**

如果你既不想把命令的输出结果保存到文件，也不想把命令的输出结果显示到屏幕上，干扰命令的执行，那么可以把命令的所有结果重定向到 /dev/null 文件中。如下所示：

```shell
ls -l &>/dev/null
```

`大家可以把 /dev/null 当成 Linux 系统的垃圾箱，任何放入垃圾箱的数据都会被丢弃，不能恢复。`

### 1.3Linux Shell 输入重定向

输入重定向就是改变输入的方向，不再使用键盘作为命令输入的来源，而是使用文件作为命令的输入。

![image-20230102200232796](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102200232796.png)

和输出重定向类似，输入重定向的完整写法是`fd<file`，其中 fd 表示文件描述符，如果不写，默认为 0，也就是标准输入文件。

**输入重定向举例**

【示例1】统计文档中有多少行文字。

Linux wc 命令可以用来对文本进行统计，包括单词个数、行数、字节数，它的用法如下：

```shell
wc  [选项]  [文件名]
```

其中，`-c`选项统计字节数，`-w`选项统计单词数，`-l`选项统计行数。

统计 readme.txt 文件中有多少行文本：

```shell
[c.biancheng.net]$ cat readme.txt  #预览一下文件内容
C语言中文网
http://c.biancheng.net/
成立7年了
日IP数万
[c.biancheng.net]$ wc -l <readme.txt  #输入重定向
4
```

【实例2】逐行读取文件内容。

```shell
#!/bin/bash

while read str; do
    echo $str
done <readme.txt

运行结果：
C语言中文网
http://c.biancheng.net/
成立7年了
日IP数万
```

这种写法叫做代码块重定向，也就是把一组命令同时重定向到一个文件。

【实例3】统计用户在终端输入的文本的行数。

此处我们使用输入重定向符号`<<`，这个符号的作用是使用特定的分界符作为命令输入的结束标志，而不使用 Ctrl+D 键。

```shell
[c.biancheng.net]$ wc -l <<END
> 123
> 789
> abc
> xyz
> END
4
```

wc 命令会一直等待用输入，直到遇见分界符 END 才结束读取。

`<<`之后的分界符可以自由定义，只要再碰到相同的分界符，两个分界符之间的内容将作为命令的输入（不包括分界符本身）。

## 2.Linux 中的文件描述符到底是什么

`Linux 中一切皆文件，比如 C++ 源文件、视频文件、Shell 脚本、可执行文件等，就连键盘、显示器、鼠标等硬件设备也都是文件。`

一个 Linux 进程可以打开成百上千个文件，为了表示和区分已经打开的文件，Linux 会给每个文件分配一个编号（一个 ID），这个编号就是一个整数，被称为`文件描述符（File Descriptor）`。

这只是一个形象的比喻，为了让读者容易理解我才这么说。如果你也仅仅理解到这个层面，那不过是浅尝辄止而已，并没有看到文件描述符的本质。

本篇文章的目的就是拨云见雾，从底层实现的角度来给大家剖析一下文件描述符，看看文件描述如到底是如何表示一个文件的。

不过，阅读本篇文章需要你有 C 语言编程基础，至少要理解数组、指针和结构体；如果理解内存，那就更好了，看了这篇文章你会醍醐灌顶。

### 2.1Linux 文件描述符到底是什么？

一个 Linux 进程启动后，会在内核空间中创建一个 PCB 控制块，PCB 内部有一个文件描述符表（File descriptor table），记录着当前进程所有可用的文件描述符，也即当前进程所有打开的文件。

>内核空间是虚拟地址空间的一部分，想死磕的读者请猛击《C 语言内存精讲》，不想纠缠细节的读者可以这样理解：进程启动后要占用内存，其中一部分内存分配给了文件描述符表。

除了文件描述符表，系统还需要维护另外两张表：

- 打开文件表（Open file table）
- i-node 表（i-node table）

文件描述符表每个进程都有一个，打开文件表和 i-node 表整个系统只有一个，它们三者之间的关系如下图所示。

![image-20230102200945535](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102200945535.png)

从本质上讲，这三种表都是结构体数组，0、1、2、73、1976 等都是数组下标。表头只是我自己添加的注释，数组本身是没有的。实线箭头表示指针的指向，虚线箭头是我自己添加的注释。

==你看，文件描述符只不过是一个数组下标吗！==

通过文件描述符，可以找到文件指针，从而进入打开文件表。该表存储了以下信息：

- 文件偏移量，也就是文件内部指针偏移量。调用 read() 或者 write() 函数时，文件偏移量会自动更新，当然也可以使用 lseek() 直接修改。

- 状态标志，比如只读模式、读写模式、追加模式、覆盖模式等。
- i-node 表指针。

然而，要想真正读写文件，还得通过打开文件表的 i-node 指针进入 i-node 表，该表包含了诸如以下的信息：

- 文件类型，例如常规文件、套接字或 FIFO。
- 文件大小。
- 时间戳，比如创建时间、更新时间。
- 文件锁。

对上图的进一步说明：

- 在进程 A 中，文件描述符 1 和 20 都指向了同一个打开文件表项，标号为 23（指向了打开文件表中下标为 23 的数组元素），这可能是通过调用 dup()、dup2()、fcntl() 或者对同一个文件多次调用了 open() 函数形成的。

- 进程 A 的文件描述符 2 和进程 B 的文件描述符 2 都指向了同一个文件，这可能是在调用 fork() 后出现的（即进程 A、B 是父子进程关系），或者是不同的进程独自去调用 open() 函数打开了同一个文件，此时进程内部的描述符正好分配到与其他进程打开该文件的描述符一样。

- 进程 A 的描述符 0 和进程 B 的描述符 3 分别指向不同的打开文件表项，但这些表项均指向 i-node 表的同一个条目（标号为 1976）；换言之，它们指向了同一个文件。发生这种情况是因为每个进程各自对同一个文件发起了 open() 调用。同一个进程两次打开同一个文件，也会发生类似情况。

有了以上对文件描述符的认知，我们很容易理解以下情形：

- 同一个进程的不同文件描述符可以指向同一个文件；
- 不同进程可以拥有相同的文件描述符；
- 不同进程的同一个文件描述符可以指向不同的文件（一般也是这样，除了 0、1、2 这三个特殊的文件）；
- 不同进程的不同文件描述符也可以指向同一个文件。

## 3.结合文件描述符谈重定向，彻底理解重定向的本质！

Linux 系统这个“傻帽”只有一根筋，每次读写文件的时候，都从文件描述符下手，通过文件描述符找到文件指针，然后进入`打开文件表`和 `i-node 表`，这两个表里面才真正保存了与打开文件相关的各种信息。

`试想一下，如果我们改变了文件指针的指向，不就改变了文件描述符对应的真实文件吗？`比如文件描述符 1 本来对应显示器，但是我们偷偷将文件指针指向了 log.txt 文件，那么文件描述符 1 也就和 log.txt 对应起来了。

文件指针只不过是一个内存地址，修改它是轻而易举的事情。文件指针是文件描述符和真实文件之间最关键的“纽带”，然而这条纽带却非常脆弱，很容易被修改。

Linux 系统提供的函数可以修改文件指针，比如 dup()、dup2()；Shell 也能修改文件指针，输入输出重定向就是这么干的。

==对，没错，输入输出重定向就是通过修改文件指针实现的！==更准确地说，发生重定向时，Linux 会用文件描述符表（一个结构体数组）中的一个元素给另一个元素赋值，或者用一个结构体变量给数组元素赋值，整体上的资源开销相当低。

你看，发生重定向的时候，文件描述符并没有改变，改变的是文件描述符对应的文件指针。对于标准输出，Linux 系统始终向文件描述符 1 中输出内容，而不管它的文件指针指向哪里；只要我们修改了文件指针，就能向任意文件中输出内容。

以下面的语句为例来说明：

```shell
echo "c.biancheng.net" 1>log.txt
```

> 文件描述符表本质上是一个结构体数组，假设这个结构体的名字叫做 FD。发生重定向时，Linux 系统首先会打开 log.txt 文件，并把各种信息添加到 i-node 表和文件打开表，然后再创建一个 FD 变量（通过这个变量其实就能读写文件了），并用这个变量给下标为 1 的数组元素赋值，覆盖原来的内容，这样就改变了文件指针的指向，完成了重定向。

### 3.1Shell 对文件描述符的操作

前面提到，>是输出重定向符号，<是输入重定向符号；更准确地说，它们应该叫做`文件描述符操作符`。> 和 < 通过修改文件描述符改变了文件指针的指向，所以能够实现重定向的功能。

除了` >` 和 `<`，Shell 还是支持`<>`，它的效果是前面两者的总和。

![image-20230102204725602](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102204725602.png)

![image-20230102204735952](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102204735952.png)

【实例 1】前面的文章中提到了下面这种用法：

```shell
command >file 2>&1
```

它省略了文件描述符 1，所以等价于：

```shell
command 1>file 2>&1
```

这个语句可以分成两步：先执行 1>file，让文件描述符 1 指向 file；再执行 2>&1，用文件描述符 1 修改文件描述符 2，让 2 和 1 的内容一样。最终 1 和 2 都指向了同一个文件，也就是 file。所以不管是向 1 还是向 2 中输出内容，最终都输出到 file 文件中。

这里需要注意执行顺序，多个操作符在一起会从左往右依次执行。对于上面的语句，就是先执行 1>file，再执行 2>&1；如果写作下面的形式，那就南辕北辙了：

```shell
command 2>&1 1>file
```

Shell 会先执行 2>&1，这样 1 和 2 都指向了标准输出文件，也即显示器；接着执行 1>file，这样 1 就指向了 file 文件，但是 2 依然指向显示器。最终的结果是，正确的输出结果输出到了 file 文件，错误信息却还是输出到显示器。

【实例 2】一个比较奇葩的重定向写法。

```shell
echo "C 语言中文网" 10>log.txt >&10
```

先执行 `10>log.txt`，打开 log.txt，并给它分配文件描述符 10；接着执行`>&10`，用文件描述符 10 来修改文件描述符 1（对于`>`，省略不写的话默认为 1），让 1 和 10 都指向 log.txt 文件，最终的结果是向 log.txt 文件中输出内容。这条语句其实等价于 `echo "C 语言中文网" >log.txt`，我之所以写得这么绕，是为了让大家理解各种操作符的用法。

文件描述符 10 只用了一次，我们在末尾最好将它关闭，这是一个好习惯。

```shell
echo "C 语言中文网" 10>log.txt >&10 10>&-
```

## 4.使用exec命令操作文件描述符

exec 是 Shell 内置命令，它有两种用法，一种是`执行 Shell 命令`，一种是`操作文件描述符`。本节只讲解后面一种，前面一种请大家自行学习。

`使用 exec 命令可以永久性地重定向`，后续命令的输入输出方向也被确定了，直到再次遇到 exec 命令才会改变重定向的方向；换句话说，一次重定向，永久有效。

嗯？什么意思？难道说我们以前使用的重定向都是临时的吗？是的！前面使用的重定向都是临时的，它们只对当前的命令有效，对后面的命令无效。

请看下面的例子：

```shell
mozhiyan@localhost ~]$ echo "c.biancheng.net" > log.txt
[mozhiyan@localhost ~]$ echo "C 语言中文网"
C 语言中文网
[mozhiyan@localhost ~]$ cat log.txt
c.biancheng.net
```

第一个 echo 命令使用了重定向，将内容输出到 log.txt 文件；第二个 echo 命令没有再次使用重定向，内容就直接输出到显示器上了。很明显，重定向只对第一个 echo 有效，对第二个 echo 无效。

有些脚本文件的输出内容很多，我们不希望直接输出到显示器上，或者我们需要把输出内容备份到文件中，方便以后检索，按照以前的思路，必须在每个命令后面都使用一次重定向，写起来非常麻烦。如果以后想修改重定向的方向，那工作量也是不小的。

exec 命令就是为解决这种困境而生的，它可以让重定向对当前 Shell 进程中的所有命令有效，它的用法为：

```shell
exec 文件描述符操作
```

```shell
[mozhiyan@localhost ~]$ echo "重定向未发生"
重定向未发生
[mozhiyan@localhost ~]$ exec >log.txt
[mozhiyan@localhost ~]$ echo "c.biancheng.net"
[mozhiyan@localhost ~]$ echo "C 语言中文网"
[mozhiyan@localhost ~]$ exec >&2
[mozhiyan@localhost ~]$ echo "重定向已恢复"
重定向已恢复
[mozhiyan@localhost ~]$ cat log.txt
c.biancheng.net
C 语言中文网
```

对代码的说明：

- exec >log.txt 将当前 Shell 进程的所有标准输出重定向到 log.txt 文件，它等价于 exec 1>log.txt。
- 后面的两个 echo 命令都没有在显示器上输出，而是输出到了 log.txt 文件。
- exec >&2 用来恢复重定向，让标准输出重新回到显示器，它等价于 exec 1>&2。2 是标准错误输出的文件描述符，它也是输出到显示器，并且没有遭到破坏，我们用 2 来覆盖 1，就能修复 1，让 1 重新指向显示器。
- 接下来的 echo 命令将结果输出到显示器上，证明 exec >&2 奏效了。
- 最后我们用 cat 命令来查看 log.txt 文件的内容，发现就是中间两个 echo 命令的输出。

### 4.1重定向的恢复

类似 `echo "1234" >log.txt` 这样的重定向只是临时的，当前命名执行完毕后会自动恢复到显示器，我们不用担心。但是诸如`exec >log.txt` 这种使用 exec 命令的重定向都是持久的，如果我们想再次回到显示器，就必须手动恢复。

以输出重定向为例，手动恢复的方法有两种：

- /dev/tty 文件代表的就是显示器，将标准输出重定向到 /dev/tty 即可，也就是 exec >/dev/tty。
- 如果还有别的文件描述符指向了显示器，那么也可以别的文件描述符来恢复标号为 1 的文件描述符，例如 exec >&2。注意，如果文件描述符 2 也被重定向了，那么这种方式就无效了。

下面的例子演示了输入重定向的恢复：

```shell
#!/bin/bash
exec 6<&0 #先将 0 号文件描述符保存
exec <nums.txt #输入重定向
sum=0
while read n; do
	((sum += n))
done
echo "sum=$sum"
exec 0<&6 6<&- #恢复输入重定向，并关闭文件描述符 6
read -p "请输入名字、网址和年龄：" name url age
echo "$name 已经$age 岁了，它的网址是 $url"
```

将代码保存到 test.sh，并执行下面的命令：

```shell
[mozhiyan@localhost ~]$ cat nums.txt
80
33
129
71
100
222
8
[mozhiyan@localhost ~]$ bash ./test.sh
sum=643
请输入名字、网址和年龄：C 语言中文网 http://c.biancheng.net 7
C 语言中文网已经 7 岁了，它的网址是 http://c.biancheng.net
```

## 5.Shell 代码块重定向

所谓代码块，就是由多条语句组成的一个整体；for、while、until 循环，或者 if...else、case...in 选择结构，或者由{ }包围的命令都可以称为代码块。

将重定向命令放在代码块的结尾处，就可以对代码块中的所有命令实施重定向。

【实例 1】使用 while 循环不断读取 nums.txt 中的数字，计算它们的总和。

```shell
#!/bin/bash
sum=0
while read n; do
	((sum += n))
done <nums.txt #输入重定向
echo "sum=$sum"
```

将代码保存到 test.sh 并运行：

```shell
[c.biancheng.net]$ cat nums.txt
80
33
129
71
100
222
8
[c.biancheng.net]$ . ./test.sh
sum=643
```

对上面的代码进行改进，记录 while 的读取过程，并将输出结果重定向到 log.txt 文件：

```shell
#!/bin/bash
sum=0
while read n; do
	((sum += n))
	echo "this number: $n"
done <nums.txt >log.txt #同时使用输入输出重定向
echo "sum=$sum"
```

将代码保存到 test.sh 并运行：

```shell
[c.biancheng.net]$ . ./test.sh
sum=643
[c.biancheng.net]$ cat log.txt
this number: 80
this number: 33
this number: 129
this number: 71
this number: 100
this number: 222
this number: 8
```

【实例 2】对{}包围的代码使用重定向。

```shell
#!/bin/bash
{
	echo "C 语言中文网";
	echo "http://c.biancheng.net";
	echo "7"
} >log.txt #输出重定向
{
	read name;
	read url;
	read age
} <log.txt #输入重定向
echo "$name 已经$age 岁了，它的网址是 $url"
```

将代码保存到 test.sh 并运行：

```shell
[c.biancheng.net]$ . ./test.sh
C 语言中文网已经 7 岁了，它的网址是 http://c.biancheng.net
[c.biancheng.net]$ cat log.txt
C 语言中文网
http://c.biancheng.net
7
```

## 6.Shell组命令

所谓组命令，就是将多个命令划分为一组，或者看成一个整体。

Shell 组命令的写法有两种：

```shell
{ command1; command2; command3; . . . }
(command1; command2; command3;. . . )
```

两种写法的区别在于：`由花括号{}包围起来的组命名在当前 Shell 进程中执行，而由小括号()包围起来的组命令会创建一个子 Shell，所有命令都在子 Shell 中执行。`

对于第一种写法，花括号和命令之间必须有一个空格，并且最后一个命令必须用一个分号或者一个换行符结束。

子 Shell 就是一个子进程，是通过当前 Shell 进程创建的一个新进程。但是子 Shell 和一般的子进程（比如 bash ./test.sh 创建的子进程）还是有差别的，我们将在《子 Shell 和子进程》一节中深入讲解，读者暂时把子 Shell 和子进程等价起来就行。

`组命令可以将多条命令的输出结果合并在一起，在使用重定向和管道时会特别方便。`

例如，下面的代码将多个命令的输出重定向到 out.txt：

```shell
ls -l > out.txt #>表示覆盖
echo "http://c.biancheng.net/shell/" >> out.txt #>>表示追加
cat readme.txt >> out.txt
```

本段代码共使用了三次重定向。

借助组命令，我们可以将以上三条命令合并在一起，简化成一次重定向：

```shell
{ ls -l; echo "http://c.biancheng.net/shell/"; cat readme.txt; } > out.txt
```

或者写作：

```shell
(ls -l; echo "http://c.biancheng.net/shell/"; cat readme.txt) > out.txt
```

使用组命令技术，我们节省了一些打字时间。

类似的道理，我们也可以将组命令和管道结合起来：

```shell
{ ls -l; echo "http://c.biancheng.net/shell/"; cat readme.txt; } | lpr
```

这里我们把三个命令的输出结果合并在一起，并把它们用管道输送给命令 lpr 的输入，以便产生一个打印报告。

### 6.1两种组命令形式的对比

虽然两种 Shell 组命令形式看起来相似，它们都能用在重定向中合并输出结果，但两者之间有一个很重要的不同：`由{}包围的组命令在当前 Shell 进程中执行，由()包围的组命令会创建一个子 Shell，所有命令都会在这个子 Shell 中执行。`

在子 Shell 中执行意味着，运行环境被复制给了一个新的 shell 进程，当这个子 Shell 退出时，新的进程也会被销毁，环境副本也会消失，所以在子 Shell 环境中的任何更改都会消失（包括给变量赋值）。因此，在大多数情况下，除非脚本要求一个子 Shell，否则使用{}比使用()更受欢迎，并且{}的进行速度更快，占用的内存更少。

## 7.Shell 进程替换

进程替换和命令替换非常相似。命令替换是把一个命令的输出结果赋值给另一个变量，例如 dir_files=`ls -l`或 date_time=`$(date)`；而进程替换则是把一个命令的输出结果传递给另一个（组）命令。

为了说明进程替换的必要性，我们先来看一个使用管道的例子：

```shell
echo "http://c.biancheng.net/shell/" | read
echo $REPLY
```

以上代码输出结果总是为空，因为 echo 命令在父 Shell 中执行，而 read 命令在子 Shell 中执行，当 read 执行结束时，子 Shell 被销毁，REPLY 变量也就消失了。`管道中的命令总是在子 Shell 中执行的，任何给变量赋值的命令都会遭遇到这个问题。`

> 使用 read 读取数据时，如果没有提供变量名，那么读取到的数据将存放到环境变量 REPLY 中，这一点已在《Shell read》中讲到。幸运的是，Shell 提供了一种“特异功能”，叫做进程替换，它可以用来解决这种麻烦。

Shell 进程替换有两种写法，一种用来产生标准输出，借助输入重定向，它的输出结果可以作为另一个命令的输入：

```shell
<(commands)
```

另一种用来接受标准输入，借助输出重定向，它可以接收另一个命令的输出结果：

```shell
>(commands)
```

commands 是一组命令列表，多个命令之间以分号;分隔。注意，<或>与圆括号之间是没有空格的。

例如，为了解决上面遇到的问题，我们可以像下面这样使用进程替换：

```shell
read < <(echo "http://c.biancheng.net/shell/")
echo $REPLY
输出结果：
http://c.biancheng.net/shell/
```

整体上来看，Shell 把 `echo "http://c.biancheng.net/shell/"`的输出结果作为 read 的输入。`<()`用来捕获 echo 命令的输出结果，`<`用来将该结果重定向到 read。

注意，两个`<`之间是有空格的，第一个`<`表示输入重定向，第二个`<和()`连在一起表示进程替换 。

本例中的`read `命令和`第二个 echo `命令都在当前 Shell 进程中运行，`读取的数据也会保存到当前进程的 REPLY 变量，大家都在一个进程中，所以使用 echo 能够成功输出。`

> read和echo都是Shell的内置命令，都在当前Shell中执行。

而在前面的例子中我们使用了管道，`echo 命令在父进程中运行`，`read 命令在子进程中运行，读取的数据也保存在子进程的 REPLY 变量中`，echo 命令和 REPLY 变量不在一个进程中，而`子进程的环境变量对父进程是不可见的`，所以读取失败。

再来看一个进程替换用作「接受标准输入」的例子：

```shell
echo "C 语言中文网" > >(read; echo "你好，$REPLY")
运行结果：
你好，C 语言中文网
```

因为使用了重定向，`read` 命令从 `echo "C 语言中文网"`的输出结果中读取数据。

### 7.1Shell进程替换的本质

为了能够在`不同进程之间传递数据`，实际上进程替换会跟系统中的文件关联起来，这个文件的名字为`/dev/fd/n`（n 是一个整数）。`该文件会作为参数传递给()中的命令，()中的命令对该文件是读取还是写入取决于进程替换格式是<还是>`：

- 如果是>()，那么该文件会给()中的命令提供输入；借助输出重定向，要输入的内容可以从其它命令而来。
- 如果是<()，那么该文件会接收()中命令的输出结果；借助输入重定向，可以将该文件的内容作为其它命令的输入。

使用 echo 命令可以查看进程替换对应的文件名：

```shell
[c.biancheng.net]$ echo >(true)
/dev/fd/63
[c.biancheng.net]$ echo <(true)
/dev/fd/63
[c.biancheng.net]$ echo >(true) <(true)
/dev/fd/63 /dev/fd/62
```

`/dev/fd/`目录下有很多序号文件，进程替换一般用的是 `63` 号文件，该文件是系统内部文件，我们一般查看不到。

我们通过下面的语句进行实例分析：

```shell
echo "shellscript" > >(read; echo "hello, $REPLY")
```

第一个`>`表示`输出重定向`，它把第一个 echo 命令的输出结果重定向到`/dev/fd/63` 文件中。

\>()`中的第一个命令是 read，它需要从标准输入中读取数据，此时就`用/dev/fd/63 作为输入文件`，把该文件的内容交给 read 命令，接着使用 echo 命令输出 read 读取到的内容。

==可以看到，/dev/fd/63 文件起到了数据中转或者数据桥梁的作用，借助重定向，它将>()内部的命令和外部的命令联系起来，使得数据能够在这些命令之间流通。==

## 8.Linux Shell管道

通过前面的学习，我们已经知道了怎样从文件重定向输入，以及重定向输出到文件。Shell 还有一种功能，就是可以将两个或者多个命令（程序或者进程）连接到一起，把一个命令的输出作为下一个命令的输入，以这种方式连接的两个或者多个命令就形成了`管道（pipe）`。

Linux 管道使用竖线`|`连接多个命令，这被称为管道符。Linux 管道的具体语法格式如下：

```shell
command1 | command2
command1 | command2 [ | commandN... ]
```

当在两个命令之间设置管道时，管道符`|`左边命令的输出就变成了右边命令的输入。只要第一个命令向标准输出写入，而第二个命令是从标准输入读取，那么这两个命令就可以形成一个管道。大部分的 Linux 命令都可以用来形成管道。

> 这里需要注意，command1 必须有正确输出，而 command2 必须可以处理 command2 的输出结果；而且 command2 只能处理 command1 的正确输出结果，不能处理 command1 的错误信息。

### 8.1为什么使用管道

我们先看下面一组命令，使用 mysqldump（一个数据库备份程序）来备份一个叫做 wiki 的数据库：

```shell
mysqldump -u root -p '123456' wiki > /tmp/wikidb.backup
gzip -9 /tmp/wikidb.backup
scp /tmp/wikidb.backup username@remote_ip:/backup/mysql/
```

上述这组命令主要做了如下任务：

- mysqldump 命令用于将名为 wike 的数据库备份到文件 /tmp/wikidb.backup；其中`-u`和`-p`选项分别指出数据库的用户名和密码。
- gzip 命令用于压缩较大的数据库文件以节省磁盘空间；其中`-9`表示最慢的压缩速度最好的压缩效果。
- scp 命令（secure copy，安全拷贝）用于将数据库备份文件复制到 IP 地址为 remote_ip 的备份服务器的 /backup/mysql/ 目录下。其中`username`是登录远程服务器的用户名，命令执行后需要输入密码。

上述三个命令依次执行。`然而，如果使用管道的话，你就可以将 mysqldump、gzip、ssh 命令相连接，这样就避免了创建临时文件 /tmp/wikidb.backup，而且可以同时执行这些命令并达到相同的效果。`

使用管道后的命令如下所示：

```shell
mysqldump -u root -p '123456' wiki | gzip -9 | ssh username@remote_ip "cat > /backup/wikidb.gz"
```

这些使用了管道的命令有如下特点：

- 命令的语法紧凑并且使用简单。
- 通过使用管道，将三个命令串联到一起就完成了远程 mysql 备份的复杂任务。
- 从管道输出的标准错误会混合到一起。

上述命令的数据流如下图所示：

![image-20230103101948871](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230103101948871.png)

### 8.2重定向和管道的区别

乍看起来，管道也有重定向的作用，它也改变了数据输入输出的方向，那么，管道和重定向之间到底有什么不同呢？

`简单地说，重定向操作符>将命令与文件连接起来，用文件来接收命令的输出；而管道符|将命令与命令连接起来，用第二个命令来接收第一个命令的输出。`如下所示：

```shell
command > file
command1 | command1
```

有些读者在学习管道时会尝试如下的命令，我们来看一下会发生什么：

```shell
command1 > command2
```

答案是，有时尝试的结果将会很糟糕。这是一个实际的例子，一个 Linux 系统管理员以超级用户（root 用户）的身份执行了如下命令：

```shell
cd /usr/bin
ls > less
```

> 第一条命令将当前目录切换到了大多数程序所存放的目录，第二条命令是告诉 Shell 用 ls 命令的输出重写文件 less。因为 /usr/bin 目录已经包含了名称为 less（less 程序）的文件，第二条命令用 ls 输出的文本重写了 less 程序，因此破坏了文件系统中的 less 程序。

这是使用重定向操作符错误重写文件的一个教训，所以在使用它时要谨慎。

### 8.3Linux管道实例

【实例1】将 ls 命令的输出发送到 grep 命令：

```shell
[c.biancheng.net]$ ls | grep log.txt
log.txt
```

上述命令是查看文件 log.txt 是否存在于当前目录下。

我们可以在命令的后面使用选项，例如使用`-al`选项：

```shell
[c.biancheng.net]$ ls -al | grep log.txt
-rw-rw-r--.  1 mozhiyan mozhiyan    0 4月  15 17:26 log.txt
```

管道符`|`与两侧的命令之间也可以不存在空格，例如将上述命令写作`ls -al|grep log.txt`；然而我还是推荐在管道符`|`和两侧的命令之间使用空格，以增加代码的可读性。

我们也可以重定向管道的输出到一个文件，比如将上述管道命令的输出结果发送到文件 output.txt 中：

```shell
[c.biancheng.net]$ ls -al | grep log.txt >output.txt
[c.biancheng.net]$ cat output.txt
-rw-rw-r--.  1 mozhiyan mozhiyan    0 4月  15 17:26 log.txt
```

【实例2】使用管道将 cat 命令的输出作为 less 命令的输入，这样就可以将 cat 命令的输出每次按照一个屏幕的长度显示，这对于查看长度大于一个屏幕的文件内容很有帮助。

```shell
cat /var/log/message | less
```

【实例3】查看指定程序的进程运行状态，并将输出重定向到文件中。

```shell
[c.biancheng.net]$ ps aux | grep httpd > /tmp/ps.output
[c.biancheng.net]$ cat /tem/ps.output
mozhiyan  4101     13776  0   10:11 pts/3  00:00:00 grep httpd
root      4578     1      0   Dec09 ?      00:00:00 /usr/sbin/httpd
apache    19984    4578   0   Dec29 ?      00:00:00 /usr/sbin/httpd
apache    19985    4578   0   Dec29 ?      00:00:00 /usr/sbin/httpd
apache    19986    4578   0   Dec29 ?      00:00:00 /usr/sbin/httpd
apache    19987    4578   0   Dec29 ?      00:00:00 /usr/sbin/httpd
apache    19988    4578   0   Dec29 ?      00:00:00 /usr/sbin/httpd
apache    19989    4578   0   Dec29 ?      00:00:00 /usr/sbin/httpd
apache    19990    4578   0   Dec29 ?      00:00:00 /usr/sbin/httpd
apache    19991    4578   0   Dec29 ?      00:00:00 /usr/sbin/httpd
```

【实例4】显示按用户名排序后的当前登录系统的用户的信息。

```shell
[c.biancheng.net]$ who | sort
mozhiyan :0           2019-04-16 12:55 (:0)
mozhiyan pts/0        2019-04-16 13:16 (:0)
```

who 命令的输出将作为 sort 命令的输入，所以这两个命令通过管道连接后会显示按照用户名排序的已登录用户的信息。

【实例5】统计系统中当前登录的用户数。

```shell
[c.biancheng.net]$ who | wc -l
5
```

### 8.4管道与输入重定向

输入重定向操作符<可以在管道中使用，以用来从文件中获取输入，其语法类似下面这样：

```shell
command1 < input.txt | command2
command1 < input.txt | command2 -option | command3
```

例如，使用 tr 命令从 os.txt 文件中获取输入，然后通过管道将输出发送给 sort 或 uniq 等命令：

```shell
[c.biancheng.net]$ cat os.txt
redhat
suse
centos
ubuntu
solaris
hp-ux
fedora
centos
redhat
hp-ux
[c.biancheng.net]$ tr a-z A-Z <os.txt | sort
CENTOS
CENTOS
FEDORA
HP-UX
HP-UX
REDHAT
REDHAT
SOLARIS
SUSE
UBUNTU
[c.biancheng.net]$ tr a-z A-Z <os.txt | sort | uniq
CENTOS
FEDORA
HP-UX
REDHAT
SOLARIS
SUSE
UBUNTU
```

### 8.5管道与输出重定向

你也可以使用重定向操作符>或>>将管道中的最后一个命令的标准输出进行重定向，其语法如下所示：

```shell
command1 | command2 | ... | commandN > output.txt
command1 < input.txt | command2 | ... | commandN > output.txt
```

【实例1】使用 mount 命令显示当前挂载的文件系统的信息，并使用 column 命令格式化列的输出，最后将输出结果保存到一个文件中。

```shell
[c.biancheng.net]$ mount | column -t >mounted.txt
[c.biancheng.net]$ cat mounted.txt
proc         on  /proc                  type  proc        (rw,nosuid,nodev,noexec,relatime)
sysfs        on  /sys                   type  sysfs       (rw,nosuid,nodev,noexec,relatime,seclabel)
devtmpfs     on  /dev                   type  devtmpfs    (rw,nosuid,seclabel,size=496136k,nr_inodes=124034,mode=755)
securityfs   on  /sys/kernel/security   type  securityfs  (rw,nosuid,nodev,noexec,relatime)
tmpfs        on  /dev/shm               type  tmpfs       (rw,nosuid,nodev,seclabel)
devpts       on  /dev/pts               type  devpts      (rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620,ptmxmode=000)
tmpfs        on  /run                   type  tmpfs       (rw,nosuid,nodev,seclabel,mode=755)
tmpfs        on  /sys/fs/cgroup         type  tmpfs       (rw,nosuid,nodev,noexec,seclabel,mode=755)
#####此处省略部分内容#####
```

【实例2】使用 tr 命令将 os.txt 文件中的内容转化为大写，并使用 sort 命令将内容排序，使用 uniq 命令去除重复的行，最后将输出重定向到文件 ox.txt.new。

```shell
[c.biancheng.net]$ cat os.txt
redhat
suse
centos
ubuntu
solaris
hp-ux
fedora
centos
redhat
hp-ux
[c.biancheng.net]$ tr a-z A-Z <os.txt | sort | uniq >os.txt.new
[c.biancheng.net]$ cat os.txt.new
CENTOS
FEDORA
HP-UX
REDHAT
SOLARIS
SUSE
UBUNTU
```

## 9.Shell过滤器

我们己经知道，将几个命令通过管道符组合在一起就形成一个管道。`通常，通过这种方式使用的命令就被称为过滤器。过滤器会获取输入，通过某种方式修改其内容，然后将其输出。`

简单地说，过滤器可以概括为以下两点：

- 如果一个 Linux 命令是从标准输入接收它的输入数据，并在标准输出上产生它的输出数据（结果），那么这个命令就被称为过滤器。
- 过滤器通常与 Linux 管道一起使用。

常用的被作为过滤器使用的命令如下所示：

| 命令    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| awk     | 用于文本处理的解释性程序设计语言，通常被作为数据提取和报告的工具。 |
| cut     | 用于将每个输入文件（如果没有指定文件则为标准输入）的每行的指定部分输出到标准输出。 |
| grep    | 用于搜索一个或多个文件中匹配指定模式的行。                   |
| tar     | 用于归档文件的应用程序。                                     |
| head    | 用于读取文件的开头部分（默认是 10 行）。如果没有指定文件，则从标准输入读取。 |
| paste   | 用于合并文件的行。                                           |
| sed     | 用于过滤和转换文本的流编辑器。                               |
| sort    | 用于对文本文件的行进行排序。                                 |
| split   | 用于将文件分割成块。                                         |
| strings | 用于打印文件中可打印的字符串。                               |
| tac     | 与 cat 命令的功能相反，用于倒序地显示文件或连接文件。        |
| tail    | 用于显示文件的结尾部分。                                     |
| tee     | 用于从标准输入读取内容并写入到标准输出和文件。               |
| tr      | 用于转换或删除字符。                                         |
| uniq    | 用于报告或忽略重复的行。                                     |
| wc      | 用于打印文件中的总行数、单词数或字节数。                     |

接下来，我们通过几个实例来演示一下过滤器的使用。

### 9.1在管道中使用 awk 命令

关于 awk 命令的具体用法，请大家自行学习，本节我们我们仅通过几个简单的实例来了解一下 awk 命令在管道中的使用。

【实例1】查看系统中的所有的账号名称，并按名称的字母顺序排序。

```shell
[c.biancheng.net]$ awk -F: '{print $1}' /etc/passwd | sort
adm
apache
avahi
avahi-autoipd
bin
daemon
dbus
ftp
games
...
```

在上例中，使用冒号`:`作为列分隔符，将文件 `/etc/passwd` 的内容分为了多列，并打印了第一列的信息（即用户名），然后将输出通过管道发送到了 sort 命令。

【实例2】列出当前账号最常使用的 10 个命令。

```shell
[c.biancheng.net]$ history | awk '{print $2}' | sort | uniq -c | sort -rn | head
140 echo
 75 man
 71 cat
 63 su
 53 ls
 50 vi
 47 cd
 40 date
 26 let
 25 paste
```

在上例中，history 命令将输出通过管道发送到 awk 命令，awk 命令默认使用空格作为列分隔符，将 history 的输出分为了两列，并把第二列内容作为输出通过管道发送到了 sort 命令，使用 sort 命令进行排序后，再将输出通过管道发送到了 uniq 命令，使用 uniq 命令 统计了历史命令重复出现的次数，再用 sort 命令将 uniq 命令的输出按照重复次数从高到低排序，最后使用 head 命令默认列出前 10 个的信息。

【实例3】显示当前系统的总内存大小，单位为 KB。

```shell
[c.biancheng.net]$ free | grep Mem | awk '{print $2}'
2029860
```

### 9.2在管道中使用 cut 命令

cut 命令被用于文本处理。你可以使用这个命令来提取文件中指定列的内容。

【实例1】查看系统中登录 Shell 是“/bin/bash”的用户名和对应的用户主目录的信息：

```shell
[c.biancheng.net]$ grep "bin/bash" /etc/passwd | cut -d: -f1,6
root:/root
mozhiyan:/home/mozhiyan
```

如果你对 Linux 系统有所了解，你会知道，/etc/passwd 文件被用来存放用户账号的信息，此文件中的每一行会记录一个账号的信息，每个字段之间用冒号分隔，第一个字段即是账号的账户名，而第六个字段就是账号的主目录的路径。

【实例2】查看当前机器的CPU类型。

```shell
[c.biancheng.net]$ cat /proc/cpuinfo | grep name | cut -d: -f2 | uniq
Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
```

上例中，执行命令`cat /proc/cpuinfo | grep name`得到的内容如下所示：

```shell
[c.biancheng.net]$ cat /proc/cpuinfo | grep name
model name    : Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
model name    : Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
model name    : Intel(R) Core(TM) i5-2520M CPU @ 2.50GHz
model name    : Intel(R) Core(TM) i5-2520M CPU 0 2.50GHz
```

然后，我们使用 cut 命令将上述输出内容以冒号作为分隔符，将内容分为了两列， 并显示第二列的内容，最后使用 uniq 命令去掉了重复的行。

【实例3】查看当前目录下的子目录数。

```shell
[c.biancheng.net]$ ls -l | cut -c 1 | grep d | wc -l
5
```

上述管道命令主要做了如下操作：

- 命令`ls -l`输出的内容中，每行的第一个字符表示文件的类型，如果第一个字符是`d`，就表示文件的类型是目录。
- 命令`cut -c 1`是截取每行的第一个字符。
- 命令`grep d`来获取文件类型是目录的行。
- 命令`wc -l`用来获得 grep 命令输出结果的行数，即目录个数。

### 9.3在管道中使用grep命令

grep 命令是在管道中比较常用的一个命令。

【实例1】查看系统日志文件中的错误信息。

```
[c.biancheng.net]$ grep -i "error:" /var/log/messages | less
```

【实例2】查看系统中 HTTP 服务的进程信息。

```shell
[c.biancheng.net]$ ps aux | grep httpd
apache 18968 0.0 0.0 26472 10404 ?    S    Dec15    0:01 /usr/sbin/httpd
apache 18969 0.0 0.0 25528  8308 ?    S    Dec15    0:01 /usr/sbin/httpd
apache 18970 0.0 0.0 26596 10524 ?    S    Dec15    0:01 /usr/sbin/httpd
```

【实例3】查找我们的程序列表中所有命令名中包含关键字 zip 的命令。

```shell
[c.biancheng.net]$ ls /bin /usr/bin | sort | uniq | grep zip
bunzip2
bzip2
bzip2recover
gunzip
gzip
```

【实例4】查看系统安装的 kernel 版本及相关的 kernel 软件包。

```shell
[c.biancheng.net]$ rpm -qa | grep kernel
kernel-2.6.18-92.e15
kernel-debuginfo-2.6.18-92.e15
kernel-debuginfo-common-2.6.18-92.e15
kernel-devel-2.6.18-92.e15
```

【实例5】查找 /etc 目录下所有包含 IP 地址的文件。

```shell
[c.biancheng.net]$ find /etc -type f -exec grep '[0-9][0-9]*[.][0-9][0-9]*[.][0-9][0-9]*[.][0-9][0-9]*' {} \;
```

### 9.4在管道中使用 tar 命令

tar 命令是 Linux 系统中最常用的打包文件的程序。

【实例1】你可以使用 tar 命令复制一个目录的整体结构。

```shell
[c.biancheng.net]$ tar cf - /home/mozhiyan | ( cd /backup/; tar xf - )
```

【实例2】跨网络地复制一个目录的整体结构。

```shell
[c.biancheng.net]$ tar cf - /home/mozhiyan | ssh remote_host "( cd /backup/; tar xf - )"
```

【实例3】跨网络地压缩复制一个目录的整体结构。

```shell
tar czf - /home/mozhiyan | ssh remote_host "( cd /backup/; tar xzf - )"
```

【实例4】检査 tar 归档文件的大小，单位为字节。

```shell
[c.biancheng.net]$ cd /; tar cf - etc | wc -c
215040
```

【实例5】检查 tar 归档文件压缩为 tar.gz 归档文件后所占的大小。

```shell
[c.biancheng.net]$ tar czf - etc.tar | wc -c
58006
```

### 9.5在管道中使用 head 命令

有时，你不需要一个命令的全部输出，可能只需要命令的前几行输出。这时，就可以使用 head 命令，它只打印命令的前几行输出。默认的输出行数为 10 行。

【实例1】显示 ls 命令的前 10 行输出。

```shell
[c.biancheng.net]$ ls /usr/bin | head
addftinfo
afmtodit
apropos
arch
ash
awk
base64
basename
bash
bashbug
```

【实例2】显示 ls 命令的前 5 行内容。

```shell
[c.biancheng.net]$ ls / | head -n 5
bin
cygdrive
Cygwin.bat
Cygwin.ico
Cygwin-Terminal.ico
```

### 9.6在管道中使用 uniq 命令

uniq 命令用于报告或删除重复的行。我们将使用一个测试文件进行管道中使用 uniq 命令的实例讲解，其内容如下所示：

```shell
[c.biancheng.net]$ cat testfile
This line occurs only once.
This line occurs twice.
This line occurs twice.
This line occurs three times.
This line occurs three times.
This line occurs three times.
```

【实例1】去掉输出中重复的行。

```shell
[c.biancheng.net]$ sort testfile | uniq
This line occurs only once.
This line occurs three times.
This line occurs twice.
```

【实例2】显示输出中各重复的行出现的次数，并按次数多少倒序显示。

```shell
[c.biancheng.net]$ sort testfile | uniq -c | sort -nr
3 This line occurs three times.
2 This line occurs twice.
1 This line occurs only once.
```

## 9.7在管道中使用 wc 命令

wc 命令用于统计包含在文本流中的字符数、单词数和行数。

【实例1】统计当前登录到系统的用户数。

```shell
[c.biancheng.net]$ who | wc -l
```

【实例2】统计当前的 Linux 系统中的进程数。

```shell
[c.biancheng.net]$ ps -ef | wc -l
```

## 10.子 Shell 和子进程到底有什么区别

Shell 中有很多方法产生子进程，比如以新进程的方式运行 Shell 脚本，使用组命令、管道、命令替换等，但是这些子进程是有区别的。

子进程的概念是由父进程的概念引申而来的。`在 Linux 系统中，系统运行的应用程序几乎都是从 init（pid 为 1 的进程）进程派生而来的，所有这些应用程序都可以视为 init 进程的子进程，而 init 则为它们的父进程。`

使用` pstree -p `命令就可以看到` init 及系统中其他进程的进程树信息（包括 pid）`：

```shell
systemd(1)─┬─ModemManager(796)─┬─{ModemManager}(821)
 │ └─{ModemManager}(882)
 ├─NetworkManager(975)─┬─{NetworkManager}(1061)
 │ └─{NetworkManager}(1077)
 ├─abrt-watch-log(774)
 ├─abrt-watch-log(776)
 ├─abrtd(773)
 ├─accounts-daemon(806)─┬─{accounts-daemon}(839)
 │ └─{accounts-daemon}(883)
 ├─alsactl(768)
 ├─at-spi-bus-laun(1954)─┬─dbus-daemon(1958)───{dbus-daemon}(1960)
 │ ├─{at-spi-bus-laun}(1955)
 │ ├─{at-spi-bus-laun}(1957)
 │ └─{at-spi-bus-laun}(1959)
 ├─at-spi2-registr(1962)───{at-spi2-registr}(1965)
 ├─atd(842)
 ├─auditd(739)─┬─audispd(753)─┬─sedispatch(757)
 │ │ └─{audispd}(759)
 │ └─{auditd}(752)
```

本教程基于 CentOS 7 编写，CentOS 7 为了提高启动速度使用 systemd 替代了 init。CentOS 7 之前的版本依然使用 init。

`Shell 脚本是从上至下、从左至右依次执行的，即执行完一个命令之后再执行下一个。`如果在 Shell 脚本中遇到子脚本（即脚本嵌套，但是必须以新进程的方式运行）或者外部命令，就会`向系统内核申请创建一个新的进程`，以便在该进程中执行子脚本或者外部命令，这个新的进程就是子进程。`子进程执行完毕后才能回到父进程，才能继续执行父脚本中后续的命令及语句。`

![image-20230103110219398](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230103110219398.png)

### 10.1子进程的创建

了解 Linux 编程的读者应该知道，`使用 fork() 函数可以创建一个子进程；除了 PID（进程 ID）等极少的参数不同外，子进程的一切都来自父进程，包括代码、数据、堆栈、打开的文件等，就连代码的执行位置（状态）都是一样的`。

也就是说，fork() 克隆了一个一模一样的自己，身高、体重、颜值、嗓音、年龄等各种属性都相同。当然，后期随着各自的发展轨迹不同，两者会变得不一样，比如 A 好吃懒做越来越肥，B 经常健身成了一个肌肉男；但是在 fork() 出来的那一刻，两者都是一样的。

Linux 还有一种创建子进程的方式，就是`子进程被 fork() 出来以后立即调用 exec() 函数加载新的可执行文件，而不使用从父进程继承来的一切`。什么意思呢？

比如在 ~/bin 目录下有两个可执行文件分别叫 a.out 和 b.out。现在我运行 a.out，就会产生一个进程，比如叫做 A。在进程 A 中我又调用 fork() 函数创建了一个进程 B，那么 B 就是 A 的子进程，此时它们是一模一样的。但是，我调用 fork() 后立即又调用 exec() 去加载 b.out，这可就坏事了，B 进程中的一切（包括代码、数据、堆栈等）都会被销毁，然后再根据 b.out 重建建立一切。这样一折腾，B 进程除了 ID 没有变，其它的都变了，再也没有属于 A 的东西了。

你看，同样是创建子进程，但是结果却大相径庭：

- 第一种只使用 fork() 函数，子进程和父进程几乎是一模一样的，父进程中的函数、变量、别名等在子进程中仍然有效。
- 第二种使用 fork() 和 exec() 函数，子进程和父进程之间除了硬生生地维持一种“父子关系”外，再也没有任何联系了，它们就是两个完全不同的程序。

对于 Shell 来说，以新进程的方式运行脚本文件，比如 `bash ./test.sh`、`chmod +x ./test.sh; ./test.sh`，或者在当前 Shell 中使用bash 命令`启动新的 Shell`，它们都属于`第二种`创建子进程的方式，所以`子进程除了能继承父进程的环境变量外，基本上也不能使用父进程的什么东西了`，比如，父进程的全局变量、局部变量、文件描述符、别名等在子进程中都无效。

但是，`组命令`、`命令替换`、`管道`这几种语法都使用`第一种`方式创建进程，所以子进程可以使用父进程的一切，包括全局变量、局部变量、别名等。我们将这种子进程称为`子 Shell（sub shell）`。

==子 Shell 虽然能使用父 Shell 的的一切，但是如果子 Shell 对数据做了修改，比如修改了全局变量，那么这种修改只能停留在子 Shell，无法传递给父 Shell。==不管是子进程还是子 Shell，都是“传子不传父”。

### 10.2总结

子 Shell 才是真正继承了父进程的一切，这才像“一个模子刻出来的”；普通子进程和父进程是完全不同的两个程序，只是维持着父子关系而已

## 11.如何检测子Shell和子进程

我们都知道使用 `$$` 变量可以获取当前进程的 ID，我在父 Shell 和子 Shell 中都输出 `$$` 的值，只要它们不一样，不就是创建了一个新的进程吗？那我们就来试一下吧。

```shell
[mozhiyan@localhost ~]$ echo $$ #父 Shell PID
3299
[mozhiyan@localhost ~]$ (echo $$) #组命令形式的子 Shell PID
3299
[mozhiyan@localhost ~]$ echo "http://c.biancheng.net" | { echo $$; } #管道形式的子 Shell PID
3299
[mozhiyan@localhost ~]$ read < <(echo $$) #进程替换形式的子 Shell PID
[mozhiyan@localhost ~]$ echo $REPLY
3299
```

你看，子 Shell 和父 Shell 的 ID 都是一样的，哪有产生新进程了？作者你是不是骗人呢？

> 其实不是我骗人，而是你掉坑里了，因为 `$$` 变量在子 Shell 中无效！Base 官方文档说，在普通的子进程中，`$$` 确实被展开为子进程的ID；但是在子 Shell 中，`$$` 却被展开成父进程的 ID。

除了 `$$`，Bash 还提供了另外两个环境变量——`SHLVL`和`BASH_SUBSHELL`，用它们来检测子 Shell 非常方便。

`SHLVL` 是记录多个 Bash 进程实例嵌套深度的累加器，每次进入一层`普通的子进程`，SHLVL 的值就加1。而 `BASH_SUBSHELL` 是记录一个 Bash 进程实例中多个子Shell（sub shell）嵌套深度的累加器，每次进入一层`子Shell`，`BASH_SUBSHELL`的值就加 1。

1)我们还是用实例来说话吧，先说 SHLVL。创建一个脚本文件，命名为 test.sh，内容如下：

```shell
#!/bin/bash
echo "$SHLVL $BASH_SUBSHELL"
```

然后打开 Shell 窗口，依次执行下面的命令：

```shell
[mozhiyan@localhost ~]$ echo "$SHLVL $BASH_SUBSHELL"
2 0
[mozhiyan@localhost ~]$ bash #执行 bash 命令开启一个新的 Shell 会话
[mozhiyan@localhost ~]$ echo "$SHLVL $BASH_SUBSHELL"
3 0
[mozhiyan@localhost ~]$ bash ./test.sh #通过 bash 命令运行脚本
4 0
mozhiyan@localhost ~]$ echo "$SHLVL $BASH_SUBSHELL"
3 0
[mozhiyan@localhost ~]$ chmod +x ./test.sh #给脚本增加执行权限
[mozhiyan@localhost ~]$ ./test.sh
4 0
[mozhiyan@localhost ~]$ echo "$SHLVL $BASH_SUBSHELL"
3 0
[mozhiyan@localhost ~]$ exit #退出内层 Shell
exit
[mozhiyan@localhost ~]$ echo "$SHLVL $BASH_SUBSHELL"
2 0
```

SHLVL 和 BASH_SUBSHELL 的初始值都是 0，但是输出结果中 SHLVL 的值从 2 开始，我猜测 Bash 在初始化阶段可能创建了子进程，我们暂时不用理会它，将关注点放在值的变化上。

仔细观察的读者应该会发现，使用 bash 命令开启新的会话后，需要使用 exit 命令退出才能回到上一级 Shell 会话。

`bash ./test.sh` 和 `chmod +x ./test.sh; ./test.sh` 这两种运行脚本的方式，在脚本运行期间会`开启一个子进程，运行结束后立即退出子进程`。

2)再说一下 BASH_SUBSHELL，请看下面的命令：

```shell
[mozhiyan@localhost ~]$ echo "$SHLVL $BASH_SUBSHELL"
2 0
[mozhiyan@localhost ~]$ (echo "$SHLVL $BASH_SUBSHELL") #组命令
2 1
[mozhiyan@localhost ~]$ echo "hello" | { echo "$SHLVL $BASH_SUBSHELL"; } #管道
2 1
[mozhiyan@localhost ~]$ var=$(echo "$SHLVL $BASH_SUBSHELL") #命令替换
[mozhiyan@localhost ~]$ echo $var
2 1
[mozhiyan@localhost ~]$ ( ( ( (echo "$SHLVL $BASH_SUBSHELL") ) ) ) #四层组命令
2 4
```

你看，组命令、管道、命令替换这几种写法都会进入子 Shell。

注意，“进程替换”看起来好像产生了一个子 Shell，其实只是玩了一个障眼法而已。进程替换只是借助文件在`()`内部和外部的命令之间传递数据，但是它并没有创建子 Shell；`换句话说，()内部和外部的命令是在一个进程（也就是当前进程）中执行的。`

我们不妨来实际检测一下：

```shell
[mozhiyan@localhost ~]$ echo "$SHLVL $BASH_SUBSHELL"
2 0
[mozhiyan@localhost ~]$ echo "hello" > >(echo "$SHLVL $BASH_SUBSHELL")
2 0
```

SHLVL 和 BASH_SUBSHELL 变量的值都没有发生改变，说明进程替换既没有进入子进程，也没有进入子 Shell。

## 12.Linux 中的信号是什么

在 Linux 中，理解信号的概念是非常重要的。这是因为，`信号被用于通过 Linux 命令行所做的一些常见活动中`。例如，每当你按`Ctrl+C` 组合键来从命令行终结一个命令的执行，你就使用了信号。每当你使用如下命令来结束一个进程时，你就使用了信号：

```shell
kill -9 [PID]
```

所以，至少知道信号的基本原理是非常有用的。

### 12.1Linux 中的信号

在 Linux 系统（以及其他类 Unix 操作系统）中，`信号被用于进程间的通信`。信号是一个发送到某个进程或同一进程中的特定线程的`异步通知`，用于通知发生的一个事件。从 1970 年贝尔实验室的 Unix 面世便有了信号的概念，而现在它已经被定义在了 POSIX 标准中。

对于在 Linux 环境进行编程的用户或系统管理员来说，较好地理解信号的概念和机制是很重要的，在某些情况下可以帮助我们更高效地编写程序。对于一个程序来说，如果每条指令都运行正常的话，它会连续地执行。`但如果在程序执行时，出现了一个错误或任何异常，内核就可以使用信号来通知相应的进程。`

信号同样被用于通信、同步进程和简化进程间通信，在 Linux 中，信号在处理异常和中断方面，扮演了极其重要的角色。信号巳经在没有任何较大修改的情况下被使用了将近 30 年。

当一个事件发生时，会产生一个信号，然后内核会将事件传递到接收的进程。有时，进程可以发送一个信号到其他进程。除了进程到进程的信号外，还有很多种情况，内核会产生一个信号，比如文件大小达到限额、一个 I/O 设备就绪或用户发送了一个类似于 Ctrl+C 或Ctrl+Z 的终端中断等。

运行在用户模式下的进程会接收信号。如果接收的进程正运行在内核模式，那么信号的执行只有在该进程返回到用户模式时才会开始。

> 发送到非运行进程的信号一定是由内核保存，直到进程重新执行为止。休眠的进程可以是可中断的，也可以是不可中断的。如果一个在可中断休眠状态的进程（例如，等待终端输入的进程）收到了一个信号，那么内核会唤醒这个进程来处理信号。如果一个在不可中断休眠状态的进程收到了一个信号，那么内核会拖延此信号，直到该事件完成为止。

当进程收到一个信号时，可能会发生以下 3 种情况：

- 进程可能会忽略此信号。有些信号不能被忽略，而有些没有默认行为的信号，默认会被忽略。
- 进程可能会捕获此信号，并执行一个被称为信号处理器的特殊函数。
- 进程可能会执行信号的默认行为。例如，信号 15(SIGTERM) 的默认行为是结束进程。

`当一个进程执行信号处理时，如果还有其他信号到达，那么新的信号会被阻断直到处理器返冋为止。`

### 12.2信号的名称和值

每个信号都有以` SIG `开头的名称，并定义为唯一的正整数。在 Shell 命令行提示符下，输入 `kill -l `命令，将显示所有信号的信号值和相应的信号名，类似如下所示：

```shell
[c.biancheng.net]$ kill -l
1) SIGHUP 2) SIGINT 3) SIGQUIT 4) SIGILL 5) SIGTRAP
6) SIGABRT 7) SIGBUS 8) SIGFPE 9) SIGKILL 10) SIGUSR1
11) SIGSEGV 12) SIGUSR2 13) SIGPIPE 14) SIGALRM 15) SIGTERM
16) SIGSTKFLT 17) SIGCHLD 18) SIGCONT 19) SIGSTOP 20) SIGTSTP
21) SIGTTIN 22) SIGTTOU 23) SIGURG 24) SIGXCPU 25) SIGXFSZ
26) SIGVTALRM 27) SIGPROF 28) SIGWINCH 29) SIGIO 30) SIGPWR
31) SIGSYS 34) SIGRTMIN 35) SIGRTMIN+1 36) SIGRTMIN+2 37) SIGRTMIN+3
38) SIGRTMIN+4 39) SIGRTMIN+5 40) SIGRTMIN+6 41) SIGRTMIN+7 42) SIGRTMIN+8
43) SIGRTMIN+9 44) SIGRTMIN+10 45) SIGRTMIN+11 46) SIGRTMIN+12 47) SIGRTMIN+13
48) SIGRTMIN+14 49) SIGRTMIN+15 50) SIGRTMAX-14 51) SIGRTMAX-13 52) SIGRTMAX-12
53) SIGRTMAX-11 54) SIGRTMAX-10 55) SIGRTMAX-9 56) SIGRTMAX-8 57) SIGRTMAX-7
58) SIGRTMAX-6 59) SIGRTMAX-5 60) SIGRTMAX-4 61) SIGRTMAX-3 62) SIGRTMAX-2
63) SIGRTMAX-1 64) SIGRTMAX
```

信号值被定义在文件` /usr/include/bits/signum.h `中，其源文件是 `/usr/src/linux/kernel/signal.c`。

在 Linux 下，可以查看 signal(7) 手册页来查阅信号名列表、信号值、默认的行为和它们是否可以被捕获。其命令如下所示：

```shell
man 7 signal
```

下标所列出的信号是 POSIX 标准的一部分，它们通常被缩写成不带 `SIG` 前缀，例如，SIGHUP 通常被简单地称为 HUP。

![image-20230103113540266](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230103113540266.png)

![image-20230103113554153](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230103113554153.png)

## 13.Bash Shell中的信号简述

当没有任何捕获时，一个交互式 Bash Shell 会忽略 SIGTERM 和 SIGQUIT 信号。由 Bash 运行的非内部命令会使用 Shell 从其父进程继承的信号处理程序。如果没有启用作业控制，异步执行的命令会忽略除了有这些信号处理程序之外的 SIGINT 和 SIGQUIT 信号。由于命令替换而运行的命令会忽略键盘产生的作业控制信号 SIGTTIN、SIGTTOU 和 SIGTSTP。

默认情况下，Shell 接收到 SIGHUP 信号后会退出。在退出之前，一个交互式的 Shell 会向所有的作业，不管是正在运行的还是已停止的，重新发送 SIGHUP 信号。对已停止的作业，Shell 还会发送 SIGCONT 信号以确保它能够接收到 SIGHUP 信号。

若要阻止 Shell 向某个特定的作业发送 SIGHUP 信号，可以使用内部命令 disown 将它从作业表中移除，或是用“disown -h”命令阻止 Shell 向特定的作业发送 SIGHUP 信号，但并不会将特定的作业从作业表中移除。

我们通过如下实例，来了解一下 disown 命令的作用：

```shell
#将 sleep 命令放在后台执行，休眠 30 秒
[c.biancheng.net]$ sleep 30 &
[1] 8052
#列出当前 Shell 下所有作业的信息
[c.biancheng.net]$ jobs -l
[1]+ 8052 Running sleep 30 &
#将作业 1 从作业表中移除
[c.biancheng.net]$ disown %1
#再次列出当前 Shell 下所有作业的信息
[c.biancheng.net]$ jobs -l
#查找 sleep 进程
[c.biancheng.net]$ ps -ef | grep sleep
mozhiyan 8052 8092 cons1 11:28:21 /usr/bin/sleep
#打印当前 Shell 的进程号
[c.biancheng.net]$ echo $$
8092
```

在上述实例中，我们首先将命令“sleep 30”放在后台运行，此时，我们使用命令“jobs -l”可以看到作业表中有一个正在运行的作业，然后，我们使用命令“disown %1”将作业 1 从作业表中移除，再使用命令“jobs -l”会看到作业表中已经没有了作业，但是我们发现其实“sleep 30”这个命令的进程仍然存在。此时，Shell 若接收到 SIGHUP 信号，它就不会向作业 1 重新发送 SIGHUP 信号，此时如果我们退出 Shell，这个作业仍将继续运行，而不会被终止。

我们再来看一下命令“disown -h”的用途：

```shell
#将 sleep 命令放在后台执行，休眠 30 秒
[c.biancheng.net]$ sleep 30 &
[1] 3184
#列出当前 Shell 下所有作业的信息
[c.biancheng.net]$ jobs -l
[1]+ 3184 Running sleep 30 &
#阻止 Shell 向作业 1 发送 SIGHUP 信号
[c.biancheng.net]$ disown -h %1
[c.biancheng.net]$ jobs -l
[1]+ 3184 Running sleep 30 &
```

我们看到，在执行了命令“disown -h %1”后，作业 1 并没有从作业表中移除，但它己经被标记，所以即使 Shell 收到 SIGHUP 信号也不会向此作业发送 SIGHUP 信号。因此， 如果此时我们退出 Shell，这个作业也仍将继续运行，而不会被终止。

>注意：如果使用内部命令 shopt 打开了 Shell 的 huponexit 选项，当一个交互式的登录 Shell 退出时，会向所有的作业发送 SIGHUP 信号。

## 14.Linux 进程简明教程

进程是 Linux 操作系统中最重要的基本概念之一，这一节我们将了解学习 Linux 进程的一些基础知识。

进程是运行在 Linux 中的程序的一个实例。这是一个你之前就可能已经听说过的基本定义。当你在 Linux 系统中执行一个程序时，系统会为这个程序创建特定的环境。这个环境包含系统运行这个程序所需的任何东西。

每当你在 Linux 中执行一个命令，它都会创建，或启动一个新的进程。比如，当你尝试运行命令“ls -l”来列出目录的内容时，你就启动了一个进程。如果有两个终端窗口显示在屏幕上，那么你可能运行了两次同样的终端程序，这时会有两个终端进程。

每个终端窗门可能都运行了一个 Shell，每个运行的 Shell 都分别是一个进程。当你从 Shell 调用一个命令时，对应的程序就会在一个新进程中执行，当这个程序的进程执行完成后，Shell 的进程将恢复运行。

操作系统通过被称为 PID 或进程 ID 的数字编码来追踪进程。系统中的每一个进程都有一个唯一的 PID。

现在我们通过一个实例来了解 Linux 中的进程。我们在 Shell 命令行下执行如下命令：

```shell
[c.biancheng.net]$ sleep 10 &
[1] 3324
```

因为程序会等待 10 秒，所以我们快速地在当前 Shell 上查找任何进程名为 sleep 的进程：

```shell
[c.biancheng.net]$ ps -ef | grep sleep
mozhiyan 3324 5712 cons1 17:11:46 /usr/bin/sleep
```

我们看到进程名为 /usr/bin/sleep 的进程正运行在系统中（其 PID 与我们在上一命令中得到的 PID 相同）。

现在，我们尝试连续运行3条上述的 sleep 命令，上述命令的输出将类似如下所示：

```shell
[c.biancheng.net]$ ps -ef | grep sleep
mozhiyan 896 5712 cons1 17:16:51 /usr/bin/sleep
mozhiyan 5924 5712 cons1 17:16:52 /usr/bin/sleep
mozhiyan 2424 5712 cons1 17:16:50 /usr/bin/sleep
```

我们看到 sleep 程序的每一个实例都创建了一个单独的进程。

每个 Linux 进程还有另一个 ID 号码，即父进程的 ID(ppid)。系统中的每一个用户进程都有一个父进程。命令“ps -f”就会列出进程的 PID 和 PPID。此命令的输出类似如下所示：

```shell
[c.biancheng.net]$ ps -f
UID      PID  PPID TTY   STIME    COMMAND
mozhiyan 4124 228  cons0 21:37:09 /usr/bin/ps
mozhiyan 228  1    cons0 21:32:23 /usr/bin/bash
```

你在 Shell 命令行提示符下运行的命令都把当前 Shell 的进程作为父进程。例如，你在 Shell 命令行提示符下输入 ls 命令，Shell 将执行 ls 命令，此时 Linux 内核会复制 Shell 的内存页，然后执行 ls 命令。

在 Unix 中，每一个进程是使用 fork 和 exec 方法创建的。然而，这种方法会导致系统资源的损耗。在 Linux 中，fork 方法是使用`写时拷贝内存页`实现的，所以它导致的仅是时间和复制父进程的内存页表所需的内存的损失，并且会为子进程创建一个唯一的任务结构。

> 写时拷贝模式在创建新进程时避免了创建不必要的结构拷贝。例如，用户在 Shell 命令行提示符下输出 ls 命令，Linux 内核将会创建一个 Shell 的子进程，即 Shell 的进程是父进程，而 ls 命令的进程是子进程，ls 命令的进程会指向与此 Shell 相同的内存页，然后子进程使用写时拷贝技术执行 ls 命令。

### 14.1前台进程和后台进程

当你启动一个进程时（运行一个命令），可以如下两种方式运行该进程：

- 前台进程

- 后台进程

默认情况下，你启动的每一个进程都是运行在前台的。它从键盘获取输入并发送它的输出到屏幕。`当一个进程运行在前台时，我们不能在同一命令行提示符下运行任何其他命令（启动任何其他进程），因为在程序结束它的进程之前命令行提示符不可用。`

启动一个后台进程最简单的方法是添加一个控制操作符“&”到命令的结尾。例如，如下命令将启动一个后台进程：

```shell
[c.biancheng.net]$ sleep 10 &
[1] 5720
```

现在 sleep 命令被放在后台运行。当 Bash 在后台启动一个作业时，它会打印一行内容显示作业编号（[1]）和进程号（PID-5720）。当作业完成时，作业会发送类似如下的信息到终端程序，来显示此作业已完成，其内容类似如下所示：

```shell
[1]+ Done sleep 10
```

将进程放在后台运行的好处是：你可以继续运行其他命令，而不需要等待此进程运行完成再运行其他命令。

### 14.2进程的状态

每个 Linux 进程都有它自己的生命周期，比如，创建、执行、结束和清除。每个进程也都有各自的状态，显示进程中当前正发生什么。

进程可以有如下几种状态：

- D（不可中断休眠状态）——进程正在休眠并且不能恢复，直到一个事件发生为止。
- R（运行状态）——进程正在运行。
- S（休眠状态）——进程没有在运行，而在等待一个事件或是信号。
- T（停止状态）——进程被信号停止，比如，信号 SIGINT 或 SIGSTOP。
- Z（僵死状态）——标记为 `<defunct>` 的进程是僵死的进程，它们之所以残留是因为它们的父进程没有适当地销毁它们。如果父进程退出，这些进程将被 init 进程销毁。

若要查看指定进程的状态，可以使用如下命令：

```shell
ps -C processName -o pid=,cmd,stat
```

例如：

```shell
[c.biancheng.net]$ ps -C sleep -o pid=,cmd,stat
CMD STAT
9434 sleep 20 S
```

## 15.Linux使用什么命令查看进程

通过前面章节的一些实例的学习，想必你已经知道了使用 ps 命令可以查看进程的信息，但除了` ps `命令，我们还可以使用` pstree `命令和` pgrep `命令查看当前进程的信息。

使用 ps 命令，可以查看当前的进程。`默认情况下，ps 命令只会输出当前用户并且是当前终端（比如，当前 Shell）下调用的进程的信息。`其输出将类似如下所示：

```shell
[c.biancheng.net]$ ps
PID  TTY   TIME     CMD
4380 pts/0 00:00:00 bash
4414 pts/0 00:00:00 ps
```

我们从上面的输出中可以看到，默认情况下，ps 命令会显示进程 ID(PID)、与进程关联的终端（TTY）、格式为“[dd-]hh:mm:ss”的进程累积 CPU 时间（TIME），以及可执行文件的名称（CMD）。并且，输出内容默认是不排序的。

使用标准语法显示系统中的每个进程：

```shell
[c.biancheng.net]$ ps -ef | head -2
UID  PID PPID C STIME TTY TIME     CMD
root 1   0    0 Janl4 ?   00:00:02 init [5]
```

使用 BSD 语法显示系统中的每个进程：

```shell
[c.biancheng.net]$ ps aux | head -2
USER PID %CPU %MEM VSZ  RSS TTY STAT START TIME COMMAND
root 1   0.0  0.0  2160 648 ?   Ss   Janl4 0:02 init [5]
```

使用 BSD 样式选项会增加进程状态（STAT）等信息作为默认显示，你也可以使用 `PS_FORMAT` 环境变量重写默认的输出格式。

查看系统中 httpd 进程的信息：

```shell
ps aux | grep httpd
```

使用 pstree 命令，可以显示进程树的信息：

```shell
[c.biancheng.net]$ pstree
init-+-acpid
 |-atd
 |-auditd-+-audispd---{audispd}
 | `-{auditd}
 |-automount---4*[{automount}]
 |-avahi-daemon---avahi-daemon
 |-crond---5*[crond-+-mj.sh]
 | `-sendmail]
 |-cupsd
  |-dbus-daemon---{dbus-daemon}
 |-events/0
 |-events/1
 |-gam_server
 |-gpm
 |-hald---hald-runner-+-hald-addon-acpi
 | |-hald-addon-keyb
 | `-hald-addon-stor
 |-hcid
 |-hidd
 |-hpiod
 |-java-+-java---17*[{java}]
 | `-14*[{java}]
 |-java-+-java---29*[{java}]
 | `-14*[{java}]
 |-java-+-java---34*[{java}]
 | `-14*[{java}]
 |-java---20*[{java}]
 |-java---292*[{java}]
 |-khelper
 |-klogd
 |-krfcommd
 |-ksoftirqd/0
 |-ksoftirqd/1
 |-kthread-+-aio/0
 | |-aio/1
 | |-ata/0
 | |-ata/1
 | |-ata_aux
 | |-cqueue/0
 | |-cqueue/1
  | |-hd-audio0
 | |-kacpid
 | |-kauditd
 | |-kblockd/0
 | |-kblockd/1
 | |-khubd
 | |-khungtaskd
 | |-2*[kjournald]
 | |-kmpath_handlerd
 | |-kmpathd/0
 | |-kmpathd/1
 | |-kondemand/0
 | |-kondemand/1
 | |-kpsmoused
 | |-kseriod
 | |-ksnapd
 | |-kstriped
 | |-kswapd0
 | |-2*[pdflush]
 | |-rpciod/0
 | |-rpciod/1
 | |-scsi_eh_0
 | |-scsi_eh_1
 | |-scsi_eh_2
 | |-scsi_eh_3
 | |-scsi_eh_4
 | `-scsi_eh_5
 |-loop0
 |-mcstransd
 |-migration/0
 |-migration/1
 |-6*[mingetty]
 |-mj.sh---make---java---11*[{java}]
 |-ntpd
 |-pcscd---{pcscd}
 |-portmap
 |-python
 |-restorecond
 |-rpc.idmapd
 |-rpc.statd
 |-screen---bash---update.sh---cvs
 |-sendmail---2*[sendmail]
 |-sendmail
 |-setroubleshootd---2*[{setroubleshootd}]
 |-smartd
 |-sshd-+-sshd---bash---update_and_rest---cvs
 | |-sshd---bash---pstree
 | `-sshd---bash
 |-start_derby.sh---java---45*[{java}]
 |-surf---8*[{surf}]
 |-syslogd
 |-tomcat---sleep
 |-udevd
 |-watchdog/0
 |-watchdog/1
 |-xfs
 |-xinetd
 `-yum-updatesd
```

pstree 命令以树形结构的形式显示系统中所有当前运行的进程的信息。此树形结构以指定的 PID 为根，若没有指定 PID，则以 init 进程为根。下面，我们看一个显示指定 PID 的进程树的例子：

```shell
[c.biancheng.net]$ pstree 4578
httpd-11*[httpd]
```

上述输出内容的含义是，PID 是 4578 的 httpd 进程下有 11 个 httpd 子进程。在显示时，pstree 命令会将一样的分支合并到一个方括号中，并在方括号前显示重复的次数。

如果 pstree 命令指定的参数是用户名，那么就会显示以此用户的进程为根的所有进程树的信息。其显示内容将类似如下所示：

```shell
[c.biancheng.net]$ pstree mozhiyan
Xvnc
dbus-daemon
dbus-launch
dcopserver
gconfd-2
kded
kdeinit-+-bt-applet
 |-esc-+-esc---9*[{esc}]
 | `-esc---6*[{esc}]
 |-2*[kio_file]
 |-kio—media
 |-klauncher
 `-kwin
kdesktop
kicker
klipper
ksmserver
bash---pstree
start_kdeinit
xstartup---startkde---kwrapper
```

使用 pgrep 命令，可以基于名称或其他属性查找进程。

pgrep 命令会检查当前运行的进程，并列出与选择标准相匹配的进程的 ID。例如，查看 root 用户的 sshd 进程的 PID：

```shell
[c.biancheng.net]$ pgrep -u root sshd
2877
6572
18563
```

列出所有者是 root 和 daemon 的进程的 PID：

```shell
pgrep -u root,daemon
```

## 16.Shell 向进程发送信号

我们可以使用键盘或 pkill 命令、kill 命令和 killall 命令向进程发送各种信号。

### 16.1使用键盘发送信号使用键盘发送信号

在 Bash Shell 下，我们可以使用键盘发送信号，如下表所示。

![image-20230103120659847](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230103120659847.png)

### 16.2kill 命令发送信号

大多数主流的 Shell，包括 Bash，都有内置的 kill 命令。Linux 系统中，也有 kill 命令，即 /bin/kill。如果使用 /bin/kill，则系统可能会激活一些额外的选项，比如，杀掉不是你自己的进程，或指定进程名作为参数，类似于 pgrep 和 pkill 命令。不过两种 kill 命令默认都是发送 SIGTERM 信号。

当准备杀掉一个进程或一连串的进程时，我们的常识是从尝试发送最安全的信号开始，即 SIGTERM 信号。以这种方式，关心正常停止运行的程序，当它收到 SIGTERM 信号时，有机会按照已经设计好的流程执行，比如，清理和关闭打开的文件。

如果你发送一个 SIGKILL 信号到进程，你将消除进程先清理而后关闭的机会，而这可能会导致不幸的结果。但如果一个有序地终结不管用，那么发送 SIGINT 或 SIGKILL 信号就可能是唯一的方法了。例如，当一个前台进程使用 Ctrl+C 组合键杀不掉时，那最好就使用命令“kill -9 PID” 了。

在前面的学习中我们已经了解，kill 命令可以发送多种信号到进程。特别有用的信号包括：

- SIGHUP (1)
- SIGINT (2)
- SIGKILL (9)
- SIGCONT (18)
- SIGSTOP (19)

在 Bash Shell 中，信号名或信号值都可作为 kill 命令的选项，而作业号或进程号则作为 kill 命令的参数。

【实例1】发送 SIGKILL 信号到 PID 是 123 的进程。

```shell
kill -9 123
```

或是：

```shell
kill -KILL 123
```

也可以是：

```shell
kill -SIGKILL 123
```

【实例2】使用 kill 命令终结一个作业。

```shell
#将 sleep 命令放在后台执行，休眠 30 秒
[c.biancheng.net]$ sleep 30 &
[1] 20551
#列出当前 Shell 下所有作业的信息
[c.biancheng.net]$ jobs -l
[1]+ 20551 Running sleep 30 &
#终结作业 1
[c.biancheng.net]$ kill %1
[1]+ 20551 Terminated sleep 30
#查看当前 Shell 下的作业的信息
[c.biancheng.net]$ jobs -l
```

### 16.3killall 命令发送信号

killall 命令会发送信号到运行任何指定命令的所有进程。所以，当一个进程启动了多个实例时，使用 killall 命令来杀掉这些进程会更方便些。

>注意：在生产环境中，若没有经验，使用 killall 命令之前请先测试该命令，因为在一些商业 Unix 系统中，它可能不像所期望的那样工作。

如果没有指定信号名，killall 命令会默认发送 SIGTERM 信号。例如，使用 killall 命令杀掉所有 firefox 进程：

```shell
killall firefox
```

发送 KILL 信号到 firefox 的进程：

```shell
killall -s SIGKILL firefox
```

### 16.4pkill 命令发送信号

使用 pkill 命令，可以通过指定进程名、用户名、组名、终端、UID、EUID 和 GID 等属性来杀掉相应的进程。pkill 命令默认也是发送SIGTERM 信号到进程。

【实例1】使用 pkill 命令杀掉所有用户的 firefox 进程。

```shell
pkill firefox
```

【实例2】强制杀掉用户 mozhiyan 的 firefox 进程。

```shell
pkill -KILL -u mozhiyan firefox
```

【实例3】让 sshd 守护进程重新加载它的配置文件。

```shell
pkill -HUP sshd
```
