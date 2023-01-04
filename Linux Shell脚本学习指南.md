# Linux Shell脚本学习指南

## 1.Shell基础

欢迎来到 Linux Shell 的世界，在真正开始 Linux Shell 编程之前，先让我们简单地了解一下 Shell 的基本知识，以便为我们接下来的学习打下一个好的基础。学习完本章，你能对 Shell 有一个初步的了解，比如 Shell 是什么，有哪些？Shell 命令是什么？

### 1.1Shell的概念

对于图形界面，用户点击某个图标就能启动某个程序；对于命令行，用户输入某个程序的名字（可以看做一个命令）就能启动某个程序。`这两者的基本过程都是类似的，都需要查找程序在硬盘上的安装位置，然后将它们加载到内存运行。`

==换句话说，图形界面和命令行要达到的目的是一样的，都是让用户控制计算机。==

然而，真正能够控制计算机硬件（CPU、内存、显示器等）的只有操作系统内核（Kernel），`图形界面和命令行只是架设在用户和内核之间的一座桥梁`。

由于安全、复杂、繁琐等原因，用户不能直接接触内核（也没有必要），需要另外再开发一个程序，让用户直接使用这个程序；该程序的作用就是接收用户的操作（点击图标、输入命令），并进行简单的处理，然后再传递给内核，这样用户就能间接地使用操作系统内核了。`你看，在用户和内核之间增加一层“代理”，既能简化用户的操作，又能保障内核的安全，何乐不为呢？`

用户界面和命令行就是这个另外开发的程序，就是这层“代理”。在Linux下，这个命令行程序叫做 `Shell`。

> Shell 是一个应用程序，它连接了用户和 Linux 内核，让用户能够更加高效、安全、低成本地使用 Linux 内核，这就是 Shell 的本质。

Shell 本身并不是内核的一部分，它只是站在内核的基础上编写的一个应用程序，它和 QQ、迅雷、Firefox 等其它软件没有什么区别。然而 Shell 也有着它的特殊性，就是`开机立马启动，并呈现在用户面前`；用户通过 Shell 来使用 Linux，不启动 Shell 的话，用户就没办法使用 Linux。

#### 1.1.1Shell 是如何连接用户和内核的？

Shell 能够接收用户输入的命令，并对命令进行处理，处理完毕后再将结果反馈给用户，比如输出到显示器、写入到文件等，这就是大部分读者对 Shell 的认知。你看，我一直都在使用 Shell，哪有使用内核哦？我也没有看到 Shell 将我和内核连接起来呀？！

其实，Shell 程序本身的功能是很弱的，比如文件操作、输入输出、进程管理等都得依赖内核。`我们运行一个命令，大部分情况下 Shell 都会去调用内核暴露出来的接口，这就是在使用内核，只是这个过程被 Shell 隐藏了起来，它自己在背后默默进行，我们看不到而已。`

==接口其实就是一个一个的函数，使用内核就是调用这些函数。这就是使用内核的全部内容了吗？嗯，是的！除了函数，你没有别的途径使用内核。==

比如，我们都知道在 Shell 中输入`cat log.txt`命令就可以查看 log.txt 文件中的内容，然而，log.txt 放在磁盘的哪个位置？分成了几个数据块？在哪里开始？在哪里终止？如何操作探头读取它？这些底层细节 Shell 统统不知道的，它只能去调用内核提供的 `open()` 和 `read()` 函数，告诉内核我要读取 log.txt 文件，请帮助我，然后内核就乖乖地按照 Shell 的吩咐去读取文件了，并将读取到的文件内容交给 Shell，最后再由 Shell 呈现给用户（其实呈现到显示器上还得依赖内核）。整个过程中 Shell 就是一个“中间商”，它在用户和内核之间“倒卖”数据，只是用户不知道罢了。

#### 1.1.2Shell 还能连接其它程序

在 Shell 中输入的命令，有一部分是 Shell 本身自带的，这叫做`内置命令`；有一部分是`其它的应用程序`（一个程序就是一个命令），这叫做`外部命令`。

`Shell 本身支持的命令并不多，功能也有限，但是 Shell 可以调用其他的程序，每个程序就是一个命令，这使得 Shell 命令的数量可以无限扩展`，其结果就是 Shell 的功能非常强大，完全能够胜任 Linux 的日常管理工作，如文本或字符串检索、文件的查找或创建、大规模软件的自动部署、更改系统设置、监控服务器性能、发送报警邮件、抓取网页内容、压缩文件等。

更加惊讶的是，Shell 还可以让多个外部程序发生连接，在它们之间很方便地传递数据，也就是把一个程序的输出结果传递给另一个程序作为输入。

大家所说的 Shell 强大，并不是 Shell 本身功能丰富，而是它擅长使用和组织其他的程序。Shell 就是一个领导者，这正是 Shell 的魅力所在。

可以将 Shell 在整个 Linux 系统中的地位描述成下图所示的样子。注意“用户”和“其它应用程序”是通过虚线连接的，因为用户启动 Linux 后直接面对的是 Shell，通过 Shell 才能运行其它的应用程序。

![image-20230101104003814](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101104003814.png)

#### 1.1.3Shell 也支持编程

Shell 并不是简单的堆砌命令，我们还可以在 Shell 中编程，这和使用 C++、C#、Java、Python 等常见的编程语言并没有什么两样。

Shell 虽然没有 C++、Java、Python 等强大，但也支持了基本的编程元素，例如：

- if...else 选择结构，case...in 开关语句，for、while、until 循环；
- 变量、数组、字符串、注释、加减乘除、逻辑运算等概念；
- 函数，包括用户自定义的函数和内置函数（例如 printf、export、eval 等）。

`站在这个角度讲，Shell 也是一种编程语言，它的编译器（解释器）是 Shell 这个程序。`我们平时所说的 Shell，有时候是指连接用户和内核的这个程序，有时候又是指 Shell 编程。

`Shell 主要用来开发一些实用的、自动化的小工具，而不是用来开发具有复杂业务逻辑的中大型软件`，例如检测计算机的硬件参数、搭建 Web 运行环境、日志分析等，Shell 都非常合适。

#### 1.1.4Shell 是一种脚本语言

任何代码最终都要被“翻译”成二进制的形式才能在计算机中执行。有的编程语言，如 C/C++、Pascal、Go语言、汇编等，`必须在程序运行之前将所有代码都翻译成二进制形式，也就是生成可执行文件，用户拿到的是最终生成的可执行文件，看不到源码`。这个过程叫做`编译`（Compile），这样的编程语言叫做编译型语言，完成编译过程的软件叫做编译器（Compiler）。

而有的编程语言，如 Shell、JavaScript、Python、PHP等，需要`一边执行一边翻译，不会生成任何可执行文件，用户必须拿到源码才能运行程序`。程序运行后会即时翻译，翻译完一部分执行一部分，不用等到所有代码都翻译完。这个过程叫做`解释`，这样的编程语言叫做解释型语言或者脚本语言（Script），完成解释过程的软件叫做解释器。

编译型语言的优点是`执行速度快`、`对硬件要求低`、`保密性好`，适合开发操作系统、大型应用程序、数据库等。

脚本语言的优点是`使用灵活`、`部署容易`、`跨平台性好`，非常适合 Web 开发以及小工具的制作。

==Shell 就是一种脚本语言，我们编写完源码后不用编译，直接运行源码即可。==

### 1.2常用的Shell

常见的 Shell 有 sh、bash、csh、tcsh、ash 等。

- sh

sh 的全称是 Bourne shell，由 AT&T 公司的 Steve Bourne 开发，为了纪念他，就用他的名字命名了。`sh 是 UNIX 上的标准 shell，很多 UNIX 版本都配有 sh。sh 是第一个流行的 Shell。`

- csh

sh 之后另一个广为流传的 shell 是由柏克莱大学的 Bill Joy 设计的，这个 shell 的语法有点类似 C 语言，所以才得名为 C shell ，简称为 csh。Bill Joy 是一个风云人物，他创立了 BSD 操作系统，开发了 vi 编辑器，还是 Sun 公司的创始人之一。BSD 是 UNIX 的一个重要分支，后人在此基础上发展出了很多现代的操作系统，最著名的有 FreeBSD、OpenBSD 和 NetBSD，就连 Mac OS X 在很大程度上也基于 BSD。

- tcsh

`tcsh 是 csh 的增强版，加入了命令补全功能，提供了更加强大的语法支持。`

- ash

一个`简单的轻量级`的 Shell，占用资源少，适合运行于低内存环境，但是与下面讲到的 bash shell 完全兼容。

- bash

`bash shell 是 Linux 的默认 shell，本教程也基于 bash 编写。`bash 由 `GNU 组织开发`，保持了对 sh shell 的兼容性，是`各种 Linux 发行版默认配置的 shell`。bash 兼容 sh 意味着，针对 sh 编写的 Shell 代码可以不加修改地在 bash 中运行。

#### 1.2.1查看Shell

Shell 是一个程序，一般都是放在`/bin` 或者`/usr/bin` 目录下，当前 Linux 系统可用的 Shell 都记录在`/etc/shells` 文件中。/etc/shells是一个`纯文本文件`，你可以在图形界面下打开它，也可以使用 cat 命令查看它。

![image-20230101112245667](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101112245667.png)

在现代的 Linux 上，sh 已经被 bash 代替，`/bin/sh` 往往是指向`/bin/bash` 的符号链接。

![image-20230101112638957](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101112638957.png)

如果你希望查看当前 Linux 的默认 Shell，那么可以输出 `SHELL` 环境变量：

![image-20230101112720558](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101112720558.png)

`SHELL` 是 Linux `系统中的环境变量`，它指明了当前使用的 Shell 程序的位置，也就是使用的哪个 Shell。

### 1.3进入Shell的两种方式

在 Linux 发展的早期，唯一能用的工具就是 Shell，Linux 用户都是在 Shell 中输入文本命令，并查看文本输出；如果有必要的话，Shell 也能显示一些基本的图形。而如今 Linux 的环境已经完全不同，几乎所有的 Linux 发行版都使用某种图形桌面环境（例如 GNOME、KDE、Unity 等），这使得原生的 Shell 入口被隐藏了，进入 Shell 仿佛变得困难起来。

#### 1.3.1进入Linux控制台

一种进入 Shell 的方法是让 Linux 系统退出图形界面模式，进入控制台模式，这样一来，显示器上只有一个简单的带着白色文字的“黑屏”，就像图形界面出现之前的样子。这种模式称为 `Linux 控制台（Console）`。

现代 Linux 系统在启动时会自动创建几个`虚拟控制台（Virtual Console）`，其中一个供图形桌面程序使用，其他的保留原生控制台的样子。虚拟控制台其实就是 Linux 系统内存中运行的虚拟终端（Virtual Terminal）。

从图形界面模式进入控制台模式也很简单，往往按下`Ctrl + Alt + Fn(n=1,2,3,4,5...)`快捷键就能够来回切换。

例如，CentOS 在启动时会创建 6 个虚拟控制台，按下快捷键`Ctrl + Alt + Fn(n=2,3,4,5,6)`可以从图形界面模式切换到控制台模式，按下`Ctrl + Alt + F1`可以从控制台模式再切换回图形界面模式。也就是说，1 号控制台被图形桌面程序占用了。

下图就是进入了控制台模式：

![image-20230101113151120](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101113151120.png)

输入用户名和密码，登录成功后就可以进入 Shell 了。`$`是命令提示符，我们可以在它后面输入 Shell 命令。

`图形界面也是一个程序，会占用 CPU 时间和内存空间，当 Linux 作为服务器系统时，安装调试完毕后，应该让 Linux 运行在控制台模式下，以节省服务器资源。`正是由于这个原因，`很多服务器甚至不安装图形界面程序`，管理员只能使用命令来完成各项操作。

#### 1.3.2使用终端

进入 Shell 的另外一种方法是使用 `Linux 桌面环境中的终端模拟包`（Terminal emulation package），也就是我们常说的`终端（Terminal）`，这样在图形桌面中就可以使用 Shell。

以 CentOS 为例，可以在“应用程序”菜单中找到终端，如下图所示：

![image-20230101113335859](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101113335859.png)

打开终端后，就可以输入 Shell 命令了：

![image-20230101113353626](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101113353626.png)

CentOS 默认的图形界面程序是 `GNOME`，该终端模拟包也是 `GNOME` 自带的。

除了 GNOME 终端，Linux 还有其他的终端模拟包，例如：

- xterm 终端

`最古老最基础的 X Windows 桌面程序自带的终端模拟包就是 xterm。`xterm 在 X Windows 出现之前便已经存在了，默认包含在大多数 X Windows 中。`xterm 虽然没有太多炫目的特性，但是运行它不需要太多的资源，所以 xterm 在针对老硬件设计的 Linux 发行版中仍然很常见`，比如 fluxbox 图形桌面环境就用它作为默认的终端模拟包。

- Konsole 终端

KDE 桌面项目也开发了自己的终端模拟包，名为 Konsole。Konsole 整合了基本的 xterm 特性以及一些更高级的类似 Windows 应用程序的特性。

### 1.4Shell命令的基本格式

进入 Shell 之后第一眼看到的内容类似下面这种形式：

```shell
[mozhiyan@localhost ~]$
```

这叫做命令提示符，看见它就意味着可以输入命令了。命令提示符不是命令的一部分，它只是起到一个提示作用。

Shell 命令的基本格式如下：

```shell
command [选项] [参数]
```

`[]`表示`可选`的，也就是可有可无。有些命令不写选项和参数也能执行，有些命令在必要的时候可以附带选项和参数。

`ls` 是常用的一个命令，它属于`目录操作命令`，用来列出当前目录下的文件和文件夹。ls 可以附带选项，也可以不带，不带选项的写法为：

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ ls
abc          demo.sh    a.out         demo.txt
getsum       main.sh    readme.txt    a.sh
module.sh    log.txt    test.sh       main.c
```

先执行`cd demo`命令进入 demo 目录，这是我在自己的主目录下创建的文件夹，用来保存教学使用的各种代码和数据。

接着执行 ls 命令，它列出了 demo 目录下的所有文件，并且进行了格式对齐。

#### 1.4.1使用选项

`ls` 命令之后不加选项和参数也能执行，不过只能执行最基本的功能，即显示当前目录下的文件名。那么加入一个选项，会出现什么结果？

```shell
[mozhiyan@localhost demo]$ ls -l
总用量 140
-rwxrwxr-x. 1 mozhiyan mozhiyan 8675 4月   2 15:01 a.out
-rwxr-xr-x. 1 mozhiyan mozhiyan  116 4月   3 09:24 a.sh
-rw-rw-r--. 1 mozhiyan mozhiyan   44 4月   2 16:41 check.sh
-rw-r--r--. 1 mozhiyan mozhiyan  399 3月  11 17:12 demo.sh
-rw-rw-r--. 1 mozhiyan mozhiyan    4 4月   8 17:56 demo.txt
-rw-rw-r--. 1 mozhiyan mozhiyan    0 4月  15 17:26 log.txt
-rw-rw-r--. 1 mozhiyan mozhiyan  650 4月  10 11:06 main.c
-rwxrwxr-x. 1 mozhiyan mozhiyan   69 3月  26 10:13 main.sh
-rw-rw-r--. 1 mozhiyan mozhiyan  111 3月  26 09:56 module.sh
-rw-rw-r--. 1 mozhiyan mozhiyan  352 3月  22 17:40 out.log
-rw-rw-r--. 1 mozhiyan mozhiyan   61 4月  16 11:19 output.txt
-rw-r--r--. 1 mozhiyan mozhiyan    5 4月  11 15:16 readme.txt
-rwxr-xr-x. 1 mozhiyan mozhiyan   88 4月  15 17:23 test.sh
```

如果加一个`-l`选项，则可以看到显示的内容明显增多了。`-l`是长格式（long list）的意思，也就是`显示文件的详细信息`。

可以看到，选项的作用是调整命令功能。如果没有选项，那么命令只能执行最基本的功能；而一旦有选项，则能执行更多功能，或者显示更加丰富的数据。

**短格式选项和长格式选项**

Linux 的选项又分为短格式选项和长格式选项。

- 短格式选项是长格式选项的简写，用一个减号`-`和一个字母表示，例如`ls -l`。
- 长格式选项是完整的英文单词，用两个减号`--`和一个单词表示，例如`ls --all`。

一般情况下，短格式选项是长格式选项的缩写，也就是一个短格式选项会有对应的长格式选项。当然也有例外，比如 ls 命令的短格式选项`-l`就没有对应的长格式选项，所以具体的命令选项还需要通过帮助手册来查询。

#### 1.4.2使用参数

`参数是命令的操作对象`，一般情况下，文件、目录、用户和进程等都可以作为参数被命令操作。例如：

```shell
[mozhiyan@localhost demo]$ ls -l main.c
-rw-rw-r--. 1 mozhiyan mozhiyan 650 4月  10 11:06 main.c
```

但是为什么一开始 ls 命令可以省略参数？`那是因为有默认参数。`命令一般都需要加入参数，用于指定命令操作的对象是谁。如果可以省略参数，则一般都有默认参数。例如 ls：

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ ls
abc          demo.sh    a.out         demo.txt
getsum       main.sh    readme.txt    a.sh
module.sh    log.txt     test.sh      main.c
```

这个 ls 命令后面如果没有指定参数的话，默认参数是当前所在位置，所以会显示当前目录下的文件名。

**选项和参数一起使用**

Shell 命令可以同时附带选项和参数，例如：

```shell
[mozhiyan@localhost ~]$ echo "http://c.biancheng.net/shell/"
http://c.biancheng.net/shell/
[mozhiyan@localhost ~]$ echo -n "http://c.biancheng.net/shell/"
http://c.biancheng.net/shell/[mozhiyan@localhost ~]$
```

`-n`是 echo 命令的选项，`"http://c.biancheng.net/shell/"`是 echo 命令的参数，它们被同时用于 echo 命令。

echo 命令用来输出一个字符串，默认输出完成后会换行；给它增加`-n`选项，就不会换行了。

#### 1.4.3选项附带的参数

有些命令的选项后面也可以附带参数，这些参数用来补全选项，或者调整选项的功能细节。

例如，read 命令用来读取用户输入的数据，并把读取到的数据赋值给一个变量，它通常的用法为：

```shell
read str
```

str 为变量名。如果我们只是想读取固定长度的字符串，那么可以给 read 命令增加`-n`选项。比如读取一个字符作为性别的标志，那么可以这样写：

```shell
read -n 1 sex
```

`1`是`-n`选项的参数，`sex`是 read 命令的参数。

`-n`选项表示读取固定长度的字符串，那么它后面必然要跟一个数字用来指明长度，否则选项是不完整的。

#### 1.4.4总结

`Shell 命令的选项用于调整命令功能，而命令的参数是这个命令的操作对象。有些选项后面也需要附带参数，以补全命令的功能。`

### 1.5Shell的本质

Shell 命令分为两种：

- Shell 自带的命令称为内置命令，它在 Shell 内部可以通过函数来实现，当 Shell 启动后，这些命令所对应的代码（函数体代码）也被加载到内存中，所以使用内置命令是非常快速的。
- 更多的命令是外部的应用程序，一个命令就对应一个应用程序。运行外部命令要开启一个新的进程，所以效率上比内置命令差很多。

`用户输入一个命令后，Shell 先检测该命令是不是内置命令，如果是就执行，如果不是就检测有没有对应的外部程序：有的话就转而执行外部程序，执行结束后再回到 Shell；没有的话就报错，告诉用户该命令不存在。`

#### 1.5.1内置命令

`内置命令不宜过多，过多的内置命令会导致 Shell 程序本身体积膨胀，运行 Shell 程序后就会占用更多的内存。`Shell 是一个常驻内存的程序，占用过多内存会影响其它的程序。只有那些最常用的命令才有理由成为内置命令，比如 cd、kill、echo 等。

#### 1.5.2外部命令

外部命令可能是读者比较疑惑的，一个外部的应用程序究竟是如何变成一个 Shell 命令的呢？

`应用程序就是一个文件，只不过这个文件是可以执行的。`既然是文件，那么它就有一个名字，并且存放在文件系统中。用户在 Shell 中输入一个外部命令后，只是将可执行文件的名字告诉了 Shell，但是并没有告诉 Shell 去哪里寻找这个文件。

难道 Shell 要遍历整个文件系统，查看每个目录吗？这显然是不能实现的。

==为了解决这个问题，Shell 在启动文件中增加了一个叫做 PATH 的环境变量，该变量就保存了 Shell 对外部命令的查找路径，如果在这些路径下找不到同名的文件，Shell 也不会再去其它路径下查找了，它就直接报错。==

![image-20230101115207546](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101115207546.png)

不同的路径之间以`:`分隔。你看，Shell 只会在`几个固定的路径`中查找外部命令。

> 如果我们自己用 C 语言或者 C++ 编写一个应用程序，并将它放到这几个目录下面，那么我们的程序也会成为 Shell 命令。当然，你也可以修改 PATH 变量给它增加另外的路径。

我自己使用 C 语言编写了一个叫做 `getsum` 的程序，它用来计算从 m 累加到 n 的和，代码如下：

```c
#include<stdio.h>
#include<unistd.h>
#include<getopt.h>
#include<stdlib.h>

int main(int argc, vhar *argv[]) {
    int start = 0;
    int end = 0;
    int sum = 0;
    int opt;
    char *optstring = ":s:e:";

    while ((opt = getopt(argc, argv, optstring) != -1)) {
        switch (opt) {
            case 's' : start = atoi(optarg); break;
            case 'e' : end = atoi(optarg); break;
            case ':' : puts("Missing parameter"); exit(1);
        }
    }

    if (start < 0 || end <= start) {
        puts("Parameter error");
        exit(2);
    }

    for (int i = start; i <= end; i++) {
        sum += i;
    }

    printf("%d\n", sum);
    return 0;
}  
```

将这段代码编译成名为 `getsum` 的应用程序，并放在`~/bin` 目录（~ 表示用户主目录）下，然后在 Shell 中输入下面的命令，就可以计算 1+2+3 ...... +99+100 的值。

```shell
[mozhiyan@localhost ~]$ getsum -s 1 -e 100
5050
```

-s 选项表示起始数字，-e 选项表示终止数字。

#### 1.5.3总结

Shell 内置命令的本质是一个自带的函数，执行内置命令就是调用这个自带的函数。因为函数代码在 Shell 启动时已经被加载到内存了，所以内置命令的执行速度很快。

Shell 外部命令的本质是一个应用程序，执行外部命令就是启动一个新的应用程序。因为要创建新的进程并加载应用程序的代码，所以外部命令的执行速度很慢。

### 1.6Shell命令的选项和参数的本质

很多 Shell 命令都是可以附带选项和参数的，不同的选项和参数也使得命令的功能细节有所差异。

Shell 命令附带参数的例子：

```shell
cd demo 命令表示进入当前目录下的 demo 目录，其中 demo 就是 cd 命令的参数。
echo "123xyz"命令表示输出字符串并换行，其中"123xyz"就是 echo 命令的参数。
```

Shell 命令附带选项的例子：

```shell
ls -l 命令用来显示当前目录下的所有文件以及它们的详细信息，其中-l 就是 ls 命令的选项。
echo -n "http://c.biancheng.net/shell/"表示在输出字符串后不换行，其中-n 是 echo 命令的选项
```

有些命令的选项后面也可以附带参数：

```shell
getsum -s 1 -e 100 命令用来计算从 1 累加到 100 的和，其中-s 和-e 是 getsum 命令的选项，1 和 100 分别是-s 和-e 选项的参数。
read -n 1 sex 命令用来读取一个字符并赋值给 sex 变量，其中-n 是 read 命令的选项，1 是-n 选项的参数，sex 是 read 命令的参数。
```

你是否对这些形形色色的选项和参数感到好奇？你是否想知道它们在底层是如何实现的？你是否也想自己动手对它们进行解析？本节就来给你揭晓答案!

死磕这个细节并不是闲得无聊，它能帮助我们理解命令的真正含义。好了，废话不多说，让我们赶紧转入正题吧。

上节我们讲到，`一个 Shell 内置命令就是一个内部的函数，一个外部命令就是一个应用程序`。==内置命令后面附带的所有数据（所有选项和参数）最终都以参数的形式传递给了函数，外部命令后面附带的所有数据（所有选项和参数）最终都以参数的形式传递给了应用程序。==

> 也就是说，不管是内置命令还是外部命令，它后面附带的所有数据都会被“打包”成参数，这些参数有的传递给了函数，有的传递给了应用程序。

有编程经验的读者应该知道，C 语言或者 C++ 程序的入口函数是 `int main(int argc, char *argv[])`，传递给应用程序的参数最终都被`main` 函数接收了。从这个角度看，传递给应用程序的参数其实也是传递给了函数。

有了以上认知，我们就不用再区分函数和应用程序了，我们就认为：`不管是内置命令还是外部命令，它后面附带的数据最终都以参数的形式传递给了函数。`实现一个命令的一项重要工作就是解析传递给函数的参数。

==注意，命令后面附带的数据并不是被合并在一起，作为一个参数传递给函数的；这些数据是由空格分隔的，它们被分隔成了几份，就会转换成几个参数。==例如 getsum -s 1 -e 100 要向函数传递四个参数，read -n 1 sex 要向函数中传递三个参数。

并且，命令后面附带的数据都是“原汁原味”地传递给了函数，比如 getsum -s 1 -e 100 要传递的四个参数分别是 -s、1、-e、100，`减号-也会一起传递过去`，在函数内部，`减号-可以用来区分该参数是否是命令的选项`。

至于在函数内部如何解析这些参数，对于外部命令来说那就是 C/C++ 程序员的工作了，这里不再过多赘述，只给出演示代码。上节我给大家演示了一个 `getsum` 程序，本节依然使用该程序演示参数的解析，只是对代码进行了微调。

```shell
#include<stdio.h>
#include<unistd.h>
#include<getopt.h>
#include<stdlib.h>
int main(int argc, vhar *argv[]) {
    int start = 0;
    int end = 0;
    int sum = 0;
    int opt;
    char *optstring = ":s:e:";
    // 分析接收到的参数
    while ((opt = getopt(argc, argv, optstring) != -1)) {
        switch (opt) {
            case 's' : start = atoi(optarg); break;
            case 'e' : end = atoi(optarg); break;
            case ':' : puts("Missing parameter"); exit(1);
        }
    }
    
    
    // 检测参数是否有效
    if (start < 0 || end <= start) {
        puts("Parameter error");
        exit(2);
    }
     
    // 打印接收到的参数
    printf("Received parameters:");
    for (int i = 0; i < argc; i++) {
        printf("%s ", argv[i]);
    }
    printf("\n");
    
    // 计算累积的和
    for (int i = start; i <= end; i++) {
        sum += i;
    }
    printf("%d\n", sum);
    
    return 0;
}
```

第 11~20 行是解析参数的关键代码，getopt.h 头文件中的 getopt() 函数是值得重点研究的，有了该函数我们就不用自己去解析参数了，省了很大的力气。

第 27~32 行将接收到的参数打印出来，以便读者更好地观察。

```shell
[mozhiyan@localhost ~]$ getsum -s 1 -e 100
Received parameters: getsum -s 1 -e 100 
sum=5050
```

### 1.7Shell命令提示符

启动 `Linux 桌面环境自带的终端模拟包`，或者从 Linux `控制台登录`后，便可以看到 `Shell 命令提示符`。看见命令提示符就意味着可以输入命令了。命令提示符不是命令的一部分，它只是起到一个提示作用。

不同的 Linux 发行版使用的提示符格式大同小异，例如在 CentOS 中，默认的提示符类似下面这样：

```shell
[mozhiyan@localhost ~]$
```

各个部分的含义如下：

- `[]`是提示符的分隔符号，没有特殊含义。
- `mozhiyan`表示当前登录的用户，我现在使用的是 mozhiyan 用户登录。

- `@`是分隔符号，没有特殊含义。
- `localhost`表示当前系统的简写主机名。

- `~`代表用户当前所在的目录为主目录（home 目录）。如果用户当前位于主目录下的 bin 目录中，那么这里显示的就是`bin`。
- `$`是命令提示符。Linux 用这个符号标识登录的用户权限等级：如果是超级用户（root 用户），提示符就是`#`；如果是普通用户，提示符就是`$`。

总结起来，Linux Shell 默认的命令提示符的格式为：

```shell
[username@host directory]$
```

或者

```shell
[username@host directory]#
```

**什么是主目录？**

Linux 系统是纯字符界面，用户登录后，要有一个`初始登录的位置`，这个初始登录位置就称为用户的主目录（home 目录）。超级用户的主目录为`/root/`，普通用户的主目录为`/home/用户名/`。

```tex
有的资料也称为“家目录”，“家”是 home 的直译，它们都是一个意思。
```

`用户在自己的主目录中拥有完整权限`，所以我们也建议操作实验可以放在主目录中进行。

我们使用 cd 命令切换一下用户所在目录，看看有什么效果。

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ cd /usr/local
[mozhiyan@localhost local]$
```

仔细看，如果切换用户所在目录，那么命令提示符中会变成用户`当前所在目录的最后一个目录`（不显示完整的所在目录 `/usr/ local/`，只显示最后一个目录 `local`）。

#### 1.7.1第二层命令提示符

有些命令不能在一行内输入完成，需要换行，这个时候就会看到第二层命令提示符。第二层命令提示符默认为`>`，请看下面的例子：

```shell
[mozhiyan@localhost ~]$ echo "Shell教程"
Shell教程
[mozhiyan@localhost ~]$ echo "
> http://
> c.biancheng.net
> "

http://
c.biancheng.net

```

第一个 echo 命令在一行内输入完成，不会出现第二层提示符。第二个 echo 命令需要多行才能输入完成，提示符`>`用来告诉用户命令还没输入完成，请继续输入。

echo 命令用来输出一个字符串。字符串是一组由`" "`包围起来的字符序列，echo 将第一个`"`作为字符串的开端，将第二个`"`作为字符串的结尾。对于第二个 echo 命令，我们将字符串分成多行，echo 遇到第一个`"`认为是不完整的字符串，所以会继续等待用户输入，直到遇见第二个`"`。

> 命令提示符的格式不是固定的，用户可以根据自己的喜好来修改。

### 1.8Shell修改命令提示符

Shell 通过 `PS1` 和 `PS2` 这两个环境变量来控制提示符的格式，修改 `PS1` 和 `PS2` 的值就能修改命令提示符的格式。

- PS1 控制最外层的命令提示符格式。

- PS2 控制第二层的命令提示符格式。

在修改 PS1 和 PS2 之前，我们先用 echo 命令输出它们的值，看看默认情况下是什么样子的：

![image-20230101145629391](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101145629391.png)

Linux 使用以\为前导的特殊字符来表示命令提示符中包含的要素，这使得 PS1 和 PS2 的格式看起来可能有点奇怪。下表展示了可以在`PS1` 和 `PS2` 中使用的特殊字符。

| 字符 | 描述                                                         |
| ---- | ------------------------------------------------------------ |
| \a   | 铃声字符                                                     |
| \d   | 格式为“日 月 年”的日期                                       |
| \h   | 本地主机名                                                   |
| \H   | 完全合格的限定域主机名                                       |
| \j   | shell 当前管理的作业数                                       |
| \l   | shell 终端设备名的基本名称                                   |
| \n   | ASCII 换行字符                                               |
| \r   | ASCII 回车字符                                               |
| \s   | shell 的名称                                                 |
| \t   | 格式为“小时:分钟:秒”的 24 小时制的当前时间                   |
| \T   | 格式为“小时:分钟:秒”的 12 小时制的当前时间                   |
| \u   | 当前用户的用户名                                             |
| \v   | bash shell 的版本                                            |
| \V   | bash shell 的发布级别                                        |
| \w   | 当前工作目录                                                 |
| \W   | 当前工作目录的基本名称                                       |
| \\$  | 如果是普通用户，则为美元符号$；如果超级用户（root 用户），则为井号#。 |
| \xxx | 对应于八进制值 xxx 的字符                                    |
| \\\\ | 斜杠                                                         |

注意，所有的特殊字符均以反斜杠`\`开头，目的是与普通字符区分开来。您可以在命令提示符中使用以上任何特殊字符的组合。

【实例】通过修改 PS1 变量的值来修改命令提示符的格式：

![image-20230101150911374](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101150911374.png)

遗憾的是，通过这种方式修改的命令提示符只在当前的 Shell 会话期间有效，再次启动 Shell 后将重新使用默认的命令提示符。

如果希望持久性地修改 PS1，让它对任何 Shell 会话都有效，那么就得把 PS1 变量的修改写入到 Shell 启动文件中。

### 1.9第一个Shell脚本

几乎所有编程语言的教程都是从使用著名的“Hello World”开始的，出于对这种传统的尊重（或者说落入俗套），我们的第一个 Shell 脚本也输出“Hello World”。

打开文本编辑器，新建一个文本文件，并命名为 `test.sh`。

> 扩展名`sh`代表 shell，扩展名并不影响脚本执行，见名知意就好，如果你用 php 写 shell 脚本，扩展名就用`php`好了。

在 `test.sh` 中输入代码：

```shell
#!/bin/bash
echo "Hello World !"  #这是一条语句
```

第 1 行的`#!`是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种 Shell；后面的`/bin/bash`就是指明了解释器的具体位置。

第 2 行的 echo 命令用于向`标准输出`文件（Standard Output，stdout，`一般就是指显示器`）输出文本。在`.sh`文件中使用命令与在终端直接输入命令的效果是一样的。

第 2 行的`#`及其后面的内容是注释。Shell 脚本中所有以`#`开头的都是注释（当然以`#!`开头的除外）。写脚本的时候，多写注释是非常有必要的，以方便其他人能看懂你的脚本，也方便后期自己维护时看懂自己的脚本——实际上，即便是自己写的脚本，在经过一段时间后也很容易忘记。

下面给出了一段稍微复杂的 Shell 脚本：

```shell
#!/bin/bash
echo "What is your name?"
read PERSON
echo "Hello, $PERSON"
```

第 3 行中表示从终端读取用户输入的数据，并赋值给 PERSON 变量。read 命令用来从`标准输入`文件（Standard Input，stdin，`一般就是指键盘`）读取用户输入的数据。

第 4 行表示输出变量 PERSON 的内容。注意在变量名前边要加上`$`，否则变量名会作为字符串的一部分处理。

### 1.10执行Shell脚本

上节我们编写了一个简单的 Shell 脚本，这节我们就让它运行起来。运行 Shell 脚本有两种方法，一种在新进程中运行，一种是在当前 Shell 进程中运行。

#### 1.10.1在新进程中运行 Shell 脚本

在新进程中运行 Shell 脚本有多种方法。

**1)将 Shell 脚本作为程序运行**

Shell 脚本也是一种解释执行的程序，可以在终端直接调用（需要使用 chmod 命令给 Shell 脚本加上执行权限），如下所示：

```shell
[mozhiyan@localhost ~]$ cd demo          #切换到 test.sh 所在的目录
[mozhiyan@localhost demo]$ chmod +x ./test.sh  #给脚本添加执行权限
[mozhiyan@localhost demo]$ ./test.sh           #执行脚本文件
Hello World !                                  #运行结果
```

第 2 行中，`chmod +x`表示给 test.sh 增加执行权限。

第 3 行中，`./`表示当前目录，整条命令的意思是执行当前目录下的 `test.sh` 脚本。如果不写`./`，Linux 会到系统路径（`由 PATH 环境变量指定`）下查找 `test.sh`，而系统路径下显然不存在这个脚本，所以会执行失败。

通过这种方式运行脚本，脚本文件第一行的`#!/bin/bash`一定要写对，好让系统查找到正确的解释器。

**2)将 Shell 脚本作为参数传递给 Bash 解释器**

你也可以直接运行 Bash 解释器，将脚本文件的名字作为参数传递给 Bash，如下所示：

```shell
[mozhiyan@localhost ~]$ cd demo              #切换到 test.sh 所在的目录
[mozhiyan@localhost demo]$ /bin/bash test.sh  #使用Bash的绝对路径
Hello World !                                 #运行结果
```

通过这种方式运行脚本，不需要在脚本文件的第一行指定解释器信息，写了也没用。

更加简洁的写法是运行 `bash` 命令。bash 是一个`外部命令`，Shell 会在 `/bin` 目录中找到对应的应用程序，也即 `/bin/bash`。

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ bash test.sh
Hello World !
```

这两种写法在本质上是一样的：第一种写法给出了绝对路径，会直接运行 Bash 解释器；第二种写法通过 bash 命令找到 Bash 解释器所在的目录，然后再运行，只不过多了一个查找的过程而已。

:feet: **检测是否开启了新进程**

有些读者可能会疑问，你怎么知道开启了新进程？你有什么证据吗？既然如此，那我就来给大家验证一下吧。

Linux 中的每一个进程都有一个唯一的 ID，称为 PID，使用`$$`变量就可以获取当前进程的 PID。`$$`是 Shell 中的特殊变量。

首先编写如下的脚本文件，并命名为 `check.sh`：

```shell
#!/bin/bash
echo $$  #输出当前进程PID
```

然后使用以上两种方式来运行 `check.sh`：

```shell
[mozhiyan@localhost demo]$ echo $$
2861  #当前进程的PID
[mozhiyan@localhost demo]$ chmod +x ./check.sh
[mozhiyan@localhost demo]$ ./check.sh
4597  #新进程的PID
[mozhiyan@localhost demo]$ echo $$
2861  #当前进程的PID
[mozhiyan@localhost demo]$ /bin/bash check.sh
4584  #新进程的PID
```

你看，进程的 PID 都不一样，当然就是两个进程了。

### 1.10.2在当前进程中运行Shell脚本

这里需要引入一个新的命令——`source` 命令。`source` 是 `Shell 内置命令`的一种，它会读取脚本文件中的代码，并依次执行所有语句。你也可以理解为，source 命令会强制执行脚本文件中的全部命令，而忽略脚本文件的权限。

source 命令的用法为：

```shell
source filename
```

也可以简写为：

```shell
. filename
```

两种写法的效果相同。对于第二种写法，注意点号`.`和文件名中间有一个空格。

例如，使用 source 运行上节的 test.sh：

```shell
[mozhiyan@localhost ~]$ cd demo              #切换到test.sh所在的目录
[mozhiyan@localhost demo]$ source ./test.sh  #使用source
Hello World !
[mozhiyan@localhost demo]$ source test.sh    #使用source
Hello World !
[mozhiyan@localhost demo]$ . ./test.sh       #使用点号
Hello World !
[mozhiyan@localhost demo]$ . test.sh         #使用点号
Hello World !
```

你看，使用 source 命令不用给脚本增加执行权限，并且写不写`./`都行，是不是很方便呢？

:dizzy: **检测是否在当前 Shell 进程中**

我们仍然借助`$$`变量来输出进程的 PID，如下所示：

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ echo $$
5169  #当前进程PID
[mozhiyan@localhost demo]$ source ./check.sh
5169  #Shell脚本所在进程PID
[mozhiyan@localhost demo]$ echo $$
5169  #当前进程PID
[mozhiyan@localhost demo]$ . ./check.sh
5169  #Shell脚本所在进程PID
```

你看，进程的 PID 都是一样的，当然是同一个进程了。

#### 1.10.3总结

如果需要在新进程中运行 Shell 脚本，我一般使用`bash test.sh`这种写法；如果在当前进程中运行 Shell 脚本，我一般使用`. ./test.sh`这种写法。这是我个人的风格。

最后再给大家演示一个稍微复杂的例子。本例中使用 read 命令从键盘读取用户输入的内容并赋值给 URL 变量，最后在显示器上输出。

```shell
#!/bin/bash
# Copyright (c) http://c.biancheng.net/shell/
echo "What is the url of the shell tutorial?"
read URL
echo "$URL is very fast!"
```

运行脚本：

```shell
[mozhiyan@localhost demo]$ . ./test.sh
What is the url of the shell tutorial?
http://c.biancheng.net/shell/↙
http://c.biancheng.net/shell/ is very fast!
```

↙ 表示按下回车键。

### 1.11Shell的四种运行方式

Shell 是一个应用程序，它的一端连接着 Linux 内核，另一端连接着用户。Shell 是用户和 Linux 系统沟通的桥梁，我们都是通过 Shell 来管理 Linux 系统。

`我们可以直接使用 Shell，也可以输入用户名和密码后再使用 Shell`；第一种叫做`非登录式`，第二种叫做`登录式`。

`我们可以在 Shell 中一个个地输入命令并及时查看它们的输出结果，整个过程都在跟 Shell 不停地互动，这叫做交互式。我们也可以运行一个 Shell 脚本文件，让所有命令批量化、一次性地执行，这叫做非交互式。`

总起来说，Shell 一共有四种运行方式：

- 交互式的登录 Shell；
- 交互式的非登录 Shell；
- 非交互式的登录 Shell；
- 非交互式的非登录 Shell。

#### 1.11.1判断Shell是否是交互式

判断是否为交互式 Shell 有两种简单的方法。

1)查看变量`-`的值，如果值中包含了字母 `i`，则表示`交互式（interactive）`。

【实例 1】在 CentOS GNOME 桌面环境自带的终端下输出`-`的值：

![image-20230101170537165](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101170537165.png)

包含了 `i`，为交互式。

【实例 2】在 Shell 脚本文件中输出`-`的值：

![image-20230101170819705](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101170819705.png)

不包含 `i`，为非交互式。注意，必须在新进程中运行 Shell 脚本。

![image-20230101170917422](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101170917422.png)

2)查看变量 `PS1` 的值，如果非空，则为交互式，否则为非交互式，`因为非交互式会清空该变量`。

【实例 1】在 CentOS GNOME 桌面环境自带的终端下输出 PS1 的值：

![image-20230101171026425](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101171026425.png)

非空，为交互式。

【实例 2】在 Shell 脚本文件中输出 PS1 的值：

![image-20230101171131661](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101171131661.png)

空值，为非交互式。注意，必须在新进程中运行 Shell 脚本。

![image-20230101171155962](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101171155962.png)

#### 1.11.2判断Shell是否是登录式

判断 Shell 是否为登录式也非常简单，只需执行 `shopt login_shell` 即可，值为 `on`表示为登录式，`off `为非登录式。

shopt 命令用来查看或设置 Shell 中的行为选项，这些选项可以增强 Shell 的易用性。

【实例 1】在 CentOS GNOME 桌面环境自带的终端下查看 login_shell 选项：

![image-20230101182055638](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101182055638.png)

【实例 2】在 Shell 脚本文件中查看 login_shel 选项：

![image-20230101182321940](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101182321940.png)

#### 1.11.3同时判断交互式、登录式

要同时判断是否为交互式和登录式，可以简单使用如下的命令：

```shell
echo $PS1; shopt login_shell
```

或者

```shell
echo $-; shopt login_shell
```

#### 1.11.4常见的Shell启动方式

1)通过 Linux 控制台（不是桌面环境自带的终端）或者 ssh 登录 Shell 时（这才是正常登录方式），为交互式的登录 Shell。

![image-20230101182644956](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101182644956.png)

2)执行 bash 命令时默认是非登录的，增加`--login` 选项（简写为`-l`）后变成登录式。

![image-20230101182902137](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101182902137.png)

3)使用由`()`包围的`组命令`或者`命令替换`进入子 Shell 时，子 Shell 会继承父 Shell 的交互和登录属性。

![image-20230101183153943](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101183153943.png)

4)ssh 执行远程命令，但不登录时，为非交互非登录式。

![image-20230101183433153](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101183433153.png)

### 1.12Shell配置文件加载

无论是否是交互式，是否是登录式，Bash Shell 在启动时总要配置其运行环境，例如初始化环境变量、设置命令提示符、指定系统命令路径等。这个过程是通过加载一系列配置文件完成的，这些配置文件其实就是 Shell 脚本文件。

与 Bash Shell 有关的配置文件主要有 `/etc/profile`、`~/.bash_profile`、 `~/.bash_login`、` ~/.profile`、` ~/.bashrc`、`/etc/bashrc`、`/etc/profile.d/*.sh`，不同的启动方式会加载不同的配置文件。

`~`表示用户`主目录`。`*`是通配符，/etc/profile.d/*.sh 表示 /etc/profile.d/ 目录下所有的脚本文件（以`.sh` 结尾的文件）。

#### 1.12.1登录式的Shell

Bash 官方文档说：如果是登录式的 Shell，首先会读取和执行 /etc/profiles，这是所有用户的全局配置文件，接着会到用户主目录中寻找 ~/.bash_profile、 ~/.bash_login 或者 ~/.profile，它们都是用户个人的配置文件。

不同的 Linux 发行版附带的个人配置文件也不同，有的可能只有其中一个，有的可能三者都有，笔者使用的是 CentOS 7，该发行版只有 ~/.bash_profile，其它两个都没有。

如果三个文件同时存在的话，到底应该加载哪一个呢？它们的优先级顺序是 ~/.bash_profile > ~/.bash_login > ~/.profile。

如果 ~/.bash_profile 存在，那么一切以该文件为准，并且到此结束，不再加载其它的配置文件。

如果 ~/.bash_profile 不存在，那么尝试加载 ~/.bash_login。 ~/.bash_login 存在的话就到此结束，不存在的话就加载 ~/.profile。

注意，/etc/profiles 文件还会嵌套加载 /etc/profile.d/*.sh，请看下面的代码：

```shell
for i in /etc/profile.d/*.sh ; do
 if [ -r "$i" ]; then
 if [ "${-#*i}" != "$-" ]; then
 . "$i"
 else
 . "$i" >/dev/null
 fi
 fi
done
```

同样，~/.bash_profile 也使用类似的方式加载 ~/.bashrc：

```shell
if [ -f ~/.bashrc ]; then
. ~/.bashrc
fi
```

#### 1.12.2非登录的Shell

如果以非登录的方式启动 Shell，那么就不会读取以上所说的配置文件，而是直接读取 ~/.bashrc。

~/.bashrc 文件还会嵌套加载 /etc/bashrc，请看下面的代码：

```shell
if [ -f /etc/bashrc ]; then
. /etc/bashrc
fi
```

### 1.13编写自己的Shell配置文件

学习了《Shell 配置文件的加载》一节，读者应该知道 Shell 在登录和非登录时都会加载哪些配置文件了。==对于普通用户来说，也许 ~/.bashrc 才是最重要的文件，因为不管是否登录都会加载该文件。==

我们可以将自己的一些代码添加到 `~/.bashrc`，这样每次启动 Shell 都可以个性化地配置。如果你有代码洁癖，也可以将自己编写的代码放到一个新文件中（假设叫 myconf.sh），只要在 `~/.bashrc` 中使用类似`. ./myconf.sh` 的形式将新文件引入进来就行了

#### 1.13.1示例一：给PATH变量增加新路径

你曾经是否感到迷惑，Shell 是怎样知道去哪里找到我们输入的命令的？例如，当我们输入 ls 后，Shell 不会查找整个计算机系统，而是在指定的几个目录中检索（最终在 /bin/ 目录中找到了 ls 程序），这些目录就包含在 PATH 变量中。

当用户登录 Shell 时，PATH 变量会在 /etc/profile 文件中设置，然后在 ~/.bash_profile 也会增加几个目录。如果没有登录 Shell，PATH 变量会在 /etc/bashrc 文件中设置。

如果我们想增加自己的路径，可以将该路径放在 ~/.bashrc 文件中，例如：

```shell
PATH=$PATH:$HOME/addon
```

将主目录下的 addon 目录也设置为系统路径。假如此时在 addon 目录下有一个 getsum 程序，它的作用是计算从 m 累加到 n 的和，那么我们不用 cd 到 addon 目录，直接输入 getsum 命令就能得到结果。

#### 1.13.2示例二：修改命令提示符的格式

在《修改 Linux 命令提示符》一节中我曾提到，修改 PS1 变量的值就可以修改命令提示符的格式，但是那个时候大家还不了解 Shell 启动文件，所以只能临时性地修改，并不能持久。

现在我们已经知道，在 `~/.bashrc` 文件中修改 PS1 变量的值就可以`持久化`，每个使用 Shell 的用户都会看见新的命令提示符。

将下面的代码添加到 `~/.bashrc` 文件中，然后重新启动 Shell，命令提示符就变成了`[c.biancheng.net]$`。

```shell
PS1="[c.biancheng.net]\$ "
```

## 2.Shell编程

### 2.1Shell变量

变量是任何一种编程语言都必不可少的组成部分，变量用来存放各种数据。脚本语言在定义变量时通常不需要指明类型，直接赋值就可以，Shell 变量也遵循这个规则。

在 Bash shell 中，每一个变量的值都是`字符串`，无论你给变量赋值时有没有使用引号，值都会以字符串的形式存储。

这意味着，`Bash shell 在默认情况下不会区分变量类型`，即使你将整数和小数赋值给变量，它们也会被视为字符串，这一点和大部分的编程语言不同。例如在C语言或者 C++ 中，变量分为整数、小数、字符串、布尔等多种类型。

当然，如果有必要，你也可以使用 `Shell declare 关键字`显式定义变量的类型，但在一般情况下没有这个需求，Shell 开发者在编写代码时自行注意值的类型即可。

#### 2.1.1定义变量

Shell 支持以下三种定义变量的方式：

```shell
variable=value
variable='value'
variable="value"
```

variable 是变量名，value 是赋给变量的值。如果 value 不包含任何空白符（例如空格、Tab 缩进等），那么可以不使用引号；如果 value 包含了空白符，那么就必须使用引号包围起来。使用`单引号`和使用`双引号`也是有区别的，稍后我们会详细说明。

> 注意，赋值号`=`的周围不能有空格，这可能和你熟悉的大部分编程语言都不一样。

Shell 变量的命名规范和大部分编程语言都一样：

- 变量名由数字、字母、下划线组成；
- 必须以字母或者下划线开头；
- 不能使用 Shell 里的关键字（通过 help 命令可以查看保留关键字）。

变量定义举例：

```shell
url=http://c.biancheng.net/shell/
echo $url
name='C语言中文网'
echo $name
author="严长生"
echo $author
```

#### 2.1.2使用变量

使用一个定义过的变量，只要在变量名前面加美元符号`$`即可，如：

```shell
author="严长生"
echo $author
echo ${author}
```

变量名外面的花括号`{ }`是可选的，加不加都行，加花括号是为了帮助解释器识别变量的边界，比如下面这种情况：

```sjell
skill="Java"
echo "I am good at ${skill}Script"
```

如果不给 skill 变量加花括号，写成`echo "I am good at $skillScript"`，解释器就会把 `$skillScript` 当成一个变量（其值为空），代码执行结果就不是我们期望的样子了。

> 推荐给所有变量加上花括号`{ }`，这是个良好的编程习惯。

#### 2.1.3修改变量的值

已定义的变量，可以被重新赋值，如：

```shell
url="http://c.biancheng.net"
echo ${url}
url="http://c.biancheng.net/shell/"
echo ${url}
```

第二次对变量赋值时不能在变量名前加`$`，只有在使用变量时才能加`$`。

#### 2.1.4单引号和双引号的区别

前面我们还留下一个疑问，定义变量时，变量的值可以由单引号`' '`包围，也可以由双引号`" "`包围，它们到底有什么区别呢？不妨以下面的代码为例来说明：

```shell
#!/bin/bash
url="http://c.biancheng.net"
website1='C语言中文网：${url}'
website2="C语言中文网：${url}"
echo $website1
echo $website2

运行结果：
C语言中文网：${url}
C语言中文网：http://c.biancheng.net
```

以单引号`' '`包围变量的值时，单引号里面是什么就输出什么，即使内容中有变量和命令（命令需要反引起来）也会把它们原样输出。这种方式比较适合定义显示纯字符串的情况，即不希望解析变量、命令等的场景。

以双引号`" "`包围变量的值时，输出时会先解析里面的变量和命令，而不是把双引号中的变量名和命令原样输出。这种方式比较适合字符串中附带有变量和命令并且想将其解析后再输出的变量定义。

> 我的建议：如果变量的内容是数字，那么可以不加引号；如果真的需要原样输出就加单引号；其他没有特别要求的字符串等最好都加上双引号，定义变量时加双引号是最常见的使用场景。

#### 2.1.5将命令的结果赋值给变量

Shell 也支持将命令的执行结果赋值给变量，常见的有以下两种方式：

```shell
variable=`command`
variable=$(command)
```

第一种方式把命令用反引号 \` \`（位于 Esc 键的下方）包围起来，反引号和单引号非常相似，容易产生混淆，所以不推荐使用这种方式；第二种方式把命令用`$()`包围起来，区分更加明显，所以推荐使用这种方式。

例如，我在 demo 目录中创建了一个名为 log.txt 的文本文件，用来记录我的日常工作。下面的代码中，使用 cat 命令将 log.txt 的内容读取出来，并赋值给一个变量，然后使用 echo 命令输出。

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ log=$(cat log.txt)
[mozhiyan@localhost demo]$ echo $log
严长生正在编写Shell教程，教程地址：http://c.biancheng.net/shell/
[mozhiyan@localhost demo]$ log=`cat log.txt`
[mozhiyan@localhost demo]$ echo $log
严长生正在编写Shell教程，教程地址：http://c.biancheng.net/shell/
```

#### 2.1.6只读变量

使用 `readonly 命令`可以将变量定义为只读变量，只读变量的值不能被改变。

下面的例子尝试更改只读变量，结果报错：

```shell
#!/bin/bash
myUrl="http://c.biancheng.net/shell/"
readonly myUrl
myUrl="http://c.biancheng.net/shell/"
```

运行脚本，结果如下：

```shell
bash: myUrl: This variable is read only.
```

#### 2.1.7删除变量

使用 **unset** 命令可以删除变量。语法：

```shell
unset variable_name
```

变量被删除后不能再次使用；unset 命令不能删除只读变量。

举个例子：

```shell
#!/bin/sh
myUrl="http://c.biancheng.net/shell/"
unset myUrl
echo $myUrl
```

上面的脚本没有任何输出。

### 2.2Shell变量的作用域

Shell 变量的作用域（Scope），就是 Shell 变量的有效范围（可以使用的范围）。Shell 变量的作用域可以分为三种：

- 有的变量只能在函数内部使用，这叫做局部变量（local variable）；
- 有的变量可以在当前 Shell 进程中使用，这叫做全局变量（global variable）；
- 而有的变量还可以在子进程中使用，这叫做环境变量（environment variable）。

#### 2.2.1局部变量

Shell 也支持自定义函数，但是 Shell 函数和 C++、Java、C# 等其他编程语言函数的一个不同点就是：`在 Shell 函数中定义的变量默认也是全局变量`，它和在函数外部定义变量拥有一样的效果。请看下面的代码：

```shell
#!/bin/bash
#定义函数
function func(){
	a=99
}
#调用函数
func
#输出函数内部的变量
echo $a
输出结果：
99
```

a 是在函数内部定义的，但是在函数外部也可以得到它的值，证明它的作用域是全局的，而不是仅限于函数内部。

要想变量的作用域仅限于函数内部，可以在定义时加上 `local 命令`，此时该变量就成了局部变量。请看下面的代码：

```shell
#!/bin/bash
#定义函数
function func(){
	local a=99
}
#调用函数
func
#输出函数内部的变量
echo $a
输出结果：

```

输出结果为空，表明变量 a 在函数外部无效，是一个局部变量。Shell 变量的这个特性和 JavaScript 中的变量是类似的。在 JavaScript 函数内部定义的变量，默认也是全局变量，只有加上 var 关键字，它才会变成局部变量。

#### 2.2.2Shell全局变量

`所谓全局变量，就是指变量在当前的整个 Shell 进程中都有效。`每个 Shell 进程都有自己的作用域，彼此之间互不影响。在 Shell 中定义的变量，默认就是全局变量。

想要实际演示全局变量在不同 Shell 进程中的互不相关性，可在图形界面下同时打开两个 Shell，或使用两个终端远程连接到服务器（SSH）。

首先打开一个 Shell 窗口，定义一个变量 a 并赋值为 99，然后打印，这时在同一个 Shell 窗口中是可正确打印变量 a 的值的。然后再打开一个新的 Shell 窗口，同样打印变量 a 的值，但结果却为空，如图 1 所示。

![image-20230102102340322](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102102340322.png)

这说明全局变量 a 仅仅在定义它的第一个 Shell 进程中有效，对新的 Shell 进程没有影响。这很好理解，就像小王家和小徐家都有一部电视机（变量名相同），但是同一时刻小王家和小徐家的电视中播放的节目可以是不同的（变量值不同）。

> 需要强调的是，`全局变量的作用范围是当前的 Shell 进程，而不是当前的 Shell 脚本文件，它们是不同的概念`。打开一个 Shell 窗口就创建了一个 Shell 进程，打开多个 Shell 窗口就创建了多个 Shell 进程，每个 Shell 进程都是独立的，拥有不同的进程 ID。在一个Shell 进程中可以使用 source 命令执行多个 Shell 脚本文件，此时全局变量在这些脚本文件中都有效。

例如，现在有两个 Shell 脚本文件，分别是 a.sh 和 b.sh。a.sh 的代码如下：

```shell
#!/bin/bash
echo $a
b=200
```

b.sh 的代码如下：

```shell
#!/bin/bash
echo $b
```

打开一个 Shell 窗口，输入以下命令:

```shell
[c.biancheng.net]$ a=99
[c.biancheng.net]$ . ./a.sh
99
[c.biancheng.net]$ . ./b.sh
200
```

`这三条命令都是在一个进程中执行的`，从输出结果可以发现，在 Shell 窗口中以命令行的形式定义的变量 a，在 a.sh 中有效；在 a.sh 中定义的变量 b，在 b.sh 中也有效，变量 b 的作用范围已经超越了 a.sh。

> 注意，必须在当前进程中运行 Shell 脚本，不能在新进程中运行 Shell 脚本

#### 2.2.3Shell环境变量

全局变量只在当前 Shell 进程中有效，对其它 Shell 进程和子进程都无效。如果使用 `export 命令`将全局变量导出，那么它就在所有的子进程中也有效了，这称为“环境变量”。

`环境变量被创建时所处的 Shell 进程称为父进程`，如果在父进程中再创建一个新的进程来执行 Shell 命令，那么这个新的进程被称作`Shell 子进程`。当 Shell 子进程产生时，它会继承父进程的环境变量为自己所用，所以说`环境变量可从父进程传给子进程`。不难理解，环境变量还可以传递给孙进程。

注意，`两个没有父子关系的 Shell 进程是不能传递环境变量的`，并且环境变量只能向下传递而不能向上传递，即“传子不传父”。

创建 Shell 子进程最简单的方式是运行 bash 命令，如图 2 所示。

![image-20230102102926545](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102102926545.png)

通过 `exit 命令`可以一层一层地退出 Shell。

下面演示一下环境变量的使用：

```shell
[c.biancheng.net]$ a=22 #定义一个全局变量
[c.biancheng.net]$ echo $a #在当前 Shell 中输出 a，成功
22
[c.biancheng.net]$ bash #进入 Shell 子进程
[c.biancheng.net]$ echo $a #在子进程中输出 a，失败
[c.biancheng.net]$ exit #退出 Shell 子进程，返回上一级 Shell
exit
[c.biancheng.net]$ export a #将 a 导出为环境变量
[c.biancheng.net]$ bash #重新进入 Shell 子进程
[c.biancheng.net]$ echo $a #在子进程中再次输出 a，成功
22
[c.biancheng.net]$ exit #退出 Shell 子进程
exit
[c.biancheng.net]$ exit #退出父进程，结束整个 Shell 会话
```

可以发现，默认情况下，a 在 Shell 子进程中是无效的；使用 export 将 a 导出为环境变量后，在子进程中就可以使用了。

`export a` 这种形式是在定义变量 a 以后再将它导出为环境变量，如果想在定义的同时导出为环境变量，可以写作 `export a=22`。

我们一直强调的是`环境变量在 Shell 子进程中有效`，并没有说它在所有的 Shell 进程中都有效；如果你通过终端创建了一个新的 Shell 窗口，那它就不是当前 Shell 的子进程，环境变量对这个新的 Shell 进程仍然是无效的。请看下图：

![image-20230102103204749](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102103204749.png)

第一个窗口中的环境变量 a 在第二个窗口中就无效。

**环境变量也是临时的**

通过 export 导出的环境变量只对当前 Shell 进程以及所有的子进程有效，如果最顶层的父进程被关闭了，那么环境变量也就随之消失了，其它的进程也就无法使用了，所以说环境变量也是临时的。

有读者可能会问，如果我想让一个变量在所有 Shell 进程中都有效，不管它们之间是否存在父子关系，该怎么办呢？

> 只有将变量写入 Shell 配置文件中才能达到这个目的！Shell 进程每次启动时都会执行配置文件中的代码做一些初始化工作，如果将变量放在配置文件中，那么每次启动进程都会定义这个变量。

### 2.3Shell命令的替换

Shell 命令替换是指将命令的输出结果赋值给某个变量。比如，在某个目录中输入 ls 命令可查看当前目录中所有的文件，但如何将输出内容存入某个变量中呢？这就需要使用命令替换了，这也是 Shell 编程中使用非常频繁的功能。

Shell 中有两种方式可以完成命令替换，一种是反引号\` \`，一种是`$()`，使用方法如下：

```shell
variable=`commands`
variable=$(commands)
```

其中，variable 是变量名，commands 是要执行的命令。commands 可以只有一个命令，也可以有多个命令，多个命令之间以分号`;`分隔。

例如，date 命令用来获得当前的系统时间，使用命令替换可以将它的结果赋值给一个变量。

```shell
#!/bin/bash
begin_time=`date`    #开始时间，使用``替换
sleep 20s            #休眠20秒
finish_time=$(date)  #结束时间，使用$()替换
echo "Begin time: $begin_time"
echo "Finish time: $finish_time"
运行脚本，20 秒后可以看到输出结果：
Begin time: 2019年 04月 19日 星期五 09:59:58 CST
Finish time: 2019年 04月 19日 星期五 10:00:18 CST
```

使用 data 命令的`%s`格式控制符可以得到当前的 UNIX 时间戳，这样就可以直接计算脚本的运行时间了。UNIX 时间戳是指从 1970 年 1 月 1 日 00:00:00 到目前为止的秒数。 

```shell
#!/bin/bash
begin_time=`date +%s`    #开始时间，使用``替换
sleep 20s                #休眠20秒
finish_time=$(date +%s)  #结束时间，使用$()替换
run_time=$((finish_time - begin_time))  #时间差
echo "begin time: $begin_time"
echo "finish time: $finish_time"
echo "run time: ${run_time}s"
运行脚本，20 秒后可以看到输出结果：
begin time: 1555639864
finish time: 1555639884
run time: 20s
```

第 5 行代码中的`(( ))`是 Shell 数学计算命令。和 [C++](http://c.biancheng.net/cplus/)、[C#](http://c.biancheng.net/csharp/)、[Java](http://c.biancheng.net/java/) 等编程语言不同，在 Shell 中进行数据计算不那么方便，必须使用专门的数学计算命令，`(( ))`就是其中之一。

注意，如果被替换的命令的输出内容包括多行（也即有换行符），或者含有多个连续的空白符，那么在输出变量时应该将变量用双引号包围，否则系统会使用默认的空白符来填充，这会导致换行无效，以及连续的空白符被压缩成一个。请看下面的代码：

```shell
#!/bin/bash
LSL=`ls -l`
echo $LSL  #不使用双引号包围
echo "--------------------------"  #输出分隔符
echo "$LSL"  #使用引号包围
```

运行结果：

```shell
total 8 drwxr-xr-x. 2 root root 21 7月 1 2016 abc -rw-rw-r--. 1 mozhiyan mozhiyan 147 10月 31 10:29 demo.sh -rw-rw-r--. 1 mozhiyan mozhiyan 35 10月 31 10:20 demo.sh~
--------------------------
total 8
drwxr-xr-x. 2 root     root      21 7月   1 2016 abc
-rw-rw-r--. 1 mozhiyan mozhiyan 147 10月 31 10:29 demo.sh
-rw-rw-r--. 1 mozhiyan mozhiyan  35 10月 31 10:20 demo.sh~
```

所以，为了防止出现格式混乱的情况，`建议在输出变量时加上双引号`。

#### 2.3.1再谈反引号和$()

原则上讲，上面提到的两种变量替换的形式是等价的，可以随意使用；但是，反引号毕竟看起来像单引号，有时候会对查看代码造成困扰，而使用 \$()就相对清晰，能有效避免这种混乱。而且有些情况必须使用\$()：$() 支持嵌套，反引号不行。

下面的例子演示了使用计算 ls 命令列出的第一个文件的行数，这里使用了两层嵌套。

```shell
[c.biancheng.net]$ Fir_File_Lines=$(wc -l $(ls | sed -n '1p'))
[c.biancheng.net]$ echo "$Fir_File_Lines"
36 anaconda-ks.cfg
```

要注意的是，$() 仅在 Bash Shell 中有效，而反引号可在多种 Shell 中使用。所以这两种命令替换的方式各有特点，究竟选用哪种方式全看个人需求。

### 2.4Shell位置参数

运行 Shell 脚本文件时我们可以给它传递一些参数，这些参数在脚本文件内部可以使用`$n`的形式来接收，例如，\$1 表示第一个参数，$2 表示第二个参数，依次类推。

同样，在调用函数时也可以传递参数。Shell 函数参数的传递和其它编程语言不同，没有所谓的形参和实参，在定义函数时也不用指明参数的名字和数目。==换句话说，定义 Shell 函数时不能带参数，但是在调用函数时却可以传递参数，这些传递进来的参数，在函数内部就也使用\$n的形式接收，例如，\$1 表示第一个参数，$2 表示第二个参数，依次类推。==

这种通过`$n`的形式来接收的参数，在 Shell 中称为位置参数。

在讲解变量的命名时，我们提到：变量的名字必须以字母或者下划线开头，不能以数字开头；但是位置参数却偏偏是数字，这和变量的命名规则是相悖的，所以我们将它们视为“特殊变量”。

除了 \$n，Shell 中还有 \$#、\$*、\$@、\$?、$$ 几个特殊参数，我们将在下节讲解。

#### 2.4.1给脚本文件传递位置参数

请编写下面的代码，并命名为 test.sh：

```shell
#!/bin/bash
echo "Language: $1"
echo "URL: $2"
```

运行 test.sh，并附带参数：

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ . ./test.sh Shell http://c.biancheng.net/shell/
Language: Shell
URL: http://c.biancheng.net/shell/
```

其中`Shell`是第一个位置参数，`http://c.biancheng.net/shell/`是第二个位置参数，两者之间以空格分隔。

#### 2.4.2给函数传递位置参数

请编写下面的代码，并命名为 test.sh：

```shell
#!/bin/bash
#定义函数
function func(){
    echo "Language: $1"
    echo "URL: $2"
}
#调用函数
func C++ http://c.biancheng.net/cplus/
```

运行 test.sh：

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ . ./test.sh
Language: C++
URL: http://c.biancheng.net/cplus/
```

**注意事项**

如果参数个数太多，达到或者超过了 10 个，那么就得用`${n}`的形式来接收了，例如 \${10}、${23}。`{ }`的作用是为了帮助解释器识别参数的边界，这跟使用变量时加`{ }`是一样的效果。

在 Shell 中，传递位置参数时除了能单独取得某个具体的参数，还能取得所有参数的列表，以及参数的个数等信息，下节我们将会详细讲解。

### 2.5Shell特殊变量

| 变量 | 含义                                                         |
| ---- | ------------------------------------------------------------ |
| $0   | 当前脚本的文件名。                                           |
| $n   | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是 \$1，第二个参数是 $2。 |
| $#   | 传递给脚本或函数的参数个数。                                 |
| $*   | 传递给脚本或函数的所有参数。                                 |
| $@   | 传递给脚本或函数的所有参数。当被双引号`" "`包含时，\$@ 与 \$* 稍有不同。 |
| $?   | 上个命令的退出状态，或函数的返回值。                         |
| $$   | 当前 Shell 进程 ID。对于 Shell 脚本，就是这些脚本所在的进程 ID。 |

下面我们通过两个例子来演示。

#### 2.5.1给脚本文件传递参数

编写下面的代码，并保存为 test.sh：

```shell
#!/bin/bash
echo "Process ID: $$"
echo "File Name: $0"
echo "First Parameter : $1"
echo "Second Parameter : $2"
echo "All parameters 1: $@"
echo "All parameters 2: $*"
echo "Total: $#"
```

运行 test.sh，并附带参数：

```shell
[mozhiyan@localhost demo]$ . ./test.sh Shell Linux
Process ID: 5943
File Name: bash
First Parameter : Shell
Second Parameter : Linux
All parameters 1: Shell Linux
All parameters 2: Shell Linux
Total: 2
```

#### 2.5.2给函数传递参数

 编写下面的代码，并保存为 test.sh：

```shell
#!/bin/bash
#定义函数
function func(){
    echo "Language: $1"
    echo "URL: $2"
    echo "First Parameter : $1"
    echo "Second Parameter : $2"
    echo "All parameters 1: $@"
    echo "All parameters 2: $*"
    echo "Total: $#"
}
#调用函数
func Java http://c.biancheng.net/java/
```

```shell
运行结果为：
Language: Java
URL: http://c.biancheng.net/java/
First Parameter : Java
Second Parameter : http://c.biancheng.net/java/
All parameters 1: Java http://c.biancheng.net/java/
All parameters 2: Java http://c.biancheng.net/java/
Total: 2
```

### 2.6Shell \$*和\$@直接的区别

 \$和\$@都表示传递给函数或脚本的所有参数，本节重点说一下它们之间的区别。当 \$ 和 \$@ 不被双引号" "包围时，它们之间没有任何区别，都是将接收到的每个参数看做一份数据，彼此之间以空格来分隔。但是当它们被双引号" "包含时，就会有区别了：

- "$*"会将所有的参数从整体上看做一份数据，而不是把每个参数都看做一份数据。
- "$@"仍然将每个参数都看作一份数据，彼此之间是独立的。

比如传递了 5 个参数，那么对于"\$*"来说，这 5 个参数会合并到一起形成一份数据，它们之间是无法分割的；而对于"\$@"来说，这 5 个参数是相互独立的，它们是 5 份数据。

> 如果使用 echo 直接输出"\$*"和"\$@"做对比，是看不出区别的；但如果使用 for 循环来逐个输出数据，立即就能看出区别来。

编写下面的代码，并保存为 test.sh：

```shell
#!/bin/bash
echo "print each param from \"\$*\""
for var in "$*"
do
	echo "$var"
done

echo "print each param from \"\$@\""
for var in "$@"
do
	echo "$var"
done
```

运行 test.sh，并附带参数：

```shell
[mozhiyan@localhost demo]$ . ./test.sh a b c d
print each param from "$*"
a b c d
print each param from "$@"
a
b
c
d
```

从运行结果可以发现，对于"\$*"，只循环了 1 次，因为它只有 1 分数据；对于"\$@"，循环了 5 次，因为它有 5 份数据。

### 2.7$?:获取函数返回值或者上一个命令的退出状态

\$? 是一个特殊变量，用来获取上一个命令的退出状态，或者上一个函数的返回值。`所谓退出状态，就是上一个命令执行后的返回结果。退出状态是一个数字，一般情况下，大部分命令执行成功会返回 0，失败返回 1，这和C语言的 main() 函数是类似的。`不过，也有一些命令返回其他值，表示不同类型的错误。

#### 2.7.1获取上一个命令的退出状态

编写下面的代码，并保存为 test.sh：

```shell
#!/bin/bash
if [ "$1" == 100 ]
then
   exit 0  #参数正确，退出状态为0
else
   exit 1  #参数错误，退出状态1
fi
```

`exit`表示退出当前 Shell 进程，我们必须在新进程中运行 test.sh，否则当前 Shell 会话（终端窗口）会被关闭，我们就无法取得它的退出状态了。

例如，运行 test.sh 时传递参数 100：

```shell
[mozhiyan@localhost ~]$ cd demo
[mozhiyan@localhost demo]$ bash ./test.sh 100  #作为一个新进程运行
[mozhiyan@localhost demo]$ echo $?
0
```

再如，运行 test.sh 时传递参数 89：

```shell
[mozhiyan@localhost demo]$ bash ./test.sh 89  #作为一个新进程运行
[mozhiyan@localhost demo]$ echo $?
1
```

#### 2.7.2获取函数的返回值

编写下面的代码，并保存为 test.sh：

```shell
#!/bin/bash
#得到两个数相加的和
function add(){
    return `expr $1 + $2`
}
add 23 50  #调用函数
echo $?  #获取函数返回值
运行结果：
73
```

有 C++、C#、Java 等编程经验的读者请注意：`严格来说，Shell 函数中的 return 关键字用来表示函数的退出状态，而不是函数的返回值`；Shell 不像其它编程语言，没有专门处理返回值的关键字。

### 2.8Shell字符串详解

字符串（String）就是一系列字符的组合。字符串是 Shell 编程中最常用的数据类型之一（除了数字和字符串，也没有其他类型了）。

字符串可以由单引号`' '`包围，也可以由双引号`" "`包围，也可以不用引号。它们之间是有区别的，稍后我们会详解。

字符串举例：

```shell
str1=c.biancheng.net
str2="shell script"
str3='C语言中文网'
```

下面我们说一下三种形式的区别：

1)由单引号`' '`包围的字符串：

- 任何字符都会原样输出，在其中使用变量是无效的。
- 字符串中不能出现单引号，即使对单引号进行转义也不行。

2)由双引号`" "`包围的字符串：

- 如果其中包含了某个变量，那么该变量会被解析（得到该变量的值），而不是原样输出。
- 字符串中可以出现双引号，只要它被转义了就行。

3)不被引号包围的字符串

- 不被引号包围的字符串中出现变量时也会被解析，这一点和双引号`" "`包围的字符串一样。
- 字符串中不能出现空格，否则空格后边的字符串会作为其他变量或者命令解析。

我们通过代码来演示一下三种形式的区别：

```shell
#!/bin/bash
n=74
str1=c.biancheng.net$n str2="shell \"script\" $n"
str3='C语言中文网 $n'
echo $str1
echo $str2
echo $str3
```

运行结果：

```shell
c.biancheng.net74
shell "script" 74
C语言中文网 $n
```

str1 中包含了`$n`，它被解析为变量 n 的引用。`$n`后边有空格，紧随空格的是 str2；Shell 将 str2 解释为一个新的变量名，而不是作为字符串 str1 的一部分。

str2 中包含了引号，但是被转义了（由反斜杠`\`开头的表示转义字符）。str2 中也包含了`$n`，它也被解析为变量 n 的引用。

str3 中也包含了`$n`，但是仅仅是作为普通字符，并没有解析为变量 n 的引用。

#### 2.8.1获取字符串的长度

在 Shell 中获取字符串长度很简单，具体方法如下：

```shell
${#string_name}
```

string_name 表示字符串名字。下面是具体的演示：

```shell
#!/bin/bash
str="http://c.biancheng.net/shell/"
echo ${#str}
运行结果：
29
```

### 2.9字符串的拼接

在脚本语言中，字符串的拼接（也称字符串连接或者字符串合并）往往都非常简单，例如：

- 在 PHP 中，使用.即可连接两个字符串；
- 在 JavaScript 中，使用+即可将两个字符串合并为一个。

然而，在 Shell 中你不需要使用任何运算符，将两个字符串并排放在一起就能实现拼接，非常简单粗暴。请看下面的例子：

```shell
#!/bin/bash
name="Shell"
url="http://c.biancheng.net/shell/"
str1=$name$url  #中间不能有空格
str2="$name $url"  #如果被双引号包围，那么中间可以有空格
str3=$name": "$url  #中间可以出现别的字符串
str4="$name: $url"  #这样写也可以
str5="${name}Script: ${url}index.html"  #这个时候需要给变量名加上大括号
echo $str1
echo $str2
echo $str3
echo $str4
echo $str5
```

```shell
运行结果：
Shellhttp://c.biancheng.net/shell/
Shell http://c.biancheng.net/shell/
Shell: http://c.biancheng.net/shell/
Shell: http://c.biancheng.net/shell/
ShellScript: http://c.biancheng.net/shell/index.html
```

对于第 4 行代码，\$name 和 \$url 之间之所以不能出现空格，是因为当字符串不被任何一种引号包围时，遇到空格就认为字符串结束了，空格后边的内容会作为其他变量或者命令解析。

对于第 8 行代码，加`{ }`是为了帮助解释器识别变量的边界。

### 2.10Shell字符串截取

Shell 截取字符串通常有两种方式：从指定位置开始截取和从指定字符（子字符串）开始截取。

#### 2.10.1从指定位置开始截取

这种方式需要两个参数：除了指定起始位置，还需要截取长度，才能最终确定要截取的字符串。既然需要指定起始位置，那么就涉及到计数方向的问题，到底是从字符串左边开始计数，还是从字符串右边开始计数。答案是 Shell 同时支持两种计数方式。

**1)从字符串左边开始计数**

如果想从字符串的左边开始计数，那么截取字符串的具体格式如下：

```shell
${string: start :length}
```

其中，string 是要截取的字符串，start 是起始位置（从左边开始，从 0 开始计数），length 是要截取的长度（省略的话表示直到字符串的末尾）。例如：

```shell
url="c.biancheng.net"
echo ${url: 2: 9}
```

结果为`biancheng`。

再如：

```shell
url="c.biancheng.net"
echo ${url: 2}  #省略 length，截取到字符串末尾
```

结果为`biancheng.net`。

**2)从右边开始计数**

如果想从字符串的右边开始计数，那么截取字符串的具体格式如下：

```shell
${string: 0-start :length}
```

同第 1) 种格式相比，第 2) 种格式仅仅多了`0-`，这是固定的写法，专门用来表示从字符串右边开始计数。

这里需要强调两点：

- 从左边开始计数时，起始数字是 0（这符合程序员思维）；从右边开始计数时，起始数字是 1（这符合常人思维）。计数方向不同，起始数字也不同。
- 不管从哪边开始计数，截取方向都是从左到右。

```shell
url="c.biancheng.net"
echo ${url: 0-13: 9}
```

结果为`biancheng`。从右边数，`b`是第 13 个字符。

再如：

```shell
url="c.biancheng.net"
echo ${url: 0-13}  #省略 length，直接截取到字符串末尾
```

结果为`biancheng.net`。

#### 2.10.2从指定字符开始截取

这种截取方式无法指定字符串长度，只能从指定字符（子字符串）截取到字符串末尾。Shell 可以截取指定字符（子字符串）右边的所有字符，也可以截取左边的所有字符。

**1)使用 # 号截取右边字符**

使用`#`号可以截取指定字符（或者子字符串）右边的所有字符，具体格式如下：

```shell
${string#*chars}
```

其中，string 表示要截取的字符，chars 是指定的字符（或者子字符串），`*`是通配符的一种，表示任意长度的字符串。`*chars`连起来使用的意思是：忽略左边的所有字符，直到遇见 chars（chars 不会被截取）。

请看下面的例子：

```shell
url="http://c.biancheng.net/index.html"
echo ${url#*:}
```

结果为`//c.biancheng.net/index.html`。

以下写法也可以得到同样的结果：

```shell
echo ${url#*p:}
echo ${url#*ttp:}
```

如果不需要忽略 chars 左边的字符，那么也可以不写`*`，例如：

```shell
url="http://c.biancheng.net/index.html"
echo ${url#http://}
```

结果为`c.biancheng.net/index.html`。

注意，以上写法遇到第一个匹配的字符（子字符串）就结束了。例如：

```shell
url="http://c.biancheng.net/index.html"
echo ${url#*/}
```

结果为`/c.biancheng.net/index.html`。url 字符串中有三个`/`，输出结果表明，Shell 遇到第一个`/`就匹配结束了。

如果希望直到最后一个指定字符（子字符串）再匹配结束，那么可以使用`##`，具体格式为：

```shell
${string##*chars}
```

请看下面的例子：

```shell
#!/bin/bash
url="http://c.biancheng.net/index.html"
echo ${url#*/}    #结果为 /c.biancheng.net/index.html
echo ${url##*/}   #结果为 index.html
str="---aa+++aa@@@"
echo ${str#*aa}   #结果为 +++aa@@@
echo ${str##*aa}  #结果为 @@@
```

2)使用 % 截取左边字符

使用`%`号可以截取指定字符（或者子字符串）左边的所有字符，具体格式如下：

```shell
${string%chars*}
```

请注意`*`的位置，因为要截取 chars 左边的字符，而忽略 chars 右边的字符，所以`*`应该位于 chars 的右侧。其他方面`%`和`#`的用法相同，这里不再赘述，仅举例说明：

```shell
#!/bin/bash
url="http://c.biancheng.net/index.html"
echo ${url%/*}  #结果为 http://c.biancheng.net
echo ${url%%/*}  #结果为 http:
str="---aa+++aa@@@"
echo ${str%aa*}  #结果为 ---aa+++
echo ${str%%aa*}  #结果为 ---
```

#### 2.10.3总结

最后，我们对以上 8 种格式做一个汇总，请看下表：

| 格式                       | 说明                                                         |
| -------------------------- | ------------------------------------------------------------ |
| ${string: start :length}   | 从 string 字符串的左边第 start 个字符开始，向右截取 length 个字符。 |
| ${string: start}           | 从 string 字符串的左边第 start 个字符开始截取，直到最后。    |
| ${string: 0-start :length} | 从 string 字符串的右边第 start 个字符开始，向右截取 length 个字符。 |
| ${string: 0-start}         | 从 string 字符串的右边第 start 个字符开始截取，直到最后。    |
| ${string#*chars}           | 从 string 字符串第一次出现 *chars 的位置开始，截取 *chars 右边的所有字符。 |
| ${string##*chars}          | 从 string 字符串最后一次出现 *chars 的位置开始，截取 *chars 右边的所有字符。 |
| ${string%chars*}           | 从 string 字符串第一次出现 chars* 的位置开始，截取 chars* 左边的所有字符。 |
| ${string%%chars*}          | 从 string 字符串最后一次出现 chars* 的位置开始，截取 chars* 左边的所有字符。 |

### 2.11Shell数组

和其他编程语言一样，Shell 也支持数组。数组（Array）是若干数据的集合，其中的每一份数据都称为元素（Element）。

Shell 并且没有限制数组的大小，理论上可以存放无限量的数据。和 C++、Java、C# 等类似，Shell 数组元素的下标也是从 0 开始计数。获取数组中的元素要使用下标`[ ]`，下标可以是一个整数，也可以是一个结果为整数的表达式；当然，下标必须大于等于 0。

`遗憾的是，常用的 Bash Shell 只支持一维数组，不支持多维数组。·`

#### 2.11.1Shell数组的定义

在 Shell 中，用括号`( )`来表示数组，数组元素之间用空格来分隔。由此，定义数组的一般形式为：

```shell
array_name=(ele1  ele2  ele3 ... elen)
```

注意，赋值号`=`两边不能有空格，必须紧挨着数组名和数组元素。

下面是一个定义数组的实例：

```shell
nums=(29 100 13 8 91 44)
```

Shell 是弱类型的，它并不要求所有数组元素的类型必须相同，例如：

```shell
arr=(20 56 "http://c.biancheng.net/shell/")
```

第三个元素就是一个“异类”，前面两个元素都是整数，而第三个元素是字符串。

> Shell 数组的长度不是固定的，定义之后还可以增加元素。例如，对于上面的 nums 数组，它的长度是 6，使用下面的代码会在最后增加一个元素，使其长度扩展到 7：

```shell
nums[6]=88
```

此外，你也无需逐个元素地给数组赋值，下面的代码就是只给特定元素赋值：

```shell
ages=([3]=24 [5]=19 [10]=12)
```

以上代码就只给第 3、5、10 个元素赋值，所以数组长度是 3。

#### 2.11.2获取数组元素

获取数组元素的值，一般使用下面的格式：

```shell
${array_name[index]}
```

其中，array_name 是数组名，index 是下标。例如：

```shell
n=${nums[2]}
```

表示获取 nums 数组的第二个元素，然后赋值给变量 n。再如：

```shell
echo ${nums[3]}
```

表示输出 nums 数组的第 3 个元素。

使用`@`或`*`可以获取数组中的所有元素，例如：

```shell
${nums[*]}
${nums[@]}
```

两者都可以得到 nums 数组的所有元素。

完整的演示：

```shell
#!/bin/bash
nums=(29 100 13 8 91 44)
echo ${nums[@]}  #输出所有数组元素
nums[10]=66  #给第10个元素赋值（此时会增加数组长度）
echo ${nums[*]}  #输出所有数组元素
echo ${nums[4]}  #输出第4个元素
```

```shell
运行结果：
29 100 13 8 91 44
29 100 13 8 91 44 66
91
```

### 2.12Shell获取数组长度

所谓数组长度，就是数组元素的个数。

利用`@`或`*`，可以将数组扩展成列表，然后使用`#`来获取数组元素的个数，格式如下：

```shell
${#array_name[@]}
${#array_name[*]}
```

其中 array_name 表示数组名。两种形式是等价的，选择其一即可。

如果某个元素是字符串，还可以通过指定下标的方式获得该元素的长度，如下所示：

```shell
${#arr[2]}
```

获取 arr 数组的第 2 个元素（假设它是字符串）的长度。

**回忆字符串长度的获取**

回想一下 Shell 是如何获取字符串长度的呢？其实和获取数组长度如出一辙，它的格式如下：

```shell
${#string_name}
```

string_name 是字符串名。

#### 2.12.1示例演示

下面我们通过实际代码来演示一下如何获取数组长度。

```shell
#!/bin/bash
nums=(29 100 13)
echo ${#nums[*]}
#向数组中添加元素
nums[10]="http://c.biancheng.net/shell/"
echo ${#nums[@]}
echo ${#nums[10]}
#删除数组元素
unset nums[1]
echo ${#nums[*]}

运行结果：
3
4
29
3
```

### 2.13Shell数组的拼接

所谓 Shell 数组拼接（数组合并），就是将两个数组连接成一个数组。

拼接数组的思路是：先利用`@`或`*`，将数组扩展成列表，然后再合并到一起。具体格式如下：

```shell
array_new=(${array1[@]}  ${array2[@]})
array_new=(${array1[*]}  ${array2[*]})
```

两种方式是等价的，选择其一即可。其中，array1 和 array2 是需要拼接的数组，array_new 是拼接后形成的新数组。

下面是完整的演示代码：

```shell
#!/bin/bash
array1=(23 56)
array2=(99 "http://c.biancheng.net/shell/")
array_new=(${array1[@]} ${array2[*]})
echo ${array_new[@]}  #也可以写作 ${array_new[*]}

运行结果：
23 56 99 http://c.biancheng.net/shell/
```

### 2.14Shell删除数组元素

在 Shell 中，使用 unset 关键字来删除数组元素，具体格式如下：

```shell
unset array_name[index]
```

其中，array_name 表示数组名，index 表示数组下标。如果不写下标，而是写成下面的形式：

```shell
unset array_name
```

那么就是删除整个数组，所有元素都会消失。

下面我们通过具体的代码来演示：

```shell
#!/bin/bash
arr=(23 56 99 "http://c.biancheng.net/shell/")
unset arr[1]
echo ${arr[@]}
unset arr
echo ${arr[*]}
```

运行结果：

```shell
23 99 http://c.biancheng.net/shell/
 
```

注意最后的空行，它表示什么也没输出，因为数组被删除了，所以输出为空。

### 2.15Shell关联数组

现在最新的 Bash Shell 已经支持关联数组了。关联数组使用字符串作为下标，而不是整数，这样可以做到见名知意。

关联数组也称为“键值对（key-value）”数组，键（key）也即字符串形式的数组下标，值（value）也即元素值。

例如，我们可以创建一个叫做 color 的关联数组，并用颜色名字作为下标。

```shell
declare -A color
color["red"]="#ff0000"
color["green"]="#00ff00"
color["blue"]="#0000ff"
```

也可以在定义的同时赋值：

```shell
declare -A color=(["red"]="#ff0000", ["green"]="#00ff00", ["blue"]="#0000ff")
```

`不同于普通数组，关联数组必须使用带有-A 选项的 declare 命令创建。`

#### 2.15.1访问关联数组元素

访问关联数组元素的方式几乎与普通数组相同，具体形式为：

```shell
array_name["index"]
```

```shell
color["white"]="#ffffff"
color["black"]="#000000"
```

加上$()即可获取数组元素的值：

```shell
$(array_name["index"])
```

```shell
echo $(color["white"])
white=$(color["black"])
```

### 2.15.2获取所有元素的下标和值

使用下面的形式可以获得关联数组的所有元素值：

```shell
${array_name[@]}
${array_name[*]}
```

使用下面的形式可以获取关联数组的所有下标值：

```sjell
${!array_name[@]}
${!array_name[*]}
```

#### 2.15.3获取关联数组的长度

使用下面的形式可以获得关联数组的长度：

```shell
${#array_name[*]}
${#array_name[@]}
```

关联数组实例演示：

```shell
#!/bin/bash

declare -A color
color["red"]="#ff0000"
color["green"]="#00ff00"
color["blue"]="#0000ff"
color["white"]="#ffffff"
color["black"]="#000000"

#获取所有元素值
for value in ${color[*]}
do
	echo $value
done
echo "****************"
#获取所有元素下标（键）
for key in ${!color[*]}
do
	echo $key
done
echo "****************"
#列出所有键值对
for key in ${!color[@]}
do
	echo "${key} -> ${color[$key]}"
done
```

### 2.16Shell内置命令

`所谓 Shell 内建命令，就是由 Bash 自身提供的命令，而不是文件系统中的某个可执行文件。`例如，用于进入或者切换目录的 cd 命令，虽然我们一直在使用它，但如果不加以注意很难意识到它与普通命令的性质是不一样的：该命令并不是某个外部文件，只要在 Shell 中你就一定可以运行这个命令。

可以使用 type 来确定一个命令是否是内建命令：

```shell
[root@localhost ~]# type cd
cd is a Shell builtin
[root@localhost ~]# type ifconfig
ifconfig is /sbin/ifconfig
```

由此可见，cd 是一个 Shell 内建命令，而 ifconfig 是一个外部文件，它的位置是`/sbin/ifconfig`。

还记得系统变量 \$PATH 吗？\$PATH 变量包含的目录中几乎聚集了系统中绝大多数的可执行命令，它们都是外部命令。

> 通常来说，内建命令会比外部命令执行得更快，执行外部命令时不但会触发磁盘 I/O，还需要 fork 出一个单独的进程来执行，执行完成后再退出。而执行内建命令相当于调用当前 Shell 进程的一个函数。

下表列出了 Bash Shell 中直接可用的内建命令。

| 命令      | 说明                                                  |
| --------- | ----------------------------------------------------- |
| :         | 扩展参数列表，执行重定向操作                          |
| .         | 读取并执行指定文件中的命令（在当前 shell 环境中）     |
| alias     | 为指定命令定义一个别名                                |
| bg        | 将作业以后台模式运行                                  |
| bind      | 将键盘序列绑定到一个 readline 函数或宏                |
| break     | 退出 for、while、select 或 until 循环                 |
| builtin   | 执行指定的 shell 内建命令                             |
| caller    | 返回活动子函数调用的上下文                            |
| cd        | 将当前目录切换为指定的目录                            |
| command   | 执行指定的命令，无需进行通常的 shell 查找             |
| compgen   | 为指定单词生成可能的补全匹配                          |
| complete  | 显示指定的单词是如何补全的                            |
| compopt   | 修改指定单词的补全选项                                |
| continue  | 继续执行 for、while、select 或 until 循环的下一次迭代 |
| declare   | 声明一个变量或变量类型。                              |
| dirs      | 显示当前存储目录的列表                                |
| disown    | 从进程作业表中刪除指定的作业                          |
| echo      | 将指定字符串输出到 STDOUT                             |
| enable    | 启用或禁用指定的内建shell命令                         |
| eval      | 将指定的参数拼接成一个命令，然后执行该命令            |
| exec      | 用指定命令替换 shell 进程                             |
| exit      | 强制 shell 以指定的退出状态码退出                     |
| export    | 设置子 shell 进程可用的变量                           |
| fc        | 从历史记录中选择命令列表                              |
| fg        | 将作业以前台模式运行                                  |
| getopts   | 分析指定的位置参数                                    |
| hash      | 查找并记住指定命令的全路径名                          |
| help      | 显示帮助文件                                          |
| history   | 显示命令历史记录                                      |
| jobs      | 列出活动作业                                          |
| kill      | 向指定的进程 ID(PID) 发送一个系统信号                 |
| let       | 计算一个数学表达式中的每个参数                        |
| local     | 在函数中创建一个作用域受限的变量                      |
| logout    | 退出登录 shell                                        |
| mapfile   | 从 STDIN 读取数据行，并将其加入索引数组               |
| popd      | 从目录栈中删除记录                                    |
| printf    | 使用格式化字符串显示文本                              |
| pushd     | 向目录栈添加一个目录                                  |
| pwd       | 显示当前工作目录的路径名                              |
| read      | 从 STDIN 读取一行数据并将其赋给一个变量               |
| readarray | 从 STDIN 读取数据行并将其放入索引数组                 |
| readonly  | 从 STDIN 读取一行数据并将其赋给一个不可修改的变量     |
| return    | 强制函数以某个值退出，这个值可以被调用脚本提取        |
| set       | 设置并显示环境变量的值和 shell 属性                   |
| shift     | 将位置参数依次向下降一个位置                          |
| shopt     | 打开/关闭控制 shell 可选行为的变量值                  |
| source    | 读取并执行指定文件中的命令（在当前 shell 环境中）     |
| suspend   | 暂停 Shell 的执行，直到收到一个 SIGCONT 信号          |
| test      | 基于指定条件返回退出状态码 0 或 1                     |
| times     | 显示累计的用户和系统时间                              |
| trap      | 如果收到了指定的系统信号，执行指定的命令              |
| type      | 显示指定的单词如果作为命令将会如何被解释              |
| typeset   | 声明一个变量或变量类型。                              |
| ulimit    | 为系统用户设置指定的资源的上限                        |
| umask     | 为新建的文件和目录设置默认权限                        |
| unalias   | 刪除指定的别名                                        |
| unset     | 刪除指定的环境变量或 shell 属性                       |
| wait      | 等待指定的进程完成，并返回退出状态码                  |

接下来的几节我们将重点讲解几个常用的 Shell 内置命令。

### 2.17Shell alias:给命令创建别名

alisa 用来给命令创建一个别名。若直接输入该命令且不带任何参数，则列出当前 Shell 进程中使用了哪些别名。现在你应该能理解类似`ll`这样的命令为什么与`ls -l`的效果是一样的吧。

下面让我们来看一下有哪些命令被默认创建了别名：

```shell
[mozhiyan@localhost ~]$ alias
alias cp='cp -i'
alias l.='ls -d .* --color=tty'
alias ll='ls -l --color=tty'
alias ls='ls --color=tty'
alias mv='mv -i'
alias rm='rm -i'
alias which='alias | /usr/bin/which --tty-only --read-alias --show-dot --show-tilde'
```

你看，为了让我们使用方便，Shell 会给某些命令默认创建别名。

#### 2.17.1使用 alias 命令自定义别名

使用 alias 命令自定义别名的语法格式为：

```shell
alias new_name='command'
```

比如，一般的关机命令是`shutdown-h now`，写起来比较长，这时可以重新定义一个关机命令，以后就方便多了。

```shell
alias myShutdown='shutdown -h now'
```

再如，通过 date 命令可以获得当前的 UNIX 时间戳，具体写法为`date +%s`，如果你嫌弃它太长或者不容易记住，那可以给它定义一个别名。

```shell
alias timestamp='date +%s'
```

在《Shell命令替换》一节中，我们使用`date +%s`计算脚本的运行时间，现在学了 alias，就可以简化代码了。

```shell
#!/bin/bash
alias timestamp='date +%s'
begin=`timestamp`  
sleep 20s
finish=$(timestamp)
difference=$((finish - begin))
echo "run time: ${difference}s"
运行脚本，20 秒后看到输出结果：
run time: 20s
```

**别名只是临时的**

在代码中使用 alias 命令定义的别名只能在当前 Shell 进程中使用，在子进程和其它进程中都不能使用。当前 Shell 进程结束后，别名也随之消失。

> 要想让别名对所有的 Shell 进程都有效，就得把别名写入 Shell 配置文件。Shell 进程每次启动时都会执行配置文件中的代码做一些初始化工作，将别名放在配置文件中，那么每次启动进程都会定义这个别名。

#### 2.17.2使用 unalias 命令删除别名

使用 unalias 内建命令可以删除当前 Shell 进程中的别名。unalias 有两种使用方法：

- 第一种用法是在命令后跟上某个命令的别名，用于删除指定的别名。
- 第二种用法是在命令后接`-a`参数，删除当前 Shell 进程中所有的别名。

同样，这两种方法都是在当前 Shell 进程中生效的。要想永久删除配置文件中定义的别名，只能进入该文件手动删除。

```shell
# 删除 ll 别名
[mozhiyan@localhost ~]$ unalias ll
# 再次运行该命令时，报“找不到该命令”的错误，说明该别名被删除了
[mozhiyan@localhost ~]$ ll
-bash: ll: command not found
```

### 2.18Shell echo命令：输出字符串

echo 是一个 Shell 内建命令，用来在终端输出字符串，并在最后默认加上换行符。请看下面的例子：

```shell
#!/bin/bash
name="Shell教程"
url="http://c.biancheng.net/shell/"
echo "读者，你好！"  #直接输出字符串
echo $url  #输出变量
echo "${name}的网址是：${url}"  #双引号包围的字符串中可以解析变量
echo '${name}的网址是：${url}'  #单引号包围的字符串中不能解析变量
```

运行结果：

```shell
读者，你好！
http://c.biancheng.net/shell/
Shell教程的网址是：http://c.biancheng.net/shell/
${name}的网址是：${url}
```

#### 2.18.1不换行

echo 命令输出结束后默认会换行，如果不希望换行，可以加上`-n`参数，如下所示：

```shell
#!/bin/bash
name="Tom"
age=20
height=175
weight=62
echo -n "${name} is ${age} years old, "
echo -n "${height}cm in height "
echo "and ${weight}kg in weight."
echo "Thank you!"
```

运行结果：

```shell
Tom is 20 years old, 175cm in height and 62kg in weight.
Thank you!
```

#### 2.18.2输出转义字符

默认情况下，echo 不会解析以反斜杠`\`开头的转义字符。比如，`\n`表示换行，echo 默认会将它作为普通字符对待。请看下面的例子：

```shell
[root@localhost ~]# echo "hello \nworld"
hello \nworld
```

我们可以添加`-e`参数来让 echo 命令解析转义字符。例如：

```shell
[root@localhost ~]# echo -e "hello \nworld"
hello
world
```

**\c 转义字符**

有了`-e`参数，我们也可以使用转义字符`\c`来强制 echo 命令不换行了。请看下面的例子：

```shell
#!/bin/bash
name="Tom"
age=20
height=175
weight=62
echo -e "${name} is ${age} years old, \c"
echo -e "${height}cm in height \c"
echo "and ${weight}kg in weight."
echo "Thank you!"
```

运行结果：

```shell
Tom is 20 years old, 175cm in height and 62kg in weight.
Thank you!
```

### 2.19Shell read命令：读取从键盘输入的数据

read 是 `Shell 内置命令`，用来从`标准输入`中读取数据并赋值给变量。如果没有进行重定向，默认就是从键盘读取用户输入的数据；如果进行了重定向，那么可以从文件中读取数据。

read 命令的用法为：

```shell
read [-options] [variables]
```

`options`表示选项，如下表所示；`variables`表示用来存储数据的变量，可以有一个，也可以有多个。

`options`和`variables`都是可选的，如果没有提供变量名，那么读取的数据将存放到环境变量 `REPLY` 中。

| 选项         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| -a array     | 把读取的数据赋值给数组 array，从下标 0 开始。                |
| -d delimiter | 用字符串 delimiter 指定读取结束的位置，而不是一个换行符（读取到的数据不包括 delimiter）。 |
| -e           | 在获取用户输入的时候，对功能键进行编码转换，不会直接显式功能键对应的字符。 |
| -n num       | 读取 num 个字符，而不是整行字符。                            |
| -p prompt    | 显示提示信息，提示内容为 prompt。                            |
| -r           | 原样读取（Raw mode），不把反斜杠字符解释为转义字符。         |
| -s           | 静默模式（Silent mode），不会在屏幕上显示输入的字符。当输入密码和其它确认信息的时候，这是很有必要的。 |
| -t seconds   | 设置超时时间，单位为秒。如果用户没有在指定时间内输入完成，那么 read 将会返回一个非 0 的退出状态，表示读取失败。 |
| -u fd        | 使用文件描述符 fd 作为输入源，而不是标准输入，类似于重定向。 |

【实例1】使用 read 命令给多个变量赋值。

```shell
#!/bin/bash
read -p "Enter some information > " name url age
echo "网站名字：$name"
echo "网址：$url"
echo "年龄：$age"
```

```shell
运行结果：
Enter some information > C语言中文网 http://c.biancheng.net 7↙
网站名字：C语言中文网
网址：http://c.biancheng.net
年龄：7
```

注意，必须在一行内输入所有的值，不能换行，否则只能给第一个变量赋值，后续变量都会赋值失败。本例还使用了`-p`选项，该选项会用一段文本来提示用户输入。

【示例2】只读取一个字符。

```shell
#!/bin/bash
read -n 1 -p "Enter a char > " char
printf "\n"  #换行
echo $char
```

```shell
运行结果：
Enter a char > 1
1
```

`-n 1`表示只读取一个字符。运行脚本后，只要用户输入一个字符，立即读取结束，不用等待用户按下回车键。

`printf "\n"`语句用来达到换行的效果，否则 echo 的输出结果会和用户输入的内容位于同一行，不容易区分。

【实例3】在指定时间内输入密码。

```shell
#!/bin/bash
if
    read -t 20 -sp "Enter password in 20 seconds(once) > " pass1 && printf "\n" &&  #第一次输入密码
    read -t 20 -sp "Enter password in 20 seconds(again)> " pass2 && printf "\n" &&  #第二次输入密码
    [ $pass1 == $pass2 ]  #判断两次输入的密码是否相等
then
    echo "Valid password"
else
    echo "Invalid password"
fi
```

这段代码中，我们使用`&&`组合了多个命令，这些命令会依次执行，并且从整体上作为 if 语句的判断条件，只要其中一个命令执行失败（退出状态为非 0 值），整个判断条件就失败了，后续的命令也就没有必要执行了。

如果两次输入密码相同，运行结果为：

```shell
Enter password in 20 seconds(once) >
Enter password in 20 seconds(again)>
Valid password
```

如果两次输入密码不同，运行结果为：

```shell
Enter password in 20 seconds(once) >
Enter password in 20 seconds(again)>
Invalid password
```

如果第一次输入超时，运行结果为：

```shell
Enter password in 20 seconds(once) > Invalid password
```

如果第二次输入超时，运行结果为：

```shell
Enter password in 20 seconds(once) >
Enter password in 20 seconds(again)> Invalid password
```

### 2.20Shell exit命令：退出当前进程

exit 是一个 Shell 内置命令，用来退出当前 Shell 进程，并返回一个退出状态；使用`$?`可以接收这个退出状态，这一点已在《Shell $?》中进行了讲解。

exit 命令可以接受一个整数值作为参数，代表退出状态。如果不指定，默认状态值是 0。一般情况下，退出状态为 0 表示成功，退出状态为非 0 表示执行失败（出错）了。

exit 退出状态只能是一个介于 0~255 之间的整数，其中只有 0 表示成功，其它值都表示失败。

`Shell 进程执行出错时，可以根据退出状态来判断具体出现了什么错误`，比如打开一个文件时，我们可以指定 1 表示文件不存在，2 表示文件没有读取权限，3 表示文件类型不对。

编写下面的脚本，并命名为 test.sh：

```shell
#!/bin/bash
echo "befor exit"
exit 8
echo "after exit"
```

运行该脚本：

```shell
[mozhiyan@localhost ~]$ bash ./test.sh
befor exit
```

可以看到，`"after exit"`并没有输出，这说明遇到 exit 命令后，test.sh 执行就结束了。

> 注意，exit 表示退出当前 Shell 进程，我们必须在新进程中运行 test.sh，否则当前 Shell 会话（终端窗口）会被关闭，我们就无法看到输出结果了。

我们可以紧接着使用`$?`来获取 test.sh 的退出状态：

```shell
[mozhiyan@localhost ~]$ echo $?
8
```

### 2.21Shell declare和typeset命令：设置变量属性

declare 和 typeset 都是 Shell 内建命令，它们的用法相同，都用来设置变量的属性。不过 typeset 已经被弃用了，建议使用 declare 代替。

declare 命令的用法如下所示：

```shell
declare [+/-] [aAfFgilprtux] [变量名=变量值]
```

其中，`-`表示设置属性，`+`表示取消属性，`aAfFgilprtux`都是具体的选项，它们的含义如下表所示：

| 选项            | 含义                                                       |
| --------------- | ---------------------------------------------------------- |
| -f [name]       | 列出之前由用户在脚本中定义的函数名称和函数体。             |
| -F [name]       | 仅列出自定义函数名称。                                     |
| -g name         | 在 Shell 函数内部创建全局变量。                            |
| -p [name]       | 显示指定变量的属性和值。                                   |
| -a name         | 声明变量为普通数组。                                       |
| -A name         | 声明变量为关联数组（支持索引下标为字符串）。               |
| -i name         | 将变量定义为整数型。                                       |
| -r name[=value] | 将变量定义为只读（不可修改和删除），等价于 readonly name。 |
| -x name[=value] | 将变量设置为环境变量，等价于 export name[=value]。         |

【实例1】将变量声明为整数并进行计算。

```shell
#!/bin/bash
declare -i m n ret  #将多个变量声明为整数
m=10
n=30
ret=$m+$n
echo $ret
运行结果：
40
```

【实例2】将变量定义为只读变量。

```shell
[c.biancheng.net]$ declare -r n=10
[c.biancheng.net]$ n=20
bash: n: 只读变量
[c.biancheng.net]$ echo $n
10
```

【实例3】显示变量的属性和值。

```shell
[c.biancheng.net]$ declare -r n=10
[c.biancheng.net]$ declare -p n
declare -r n="10"
```

### 2.22Shell数学计算

如果要执行算术运算（数学计算），就离不开各种运算符号，和其他编程语言类似，Shell 也有很多算术运算符，下面就给大家介绍一下常见的 Shell 算术运算符，如下表所示。

| 算术运算符            | 说明/含义                                                |
| --------------------- | -------------------------------------------------------- |
| +、-                  | 加法（或正号）、减法（或负号）                           |
| *、/、%               | 乘法、除法、取余（取模）                                 |
| **                    | 幂运算                                                   |
| ++、--                | 自增和自减，可以放在变量的前面也可以放在变量的后面       |
| !、&&、\|\|           | 逻辑非（取反）、逻辑与（and）、逻辑或（or）              |
| <、<=、>、>=          | 比较符号（小于、小于等于、大于、大于等于）               |
| ==、!=、=             | 比较符号（相等、不相等；对于字符串，= 也可以表示相当于） |
| <<、>>                | 向左移位、向右移位                                       |
| ~、\|、 &、^          | 按位取反、按位或、按位与、按位异或                       |
| =、+=、-=、*=、/=、%= | 赋值运算符，例如 a+=1 相当于 a=a+1，a-=1 相当于 a=a-1    |

但是，Shell 和其它编程语言不同，Shell 不能直接进行算数运算，必须使用数学计算命令，这让初学者感觉很困惑，也让有经验的程序员感觉很奇葩。

下面我们先来看一个反面的例子：

```shell
[c.biancheng.net]$ echo 2+8
2+8
[c.biancheng.net]$ a=23
[c.biancheng.net]$ b=$a+55
[c.biancheng.net]$ echo $b
23+55
[c.biancheng.net]$ b=90
[c.biancheng.net]$ c=$a+$b
[c.biancheng.net]$ echo $c
23+90
```

从上面的运算结果可以看出，默认情况下，Shell 不会直接进行算术运算，而是把`+`两边的数据（数值或者变量）当做字符串，把`+`当做字符串连接符，最终的结果是把两个字符串拼接在一起形成一个新的字符串。

==这是因为，在 Bash Shell 中，如果不特别指明，每一个变量的值都是字符串，无论你给变量赋值时有没有使用引号，值都会以字符串的形式存储。==

换句话说，Bash shell 在默认情况下不会区分变量类型，即使你将整数和小数赋值给变量，它们也会被视为字符串，这一点和大部分的编程语言不同。

#### 2.22.1数学计算命令

要想让数学计算发挥作用，必须使用数学计算命令，Shell 中常用的数学计算命令如下表所示。

| 运算操作符/运算命令 | 说明                                                         |
| ------------------- | ------------------------------------------------------------ |
| (( ))               | 用于整数运算，效率很高，**推荐使用**。                       |
| let                 | 用于整数运算，和 (()) 类似。                                 |
| [$]                 | 用于整数运算，不如 (()) 灵活。                               |
| expr                | 可用于整数运算，也可以处理字符串。比较麻烦，需要注意各种细节，不推荐使用。 |
| bc                  | Linux下的一个计算器程序，可以处理整数和小数。Shell 本身只支持整数运算，想计算小数就得使用 bc 这个外部的计算器。 |
| declare -i          | 将变量定义为整数，然后再进行数学运算时就不会被当做字符串了。功能有限，仅支持最基本的数学运算（加减乘除和取余），不支持逻辑运算、自增自减等，所以在实际开发中很少使用。 |

> 如果大家时间有限，只学习 (()) 和 bc 即可，不用学习其它的了：(()) 可以用于整数计算，bc 可以小数计算。

在接下来的章节中，我们将逐一为大家讲解 Shell 中的各种运算符号及运算命令。

### 2.23Shell (())：对整数进行数学运算

双小括号` (( )) `是 Bash Shell 中专门用来进行整数运算的命令，它的效率很高，写法灵活，是企业运维中常用的运算命令。

> 注意：(( )) 只能进行整数运算，不能对小数（浮点数）或者字符串进行运算。后续讲到的 bc 命令可以用于小数运算。

#### 2.23.1Shell (( )) 的用法

双小括号 (( )) 的语法格式为：

```shell
((表达式))
```

通俗地讲，就是将数学运算表达式放在`((`和`))`之间。

表达式可以只有一个，也可以有多个，多个表达式之间以逗号`,`分隔。对于多个表达式的情况，以最后一个表达式的值作为整个 (( )) 命令的执行结果。

可以使用`$`获取 (( )) 命令的结果，这和使用`$`获得变量值是类似的。

| 运算操作符/运算命令                   | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| ((a=10+66) ((b=a-15)) ((c=a+b))       | 这种写法可以在计算完成后给变量赋值。以 ((b=a-15)) 为例，即将 a-15 的运算结果赋值给变量 c。  注意，使用变量时不用加`$`前缀，(( )) 会自动解析变量名。 |
| a=\$((10+66) b=\$((a-15)) c=\$((a+b)) | 可以在 (( )) 前面加上`$`符号获取 (( )) 命令的执行结果，也即获取整个表达式的值。以 c=$((a+b)) 为例，即将 a+b 这个表达式的运算结果赋值给变量 c。  注意，类似 c=((a+b)) 这样的写法是错误的，不加`$`就不能取得表达式的结果。 |
| ((a>7 && b==c))                       | (( )) 也可以进行逻辑运算，在 if 语句中常会使用逻辑运算。     |
| echo $((a+10))                        | 需要立即输出表达式的运算结果时，可以在 (( )) 前面加`$`符号。 |
| ((a=3+5, b=a+10))                     | 对多个表达式同时进行计算。                                   |

==在 (( )) 中使用变量无需加上`$`前缀，(( )) 会自动解析变量名，这使得代码更加简洁，也符合程序员的书写习惯。==

#### 2.23.2Shell (( )) 实例演示

【实例1】利用 (( )) 进行简单的数值计算。

```shell
[c.biancheng.net]$ echo $((1+1))
2
[c.biancheng.net]$ echo $((6-3))
3
[c.biancheng.net]$ i=5
[c.biancheng.net]$ ((i=i*2))  #可以简写为 ((i*=2))。
[c.biancheng.net]$ echo $i   #使用 echo 输出变量结果时要加 $。
10
```

【实例2】用 (( )) 进行稍微复杂一些的综合算术运算。

```shell
[c.biancheng.net]$ ((a=1+2**3-4%3))
[c.biancheng.net]$ echo $a
8
[c.biancheng.net]$ b=$((1+2**3-4%3)) #运算后将结果赋值给变量，变量放在了括号的外面。
[c.biancheng.net]$ echo $b
8
[c.biancheng.net]$ echo $((1+2**3-4%3)) #也可以直接将表达式的结果输出，注意不要丢掉 $ 符号。
8
[c.biancheng.net]$ a=$((100*(100+1)/2)) #利用公式计算1+2+3+...+100的和。
[c.biancheng.net]$ echo $a
5050
[c.biancheng.net]$ echo $((100*(100+1)/2)) #也可以直接输出表达式的结果。
5050
```

【实例3】利用 (( )) 进行逻辑运算。

```shell
[c.biancheng.net]$ echo $((3<8))  #3<8 的结果是成立的，因此，输出了 1，1 表示真
1
[c.biancheng.net]$ echo $((8<3))  #8<3 的结果是不成立的，因此，输出了 0，0 表示假。
0
[c.biancheng.net]$ echo $((8==8)) #判断是否相等。
1
[c.biancheng.net]$ if ((8>7&&5==5))
> then
> echo yes
> fi
yes
```

最后是一个简单的 if 语句的格式，它的意思是，如果 8>7 成立，并且 5==5 成立，那么输出 yes。显然，这两个条件都是成立的，所以输出了 yes。

实例4】利用 (( )) 进行自增（++）和自减（--）运算

```shell
[c.biancheng.net]$ a=10
[c.biancheng.net]$ echo $((a++))  #如果++在a的后面，那么在输出整个表达式时，会输出a的值,因为a为10，所以表达式的值为10。
10
[c.biancheng.net]$ echo $a #执行上面的表达式后，因为有a++，因此a会自增1，因此输出a的值为11。
11
[c.biancheng.net]$ a=11
[c.biancheng.net]$ echo $((a--)) #如果--在a的后面，那么在输出整个表达式时，会输出a的值，因为a为11，所以表达式的值的为11。
11
[c.biancheng.net]$ echo $a #执行上面的表达式后，因为有a--，因此a会自动减1，因此a为10。
10
[c.biancheng.net]$ a=10
[c.biancheng.net]$ echo $((--a))  #如果--在a的前面，那么在输出整个表达式时，先进行自增或自减计算，因为a为10，且要自减，所以表达式的值为9。
9
[c.biancheng.net]$ echo $a #执行上面的表达式后，a自减1,因此a为9。
9
[c.biancheng.net]$ echo $((++a))  #如果++在a的前面，输出整个表达式时，先进行自增或自减计算，因为a为9，且要自增1，所以输出10。
10
[c.biancheng.net]$ echo $a  #执行上面的表达式后，a自增1,因此a为10。
10
```

【实例5】利用 (( )) 同时对多个表达式进行计算。

```shell
[c.biancheng.net]$ ((a=3+5, b=a+10))  #先计算第一个表达式，再计算第二个表达式
[c.biancheng.net]$ echo $a $b
8 18
[c.biancheng.net]$ c=$((4+8, a+b))  #以最后一个表达式的结果作为整个(())命令的执行结果
[c.biancheng.net]$ echo $c
26
```

### 2.24Shell let命令：对整数进行数学运算

==注意：和双小括号 (( )) 一样，let 命令也只能进行整数运算，不能对小数（浮点数）或者字符串进行运算。==

Shell let 命令的语法格式为：

```shell
let 表达式
```

或者

```shell
let "表达式"
```

或者

```shell
let '表达式'
```

它们都等价于`((表达式))`。

当表达式中含有 Shell 特殊字符（例如 |）时，需要用双引号`" "`或者单引号`' '`将表达式包围起来。

和 (( )) 类似，let 命令也支持一次性计算多个表达式，并且以最后一个表达式的值作为整个 let 命令的执行结果。但是，对于多个表达式之间的分隔符，let 和 (( )) 是有区别的：

- let 命令以`空格`来分隔多个表达式；
- (( )) 以逗号`,`来分隔多个表达式。

另外还要注意，对于类似`let x+y`这样的写法，Shell 虽然计算了 x+y 的值，但却将结果丢弃；若不想这样，可以使用`let sum=x+y`将 x+y 的结果保存在变量 sum 中。

这种情况下 (( )) 显然更加灵活，可以使用`$((x+y))`来获取 x+y 的结果。请看下面的例子：

```shell
[c.biancheng.net]$ a=10 b=20
[c.biancheng.net]$ echo $((a+b))
30
[c.biancheng.net]$ echo let a+b  #错误，echo会把 let a+b作为一个字符串输出
let a+b
```

#### 2.24.1Shell let 命令实例演示

【实例1】给变量 i 加 8：

```shell
[c.biancheng.net]$ i=2
[c.biancheng.net]$ let i+=8
[c.biancheng.net]$ echo $i
10
```

let i+=8 等同于 ((i+=8))，但后者效率更高。

【实例2】let 后面可以跟多个表达式。

```shell
[c.biancheng.net]$ a=10 b=35
[c.biancheng.net]$ let a+=6 c=a+b  #多个表达式以空格为分隔
[c.biancheng.net]$ echo $a $c
16 51
```

### 2.25Linux bc命令：一款数学计算器

Bash Shell 内置了对整数运算的支持，但是并不支持浮点运算，而 Linux bc 命令可以很方便的进行浮点运算，当然整数运算也不再话下。

bc 甚至可以称得上是一种编程语言了，它支持变量、数组、输入输出、分支结构、循环结构、函数等基本的编程元素，所以 Linux 手册中是这样来描述 bc 的：

```shell
An arbitrary precision calculator language
```

翻译过来就是“一个任意精度的计算器语言”。

在终端输入 bc 命令，然后回车即可进入 bc 进行交互式的数学计算。在 Shell 编程中，我们也可以通过管道和输入重定向来使用 bc。本节我们先学习如何在交互式环境下使用 bc，然后再学习如何在 Shell 编程中使用 bc，这样就易如反掌了。

#### 2.25.1从终端进入 bc

在终端输入 bc 命令，然后回车，就可以进入 bc，请看下图：

![image-20230102143711013](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102143711013.png)

bc 命令还有一些选项，可能你会用到，请看下表。

![image-20230102143750823](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102143750823.png)

例如你不想输入 bc 命令后显示一堆没用的信息，那么可以输入 `bc -q`：

![image-20230102143811876](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102143811876.png)

#### 2.25.2在交互式环境下使用 bc

使用 bc 进行数学计算是非常容易的，像平常一样输入数学表达式，然后按下回车键就可以看到结果，请看下图。

![image-20230102143846829](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102143846829.png)

值得一提的是，我们定义了一个变量 n，然后在计算中也使用了 n，可见 bc 是支持变量的。除了变量，bc 还支持函数、循环结构、分支结构等常见的编程元素，它们和其它编程语言的语法类似。下面我们定义一个求阶乘的函数：

![image-20230102143915406](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102143915406.png)

其实我们很少使用这么复杂的功能，大部分情况下还是把 bc 作为普通的数学计算器，求一下表达式的值而已，所以大家不必深究，了解一下即可。

**内置变量**

bc 有四个内置变量，我们在计算时会经常用到，如下表所示：

![image-20230102144001111](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102144001111.png)

【实例 1】scale 变量用法举例：

![image-20230102144031987](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102144031987.png)

刚开始的时候，10/3 的值为 3，不带小数部分，就是因为 scale 变量的默认值为 0；后边给 scale 指定了一个大于 0 的值，就能看到小数部分了。

【实例 2】ibase 和 obase 变量用法举例：

![image-20230102144122743](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102144122743.png)

> 注意：obase 要尽量放在 ibase 前面，因为 ibase 设置后，后面的数字都是以 ibase 的进制来换算的。

**内置函数**

除了内置变量，bc 还有一些内置函数，如下表所示：

![image-20230102144223637](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102144223637.png)

要想使用这些数学函数，在输入 bc 命令时需要使用-l 选项，表示启用数学库。请看下面的例子：

![image-20230102144239845](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102144239845.png)

**在一行中使用多个表达式**

在前边的例子中，我们基本上是一行一个表达式，这样看起来更加舒服；如果你愿意，也可以将多个表达式放在一行，只要用分号;隔开就行。请看下面的例子：

![image-20230102144310977](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102144310977.png)

#### 2.25.3在 Shell 中使用 bc 计算器

在 Shell 脚本中，我们可以借助管道或者输入重定向来使用 bc 计算器。

- 管道是 Linux 进程间的一种通信机制，它可以将前一个命令（进程）的输出作为下一个命令（进程）的输入，两个命令之间使用竖线`|`分隔。

- 通常情况下，一个命令从终端获得用户输入的内容，如果让它从其他地方比如文件获得输入，那么就需要重定向。

**借助管道使用 bc 计算器**

如果读者希望直接输出 bc 的计算结果，那么可以使用下面的形式：

```shell
echo "expression" | bc
```

expression 就是希望计算的数学表达式，它必须符合 bc 的语法，上面我们已经进行了介绍。在 expression 中，还可以使用 Shell 脚本中的变量。

使用下面的形式可以将 bc 的计算结果赋值给 Shell 变量：

```shell
variable=$(echo "expression" | bc)
```

【实例 1】最简单的形式：

```shell
[c.biancheng.net]$ echo "3*8"|bc
24
[c.biancheng.net]$ ret=$(echo "4+9"|bc)
[c.biancheng.net]$ echo $ret
13
```

【实例 2】使用 bc 中的变量：

```shell
[c.biancheng.net]$ echo "scale=4;3*8/7"|bc
3.4285
[c.biancheng.net]$ echo "scale=4;3*8/7;last*5"|bc
3.4285
17.1425
```

【实例 3】使用 Shell 脚本中的变量：

```shell
[c.biancheng.net]$ x=4
[c.biancheng.net]$ echo "scale=5;n=$x+2;e(n)" | bc -l
403.42879
```

在第二条命令中，`$x` 表示使用第一条 Shell 命令中定义的变量，`n` 是在 bc 中定义的新变量，它和 Shell 脚本是没关系的。

【实例 4】进制转换：

```shell
#十进制转十六进制
[mozhiyan@localhost ~]$ m=31
[mozhiyan@localhost ~]$ n=$(echo "obase=16;$m"|bc)
[mozhiyan@localhost ~]$ echo $n
1F
#十六进制转十进制
[mozhiyan@localhost ~]$ m=1E
[mozhiyan@localhost ~]$ n=$(echo "obase=10;ibase=16;$m"|bc)
[mozhiyan@localhost ~]$ echo $n
30
```

**借助输入重定向使用 bc 计算器**

可以使用下面的形式将 bc 的计算结果赋值给 Shell 变量：

```shell
variable=$(bc << EOF
expressions
EOF
)
```

其中，`variable` 是 Shell 变量名，`express` 是要计算的数学表达式（可以换行，和进入 bc 以后的书写形式一样），`EOF` 是数学表达式的开始和结束标识（你也可以换成其它的名字，比如 aaa、bbb 等）。

请看下面的例子：

```shell
[c.biancheng.net]$ m=1E
[c.biancheng.net]$ n=$(bc << EOF
> obase=10;
> ibase=16;
> print $m
> EOF
> )
[c.biancheng.net]$ echo $n
30
```

**如果你有大量的数学计算，那么使用输入重定向就比较方便，因为数学表达式可以换行，写起来更加清晰明了。**

### 2.26Shell if else语句

和其它编程语言类似，Shell 也支持选择结构，并且有两种形式，分别是 if else 语句和 case in 语句。本节我们先介绍 if else 语句，case in 语句将会在《Shell case in》中介绍。如果你已经熟悉了C语言、Java、JavaScript 等其它编程语言，那么你可能会觉得 Shell 中的 if else 语句有点奇怪。

#### 2.26.1if 语句

最简单的用法就是只使用 if 语句，它的语法格式为：

```shell
if  condition
then
    statement(s)
fi
```

`condition`是判断条件，如果 condition 成立（返回“真”），那么 then 后边的语句将会被执行；如果 condition 不成立（返回“假”），那么不会执行任何语句。

> 从本质上讲，if 检测的是命令的退出状态，我们将在下节《Shell退出状态》中深入讲解。

注意，最后必须以`fi`来闭合，fi 就是 if 倒过来拼写。也正是有了 fi 来结尾，所以即使有多条语句也不需要用`{ }`包围起来。

如果你喜欢，也可以将 then 和 if 写在一行：

```shell
if  condition;  then
    statement(s)
fi
```

请注意 condition 后边的分号`;`，当 if 和 then 位于同一行的时候，这个分号是必须的，否则会有语法错误。

**实例1**

下面的例子使用 if 语句来比较两个数字的大小：

```shell
#!/bin/bash
read a
read b
if (( $a == $b ))
then
    echo "a和b相等"
fi
运行结果：
84↙
84↙
a和b相等
```

在《Shell (())》一节中我们讲到，(())是一种数学计算命令，它除了可以进行最基本的加减乘除运算，还可以进行大于、小于、等于等关系运算，以及与、或、非逻辑运算。当 a 和 b 相等时，(( \$a == \$b ))判断条件成立，进入 if，执行 then 后边的 echo 语句。

**实例2**

在判断条件中也可以使用逻辑运算符，例如：

```shell
#!/bin/bash
read age
read iq
if (( $age > 18 && $iq < 60 ))
then
    echo "你都成年了，智商怎么还不及格！"
    echo "来C语言中文网（http://c.biancheng.net/）学习编程吧，能迅速提高你的智商。"
fi
运行结果：
20↙
56↙
你都成年了，智商怎么还不及格！
来C语言中文网（http://c.biancheng.net/）学习编程吧，能迅速提高你的智商。
```

`&&`就是逻辑“与”运算符，只有当`&&`两侧的判断条件都为“真”时，整个判断条件才为“真”。

熟悉其他编程语言的读者请注意，即使 then 后边有多条语句，也不需要用`{ }`包围起来，因为有 fi 收尾呢。

#### 2.26.2if else 语句

如果有两个分支，就可以使用 if else 语句，它的格式为：

```shell
if  condition
then
   statement1
else
   statement2
fi
```

如果 condition 成立，那么 then 后边的 statement1 语句将会被执行；否则，执行 else 后边的 statement2 语句。

举个例子：

```shell
#!/bin/bash
read a
read b
if (( $a == $b ))
then
    echo "a和b相等"
else
    echo "a和b不相等，输入错误"
fi
运行结果：
10↙
20↙
a 和 b 不相等，输入错误
```

从运行结果可以看出，a 和 b 不相等，判断条件不成立，所以执行了 else 后边的语句。

#### 2.26.3if elif else 语句

Shell 支持任意数目的分支，当分支比较多时，可以使用 if elif else 结构，它的格式为：

```shell
if  condition1
then
   statement1
elif condition2
then
    statement2
elif condition3
then
    statement3
……
else
   statementn
fi
```

注意，if 和 elif 后边都得跟着 then。

举个例子，输入年龄，输出对应的人生阶段：

```shell
#!/bin/bash
read age
if (( $age <= 2 )); then
    echo "婴儿"
elif (( $age >= 3 && $age <= 8 )); then
    echo "幼儿"
elif (( $age >= 9 && $age <= 17 )); then
    echo "少年"
elif (( $age >= 18 && $age <=25 )); then
    echo "成年"
elif (( $age >= 26 && $age <= 40 )); then
    echo "青年"
elif (( $age >= 41 && $age <= 60 )); then
    echo "中年"
else
    echo "老年"
fi

运行结果1：
19
成年

运行结果2：
100
老年
```

再举一个例子，输入一个整数，输出该整数对应的星期几的英文表示：

```shell
#!/bin/bash
printf "Input integer number: "
read num
if ((num==1)); then
    echo "Monday"
elif ((num==2)); then
    echo "Tuesday"
elif ((num==3)); then
    echo "Wednesday"
elif ((num==4)); then
    echo "Thursday"
elif ((num==5)); then
    echo "Friday"
elif ((num==6)); then
    echo "Saturday"
elif ((num==7)); then
    echo "Sunday"
else
    echo "error"
fi

运行结果1：
Input integer number: 4
Thursday

运行结果2：
Input integer number: 9
error
```

### 2.27Shell退出状态

每一条 Shell 命令，不管是 Bash 内置命令（例如 cd、echo），还是外部的 Linux 命令（例如 ls、awk），还是自定义的 Shell 函数，当它退出（运行结束）时，都会返回一个比较小的整数值给调用（使用）它的程序，这就是命令的`退出状态（exit statu）`。

> 很多 Linux 命令其实就是一个C语言程序，熟悉C语言的读者都知道，main() 函数的最后都有一个`return 0`，如果程序想在中间退出，还可以使用`exit 0`，这其实就是C语言程序的退出状态。当有其它程序调用这个程序时，就可以捕获这个退出状态。

if 语句的判断条件，从本质上讲，判断的就是命令的退出状态。

==按照惯例来说，退出状态为 0 表示“成功”；也就是说，程序执行完成并且没有遇到任何问题。除 0 以外的其它任何退出状态都为“失败”。==

之所以说这是“惯例”而非“规定”，是因为也会有例外，比如 diff 命令用来比较两个文件的不同，对于“没有差别”的文件返回 0，对于“找到差别”的文件返回 1，对无效文件名返回 2。

> 有编程经验的读者请注意，Shell 的这个部分与你所熟悉的其它编程语言正好相反：在C语言、C++、Java、Python 中，0 表示“假”，其它值表示“真”。

在 Shell 中，有多种方式取得命令的退出状态，其中` $? `是最常见的一种。上节《Shell if else》中使用了` (()) `进行数学计算，我们不妨来看一下它的退出状态。请看下面的代码：

```shell
#!/bin/bash
read a
read b
(( $a == $b ));
echo "退出状态："$?

运行结果1：
26
26
退出状态：0

运行结果2：
17
39
退出状态：1
```

#### 2.27.1退出状态和逻辑运算符的组合

Shell if 语句的一个神奇之处是允许我们使用逻辑运算符将多个退出状态组合起来，这样就可以一次判断多个条件了。

| 运算符 | 使用格式                     | 说明                                                         |
| ------ | ---------------------------- | ------------------------------------------------------------ |
| &&     | expression1 && expression2   | 逻辑与运算符，当 expression1 和 expression2 同时成立时，整个表达式才成立。  如果检测到 expression1 的退出状态为 1，就不会再检测 expression2 了，因为不管 expression2 的退出状态是什么，整个表达式必然都是不成立的，检测了也是多此一举。 |
| \|\|   | expression1 \|\| expression2 | 逻辑或运算符，expression1 和 expression2 两个表达式中只要有一个成立，整个表达式就成立。  如果检测到 expression1 的退出状态为 0，就不会再检测 expression2 了，因为不管 expression2 的退出状态是什么，整个表达式必然都是成立的，检测了也是多此一举。 |
| !      | !expression                  | 逻辑非运算符，相当于“取反”的效果。如果 expression 成立，那么整个表达式就不成立；如果 expression 不成立，那么整个表达式就成立。 |

【实例】将用户输入的 URL 写入到文件中。

```shell
#!/bin/bash
read filename
read url
if test -w $filename && test -n $url
then
    echo $url > $filename
    echo "写入成功"
else
    echo "写入失败"
fi
```

在 Shell 脚本文件所在的目录新建一个文本文件并命名为 urls.txt，然后运行 Shell 脚本，运行结果为：

```shell
urls.txt↙
http://c.biancheng.net/shell/↙
写入成功
```

test 是 Shell 内置命令，可以对文件或者字符串进行检测，其中，`-w`选项用来检测文件是否存在并且可写，`-n`选项用来检测字符串是否非空。

`>`表示重定向，默认情况下，echo 向控制台输出，这里我们将输出结果重定向到文件。

### 2.28Shell test命令

test 是 Shell 内置命令，用来检测某个条件是否成立。test 通常和 if 语句一起使用，并且大部分 if 语句都依赖 test。

test 命令有很多选项，可以进行`数值`、`字符串`和`文件`三个方面的检测。

Shell test 命令的用法为：

```shell
test expression
```

当 test 判断 expression 成立时，退出状态为 0，否则为非 0 值。

test 命令也可以简写为`[]`，它的用法为：

```shell
[ expression ]
```

注意`[]`和`expression`之间的空格，这两个空格是必须的，否则会导致语法错误。`[]`的写法更加简洁，比 test 使用频率高。

> test 和 [] 是等价的，后续我们会交替使用 test 和 []，以让读者尽快熟悉。

在《Shell if else》中，我们使用 (()) 进行数值比较，这节我们就来看一下如何使用 test 命令进行数值比较。

```shell
#!/bin/bash
read age
if test $age -le 2; then
    echo "婴儿"
elif test $age -ge 3 && test $age -le 8; then
    echo "幼儿"
elif [ $age -ge 9 ] && [ $age -le 17 ]; then
    echo "少年"
elif [ $age -ge 18 ] && [ $age -le 25 ]; then
    echo "成年"
elif test $age -ge 26 && test $age -le 40; then
    echo "青年"
elif test $age -ge 41 && [ $age -le 60 ]; then
    echo "中年"
else
    echo "老年"
fi
```

其中，`-le`选项表示小于等于，`-ge`选项表示大于等于，`&&`是逻辑与运算符。

学习 test 命令，重点是学习它的各种选项，下面我们就逐一讲解。

#### 2.28.1与文件检测相关的 test 选项

| 文件类型判断            |                                                              |
| ----------------------- | ------------------------------------------------------------ |
| 选 项                   | 作 用                                                        |
| -b filename             | 判断文件是否存在，并且是否为块设备文件。                     |
| -c filename             | 判断文件是否存在，并且是否为字符设备文件。                   |
| -d filename             | 判断文件是否存在，并且是否为目录文件。                       |
| -e filename             | 判断文件是否存在。                                           |
| -f filename             | 判断文件是否存在，井且是否为普通文件。                       |
| -L filename             | 判断文件是否存在，并且是否为符号链接文件。                   |
| -p filename             | 判断文件是否存在，并且是否为管道文件。                       |
| -s filename             | 判断文件是否存在，并且是否为非空。                           |
| -S filename             | 判断该文件是否存在，并且是否为套接字文件。                   |
| 文件权限判断            |                                                              |
| 选 项                   | 作 用                                                        |
| -r filename             | 判断文件是否存在，并且是否拥有读权限。                       |
| -w filename             | 判断文件是否存在，并且是否拥有写权限。                       |
| -x filename             | 判断文件是否存在，并且是否拥有执行权限。                     |
| -u filename             | 判断文件是否存在，并且是否拥有 SUID 权限。                   |
| -g filename             | 判断文件是否存在，并且是否拥有 SGID 权限。                   |
| -k filename             | 判断该文件是否存在，并且是否拥有 SBIT 权限。                 |
| 文件比较                |                                                              |
| 选 项                   | 作 用                                                        |
| filename1 -nt filename2 | 判断 filename1 的修改时间是否比 filename2 的新。             |
| filename -ot filename2  | 判断 filename1 的修改时间是否比 filename2 的旧。             |
| filename1 -ef filename2 | 判断 filename1 是否和 filename2 的 inode 号一致，可以理解为两个文件是否为同一个文件。这个判断用于判断硬链接是很好的方法 |

Shell test 文件检测举例：

```shell
#!/bin/bash
read filename
read url
if test -w $filename && test -n $url
then
    echo $url > $filename
    echo "写入成功"
else
    echo "写入失败"
fi
```

在 Shell 脚本文件所在的目录新建一个文本文件并命名为 urls.txt，然后运行 Shell 脚本，运行结果为：

```shell
urls.txt↙
http://c.biancheng.net/shell/↙
写入成功
```

#### 2.28.2与数值比较相关的 test 选项

| 选 项         | 作 用                          |
| ------------- | ------------------------------ |
| num1 -eq num2 | 判断 num1 是否和 num2 相等。   |
| num1 -ne num2 | 判断 num1 是否和 num2 不相等。 |
| num1 -gt num2 | 判断 num1 是否大于 num2 。     |
| num1 -lt num2 | 判断 num1 是否小于 num2。      |
| num1 -ge num2 | 判断 num1 是否大于等于 num2。  |
| num1 -le num2 | 判断 num1 是否小于等于 num2。  |

注意，test 只能用来比较整数，小数相关的比较还得依赖 bc 命令。

Shell test 数值比较举例：

```shell
#!/bin/bash
read a b
if test $a -eq $b
then
    echo "两个数相等"
else
    echo "两个数不相等"
fi

运行结果1：
10 10
两个数相等

运行结果2：
10 20
两个数不相等
```

#### 2.28.3与字符串判断相关的 test 选项

| 选 项                    | 作 用                                                        |
| ------------------------ | ------------------------------------------------------------ |
| -z str                   | 判断字符串 str 是否为空。                                    |
| -n str                   | 判断宇符串 str 是否为非空。                                  |
| str1 = str2 str1 == str2 | `=`和`==`是等价的，都用来判断 str1 是否和 str2 相等。        |
| str1 != str2             | 判断 str1 是否和 str2 不相等。                               |
| str1 \\> str2            | 判断 str1 是否大于 str2。`\>`是`>`的转义字符，这样写是为了防止`>`被误认为成重定向运算符。 |
| str1 \\< str2            | 判断 str1 是否小于 str2。同样，`\<`也是转义字符。            |

> 有C语言、C++、Python、Java 等编程经验的读者请注意，==、>、< 在大部分编程语言中都用来比较数字，而在 Shell 中，它们只能用来比较字符串，不能比较数字，这是非常奇葩的，大家要习惯。其次，不管是比较数字还是字符串，Shell 都不支持 >= 和 <= 运算符，切记。

Shell test 字符串比较举例：

```shell
#!/bin/bash
read str1
read str2
#检测字符串是否为空
if [ -z "$str1" ] || [ -z "$str2" ]
then
    echo "字符串不能为空"
    exit 0
fi
#比较字符串
if [ $str1 = $str2 ]
then
    echo "两个字符串相等"
else
    echo "两个字符串不相等"
fi
运行结果：
http://c.biancheng.net/
http://c.biancheng.net/shell/
两个字符串不相等
```

细心的读者可能已经注意到，变量 \$str1 和 \$str2 都被双引号包围起来，这样做是为了防止 \$str1 或者 \$str2 是空字符串时出现错误，本文的后续部分将为你分析具体原因。

#### 2.28.4与逻辑运算相关的 test 选项

| 选 项                      | 作 用                                                        |
| -------------------------- | ------------------------------------------------------------ |
| expression1 -a expression  | 逻辑与，表达式 expression1 和 expression2 都成立，最终的结果才是成立的。 |
| expression1 -o expression2 | 逻辑或，表达式 expression1 和 expression2 有一个成立，最终的结果就成立。 |
| !expression                | 逻辑非，对 expression 进行取反。                             |

改写上面的代码，使用逻辑运算选项：

```shell
#!/bin/bash
read str1
read str2
#检测字符串是否为空
if [ -z "$str1" -o -z "$str2" ]  #使用 -o 选项取代之前的 ||
then
    echo "字符串不能为空"
    exit 0
fi
#比较字符串
if [ $str1 = $str2 ]
then
    echo "两个字符串相等"
else
    echo "两个字符串不相等"
fi
```

前面的代码我们使用两个`[]`命令，并使用`||`运算符将它们连接起来，这里我们改成`-o`选项，只使用一个`[]`命令就可以了。

**在 test 中使用变量建议用双引号包围起来**

test 和 [] 都是命令，一个命令本质上对应一个程序或者一个函数。即使是一个程序，它也有入口函数，例如C语言程序的入口函数是 main()，运行C语言程序就从 main() 函数开始，所以也可以将一个程序等效为一个函数，这样我们就不用再区分函数和程序了，直接将一个命令和一个函数对应起来即可。

有了以上认知，就很容易看透命令的本质了：使用一个命令其实就是调用一个函数，命令后面附带的选项和参数最终都会作为实参传递给函数。

假设 test 命令对应的函数是 func()，使用`test -z $str1`命令时，会先将变量 \$str1 替换成字符串：

- 如果 $str1 是一个正常的字符串，比如 abc123，那么替换后的效果就是`test -z abc123`，调用 func() 函数的形式就是`func("-z abc123")`。test 命令后面附带的所有选项和参数会被看成一个整体，并作为实参传递进函数。
- 如果 $str1 是一个空字符串，那么替换后的效果就是`test -z`，调用 func() 函数的形式就是`func("-z ")`，这就比较奇怪了，因为`-z`选项没有和参数成对出现，func() 在分析时就会出错。

如果我们给 \$str1 变量加上双引号，当 \$str1 是空字符串时，`test -z "$str1"`就会被替换为`test -z ""`，调用 func() 函数的形式就是`func("-z \"\"")`，很显然，`-z`选项后面跟的是一个空字符串（`\"`表示转义字符），这样 func() 在分析时就不会出错了。

> 所以，当你在 test 命令中使用变量时，我强烈建议将变量用双引号`""`包围起来，这样能避免变量为空值时导致的很多奇葩问题。

#### 2.28.5总结

test 命令比较奇葩，>、<、== 只能用来比较字符串，不能用来比较数字，比较数字需要使用 -eq、-gt 等选项；不管是比较字符串还是数字，test 都不支持 >= 和 <=。有经验的程序员需要慢慢习惯 test 命令的这些奇葩用法。

对于整型数字的比较，我建议大家使用` (())`，这在《Shell if else》中已经进行了演示。(()) 支持各种运算符，写法也符合数学规则，用起来更加方便，何乐而不为呢？

几乎完全兼容 test ，并且比 test 更加强大，比 test 更加灵活的是`[[ ]]`；`[[ ]]`不是命令，而是 Shell 关键字。

### 2.29Shell [[]]详解：检测某个条件是否成立

`[[ ]]`是 Shell 内置关键字，它和 test 命令类似，也用来检测某个条件是否成立。test 能做到的，[[ ]] 也能做到，而且 [[ ]] 做的更好；test 做不到的，[[ ]] 还能做到。可以认为 [[ ]] 是 test 的升级版，对细节进行了优化，并且扩展了一些功能。

[[ ]] 的用法为：

```shell
[[ expression ]]
```

当 [[ ]] 判断 expression 成立时，退出状态为 0，否则为非 0 值。注意`[[ ]]`和`expression`之间的空格，这两个空格是必须的，否则会导致语法错误。

#### 2.29.1[[ ]] 不需要注意某些细枝末节

[[ ]] 是 Shell 内置关键字，不是命令，在使用时没有给函数传递参数的过程，所以 test 命令的某些注意事项在 [[ ]] 中就不存在了，具体包括：

- 不需要把变量名用双引号`""`包围起来，即使变量是空值，也不会出错。
- 不需要、也不能对 >、< 进行转义，转义后会出错。

请看下面的演示代码：

```shell
#!/bin/bash
read str1
read str2
if [[ -z $str1 ]] || [[ -z $str2 ]]  #不需要对变量名加双引号
then
    echo "字符串不能为空"
elif [[ $str1 < $str2 ]]  #不需要也不能对 < 进行转义
then
    echo "str1 < str2"
else
    echo "str1 >= str2"
fi

运行结果：
http://c.biancheng.net/shell/
http://data.biancheng.net/
str1 < str2
```

#### 2.29.2[[ ]] 支持逻辑运算符

对多个表达式进行逻辑运算时，可以使用逻辑运算符将多个 test 命令连接起来，例如：

```shell
[ -z "$str1" ] || [ -z "$str2" ]
```

你也可以借助选项把多个表达式写在一个 test 命令中，例如：

```shell
[ -z "$str1" -o -z "$str2" ]
```

但是，这两种写法都有点“别扭”，完美的写法是在一个命令中使用逻辑运算符将多个表达式连接起来。我们的这个愿望在 [[ ]] 中实现了，[[ ]] 支持 &&、|| 和 ! 三种逻辑运算符。

使用 [[ ]] 对上面的语句进行改进：

```shell
[[ -z $str1 || -z $str2 ]]
```

这种写法就比较简洁漂亮了。

注意，[[ ]] 剔除了 test 命令的`-o`和`-a`选项，你只能使用 `||` 和 `&&`。这意味着，你不能写成下面的形式：

```shell
[[ -z $str1 -o -z $str2 ]]
```

当然，使用逻辑运算符将多个 [[ ]] 连接起来依然是可以的，因为这是 Shell 本身提供的功能，跟 [[ ]] 或者 test 没有关系，如下所示：

```shell
[[ -z $str1 ]] || [[ -z $str2 ]]
```

| test 或 []                           |       | [[ ]]                                |       |
| ------------------------------------ | ----- | ------------------------------------ | ----- |
| [ -z "\$str1" ] \|\| [ -z "\$str2" ] | **√** | [[ -z \$str1 ]] \|\| [[ -z \$str2 ]] | **√** |
| [ -z "\$str1" -o -z "\$str2" ]       | **√** | [[ -z \$str1 -o -z \$str2 ]]         | **×** |
| [ -z \$str1 \|\| -z \$str2 ]         | **×** | [[ -z \$str1 \|\| -z \$str2 ]]       | **√** |

#### 2.29.3[[ ]] 支持正则表达式

在 Shell [[ ]] 中，可以使用`=~`来检测字符串是否符合某个正则表达式，它的用法为：

```shell
[[ str =~ regex ]]
```

str 表示字符串，regex 表示正则表达式。

下面的代码检测一个字符串是否是手机号：

```shell
#!/bin/bash
read tel
if [[ $tel =~ ^1[0-9]{10}$ ]]
then
    echo "你输入的是手机号码"
else
    echo "你输入的不是手机号码"
fi

运行结果1：
13203451100
你输入的是手机号码

运行结果2：
132034511009
你输入的不是手机号码
```

对`^1[0-9]{10}$`的说明：

- `^`匹配字符串的开头（一个位置）；
- `[0-9]{10}`匹配连续的十个数字；
- `$`匹配字符串的末尾（一个位置）。

#### 2.29.4总结

有了 [[ ]]，你还有什么理由使用 test 或者 [ ]，[[ ]] 完全可以替代之，而且更加方便，更加强大。

`但是 [[ ]] 对数字的比较仍然不友好，所以我建议，以后大家使用 if 判断条件时，用 (()) 来处理整型数字，用 [[ ]] 来处理字符串或者文件。`

### 2.30Shell case in语句

和其它编程语言类似，Shell 也支持两种分支结构（选择结构），分别是 if else 语句和 case in 语句。在《Shell if else》一节中我们讲解了 if else 语句的用法，这节我们就来讲解 case in 语句。

当分支较多，并且判断条件比较简单时，使用 case in 语句就比较方便了。

```shell
#!/bin/bash
printf "Input integer number: "
read num
case $num in
    1)
        echo "Monday"
        ;;
    2)
        echo "Tuesday"
        ;;
    3)
        echo "Wednesday"
        ;;
    4)
        echo "Thursday"
        ;;
    5)
        echo "Friday"
        ;;
    6)
        echo "Saturday"
        ;;
    7)
        echo "Sunday"
        ;;
    *)
        echo "error"
esac
运行结果：
Input integer number:3↙
Wednesday
```

看了这个例子，相信大家对 case in 语句有了一个大体上的认识，那么，接下来我们就正式开始讲解 case in 的用法，它的基本格式如下：

```shell
case expression in
    pattern1)
        statement1
        ;;
    pattern2)
        statement2
        ;;
    pattern3)
        statement3
        ;;
    ……
    *)
        statementn
esac
```

case、in 和 esac 都是 Shell 关键字，expression 表示表达式，pattern 表示匹配模式。

- expression 既可以是一个变量、一个数字、一个字符串，还可以是一个数学计算表达式，或者是命令的执行结果，只要能够得到 expression 的值就可以。
- pattern 可以是一个数字、一个字符串，甚至是一个简单的正则表达式。

case 会将 expression 的值与 pattern1、pattern2、pattern3 逐个进行匹配：

- 如果 expression 和某个模式（比如 pattern2）匹配成功，就会执行这模式（比如 pattern2）后面对应的所有语句（该语句可以有一条，也可以有多条），直到遇见双分号`;;`才停止；然后整个 case 语句就执行完了，程序会跳出整个 case 语句，执行 esac 后面的其它语句。

- 如果 expression 没有匹配到任何一个模式，那么就执行`*)`后面的语句（`*`表示其它所有值），直到遇见双分号`;;`或者`esac`才结束。`*)`相当于多个 if 分支语句中最后的 else 部分。

> 如果你有C语言、C++、Java 等编程经验，这里的;;和*)就相当于其它编程语言中的 break 和 default。

对`*)`的几点说明：

- Shell case in 语句中的`*)`用来“托底”，万一 expression 没有匹配到任何一个模式，`*)`部分可以做一些“善后”工作，或者给用户一些提示。

- 可以没有`*)`部分。如果 expression 没有匹配到任何一个模式，那么就不执行任何操作。

除最后一个分支外（这个分支可以是普通分支，也可以是`*)`分支），其它的每个分支都必须以`;;`结尾，`;;`代表一个分支的结束，不写的话会有语法错误。最后一个分支可以写`;;`，也可以不写，因为无论如何，执行到 esac 都会结束整个 case in 语句。

上面的代码是 case in 最常见的用法，即 expression 部分是一个变量，pattern 部分是一个数字或者表达式。

#### 2.30.1case in 和正则表达式

case in 的 pattern 部分支持简单的正则表达式，具体来说，可以使用以下几种格式：

| 格式  | 说明                                                         |
| ----- | ------------------------------------------------------------ |
| *     | 表示任意字符串。                                             |
| [abc] | 表示 a、b、c 三个字符中的任意一个。比如，[15ZH] 表示 1、5、Z、H 四个字符中的任意一个。 |
| [m-n] | 表示从 m 到 n 的任意一个字符。比如，[0-9] 表示任意一个数字，[0-9a-zA-Z] 表示字母或数字。 |
| \|    | 表示多重选择，类似逻辑运算中的或运算。比如，abc \| xyz 表示匹配字符串 "abc" 或者 "xyz"。 |

如果不加以说明，Shell 的值都是字符串，expression 和 pattern 也是按照字符串的方式来匹配的；本节第一段代码看起来是判断数字是否相等，其实是判断字符串是否相等。

最后一个分支`*)`并不是什么语法规定，它只是一个正则表达式，`*`表示任意字符串，所以不管 expression 的值是什么，`*)`总能匹配成功。

下面的例子演示了如何在 case in 中使用正则表达式：

```shell
#!/bin/bash
printf "Input a character: "
read -n 1 char
case $char in
    [a-zA-Z])
        printf "\nletter\n"
        ;;
    [0-9])
        printf "\nDigit\n"
        ;;
    [,.?!])
        printf "\nPunctuation\n"
        ;;
    *)
        printf "\nerror\n"
esac
```

```shell
运行结果1：
Input integer number: S
letter

运行结果2：
Input integer number: ,
Punctuation
```

### 2.31Shell while循环

while 循环是 Shell 脚本中最简单的一种循环，当条件满足时，while 重复地执行一组语句，当条件不满足时，就退出 while 循环。

Shell while 循环的用法如下：

```shell
while condition
do
    statements
done
```

`condition`表示判断条件，`statements`表示要执行的语句（可以只有一条，也可以有多条），`do`和`done`都是 Shell 中的关键字。

while 循环的执行流程为：

- 先对 condition 进行判断，如果该条件成立，就进入循环，执行 while 循环体中的语句，也就是 do 和 done 之间的语句。这样就完成了一次循环。
- 每一次执行到 done 的时候都会重新判断 condition 是否成立，如果成立，就进入下一次循环，继续执行 do 和 done 之间的语句，如果不成立，就结束整个 while 循环，执行 done 后面的其它 Shell 代码。
- 如果一开始 condition 就不成立，那么程序就不会进入循环体，do 和 done 之间的语句就没有执行的机会。

`注意，在 while 循环体中必须有相应的语句使得 condition 越来越趋近于“不成立”，只有这样才能最终退出循环，否则 while 就成了死循环，会一直执行下去，永无休止。`

while 语句和 if else 语句中的 condition 用法都是一样的，你可以使用 test 或 [] 命令，也可以使用 (()) 或 [[]]。

#### 2.31.1while 循环举例

【实例1】计算从 1 加到 100 的和。

```shell
#!/bin/bash
i=1
sum=0
while ((i <= 100))
do
    ((sum += i))
    ((i++))
done
echo "The sum is: $sum"

运行结果：
The sum is: 5050
```

对上面的例子进行改进，计算从 m 加到 n 的值。

```shell
#!/bin/bash
read m
read n
sum=0
while ((m <= n))
do
    ((sum += m))
    ((m++))
done
echo "The sum is: $sum"

运行结果：
1↙
100↙
The sum is: 5050
```

【实例2】实现一个简单的加法计算器，用户每行输入一个数字，计算所有数字的和。

```shell
#!/bin/bash
sum=0
echo "请输入您要计算的数字，按 Ctrl+D 组合键结束读取"
while read n
do
    ((sum += n))
done
echo "The sum is: $sum"

运行结果：
12↙
33↙
454↙
6767↙
1↙
2↙
The sum is: 7269
```

在终端中读取数据，可以等价为在文件中读取数据，按下 Ctrl+D 组合键表示读取到文件流的末尾，此时 read 就会读取失败，得到一个非 0 值的退出状态，从而导致判断条件不成立，结束循环。

### 2.32Shell until循环用法

unti 循环和 while 循环恰好相反，当判断条件不成立时才进行循环，一旦判断条件成立，就终止循环。until 的使用场景很少，一般使用 while 即可。

Shell until 循环的用法如下：

```shell
until condition
do
    statements
done
```

`condition`表示判断条件，`statements`表示要执行的语句（可以只有一条，也可以有多条），`do`和`done`都是 Shell 中的关键字。

until 循环的执行流程为：

- 先对 condition 进行判断，如果该条件不成立，就进入循环，执行 until 循环体中的语句（do 和 done 之间的语句），这样就完成了一次循环。
- 每一次执行到 done 的时候都会重新判断 condition 是否成立，如果不成立，就进入下一次循环，继续执行循环体中的语句，如果成立，就结束整个 until 循环，执行 done 后面的其它 Shell 代码。
- 如果一开始 condition 就成立，那么程序就不会进入循环体，do 和 done 之间的语句就没有执行的机会。

注意，在 until 循环体中必须有相应的语句使得 condition 越来越趋近于“成立”，只有这样才能最终退出循环，否则 until 就成了死循环，会一直执行下去，永无休止。

```shell
#!/bin/bash
i=1
sum=0
until ((i > 100))
do
    ((sum += i))
    ((i++))
done
echo "The sum is: $sum"

运行结果：
The sum is: 5050
```

在 while 循环中，判断条件为`((i<=100))`，这里将判断条件改为`((i>100))`，两者恰好相反，请读者注意区分。

### 2.33Shell for循环

除了 while 循环和 until 循环，Shell 脚本还提供了 for 循环，它更加灵活易用，更加简洁明了。Shell for 循环有两种使用形式，下面我们逐一讲解。

#### 2.33.1C语言风格的 for 循环

C语言风格的 for 循环的用法如下：

```shell
for((exp1; exp2; exp3))
do
    statements
done
```

几点说明：

- exp1、exp2、exp3 是三个表达式，其中 exp2 是判断条件，for 循环根据 exp2 的结果来决定是否继续下一次循环；
- statements 是循环体语句，可以有一条，也可以有多条；
- do 和 done 是 Shell 中的关键字。

for 循环的执行过程可用下图表示：

![image-20230102175714273](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102175714273.png)

下面我们给出一个实际的例子，计算从 1 加到 100 的和。

```shell
#!/bin/bash
sum=0
for ((i=1; i<=100; i++))
do
    ((sum += i))
done
echo "The sum is: $sum"

运行结果：
The sum is: 5050
```

由此我们可以总结出 for 循环的一般形式为：

```shell
for(( 初始化语句; 判断条件; 自增或自减 ))
do
    statements
done
```

**for 循环中的三个表达式**

for 循环中的 exp1（初始化语句）、exp2（判断条件）和 exp3（自增或自减）都是可选项，都可以省略（但分号`;`必须保留）。

1)修改“从 1 加到 100 的和”的代码，省略 exp1：

```shell
#!/bin/bash
sum=0
i=1
for ((; i<=100; i++))
do
    ((sum += i))
done
echo "The sum is: $sum"
```

可以看到，将`i=1`移到了 for 循环的外面。

2)省略 exp2，就没有了判断条件，如果不作其他处理就会成为死循环，我们可以在循环体内部使用 break 关键字强制结束循环：

```shell
#!/bin/bash
sum=0
for ((i=1; ; i++))
do
    if(( i>100 )); then
        break
    fi
    ((sum += i))
done
echo "The sum is: $sum"
```

break 是 Shell 中的关键字，专门用来结束循环，后续章节还会深入讲解。

3)省略了 exp3，就不会修改 exp2 中的变量，这时可在循环体中加入修改变量的语句。例如：

```shell
#!/bin/bash
sum=0
for ((i=1; i<=100; ))
do
    ((sum += i))
    ((i++))
done
echo "The sum is: $sum"
```

4)最后给大家看一个更加极端的例子，同时省略三个表达式：

```shell
#!/bin/bash
sum=0
i=0
for (( ; ; ))
do
     if(( i>100 )); then
        break
    fi
    ((sum += i))
    ((i++))
done
echo "The sum is: $sum"
```

#### 2.33.2Python 风格的 for in 循环

Python 风格的 for in 循环的用法如下：

```shell
for variable in value_list
do
    statements
done
```

variable 表示变量，value_list 表示取值列表，in 是 Shell 中的关键字。

> in value_list 部分可以省略，省略后的效果相当于 in $@

每次循环都会从 value_list 中取出一个值赋给变量 variable，然后进入循环体（do 和 done 之间的部分），执行循环体中的 statements。直到取完 value_list 中的所有值，循环就结束了。

Shell for in 循环举例：

```shell
#!/bin/bash
sum=0
for n in 1 2 3 4 5 6
do
    echo $n
     ((sum+=n))
done
echo "The sum is "$sum

运行结果：
1
2
3
4
5
6
The sum is 21
```

**对 value_list 的说明**

取值列表 value_list 的形式有多种，你可以直接给出具体的值，也可以给出一个范围，还可以使用命令产生的结果，甚至使用通配符，下面我们一一讲解。

1)直接给出具体的值

可以在 in 关键字后面直接给出具体的值，多个值之间以空格分隔，比如`1 2 3 4 5`、`"abc" "390" "tom"`等。

上面的代码中用一组数字作为取值列表，下面我们再演示一下用一组字符串作为取值列表：

```shell
#!/bin/bash
for str in "C语言中文网" "http://c.biancheng.net/" "成立7年了" "日IP数万"
do
    echo $str
done

运行结果：
C语言中文网
http://c.biancheng.net/
成立7年了
日IP数万
```

2)给出一个取值范围

给出一个取值范围的具体格式为：

```shell
{start..end}
```

start 表示起始值，end 表示终止值；注意中间用两个点号相连，而不是三个点号。根据笔者的实测，这种形式只支持数字和字母。

例如，计算从 1 加到 100 的和：

```shell
#!/bin/bash
sum=0
for n in {1..100}
do
    ((sum+=n))
done
echo $sum

运行结果：
5050
```

再如，输出从 A 到 z 之间的所有字符：

```shell
#!/bin/bash
for c in {A..z}
do
    printf "%c" $c
done

输出结果：
ABCDEFGHIJKLMNOPQRSTUVWXYZ[]^_`abcdefghijklmnopqrstuvwxyz
```

可以发现，Shell 是根据 ASCII 码表来输出的。

3)使用命令的执行结果

使用反引号``或者\$()都可以取得命令的执行结果，我们在《Shell变量》一节中已经进行了详细讲解，并对比了两者的优缺点。本节我们使用\$()这种形式，因为它不容易产生混淆。

例如，计算从 1 到 100 之间所有偶数的和：

```shell
#!/bin/bash
sum=0
for n in $(seq 2 2 100)
do
    ((sum+=n))
done
echo $sum

运行结果：
2550
```

seq 是一个 Linux 命令，用来产生某个范围内的整数，并且可以设置步长，不了解的读者请自行百度。`seq 2 2 100`表示从 2 开始，每次增加 2，到 100 结束。

再如，列出当前目录下的所有 Shell 脚本文件：

```shell
#!/bin/bash
for filename in $(ls *.sh)
do
    echo $filename
done

运行结果：
demo.sh
test.sh
abc.sh
```

ls 是一个 Linux 命令，用来列出当前目录下的所有文件，`*.sh`表示匹配后缀为`.sh`的文件，也就是 Shell 脚本文件。

4)使用 Shell 通配符

`Shell 通配符可以认为是一种精简化的正则表达式，通常用来匹配目录或者文件，而不是文本。`有了 Shell 通配符，不使用 ls 命令也能显示当前目录下的所有脚本文件，请看下面的代码：

```shell
#!/bin/bash
for filename in *.sh
do
    echo $filename
done

运行结果：
demo.sh
test.sh
abc.sh
```

5)使用特殊变量

Shell 中有多个特殊的变量，例如 \$#、\$*、\$@、\$?、\$\$ 等，在 value_list 中就可以使用它们。

```shell
#!/bin/bash
function func(){
    for str in $@
    do
        echo $str
    done
}
func C++ Java Python C#

运行结果：
C++
Java
Python
C#
```

其实，我们也可以省略 value_list，省略后的效果和使用`$@`一样。请看下面的演示：

```shell
#!/bin/bash
function func(){
    for str
    do
        echo $str
    done
}
func C++ Java Python C#

运行结果：
C++
Java
Python
C#
```

### 2.34Shell select in循环

select in 循环用来`增强交互性`，它可以`显示出带编号的菜单`，用户输入不同的编号就可以选择不同的菜单，并执行不同的功能。

select in 是 `Shell 独有`的一种循环，非常适合终端（Terminal）这样的交互场景，C语言、C++、Java、Python、C# 等其它编程语言中是没有的。

Shell select in 循环的用法如下：

```shell
select variable in value_list
do
    statements
done
```

`variable` 表示变量，`value_list` 表示取值列表，`in` 是 Shell 中的关键字。你看，`select in` 和 `for in` 的语法是多么地相似。

我们先来看一个 select in 循环的例子：

```shell
#!/bin/bash
echo "What is your favourite OS?"
select name in "Linux" "Windows" "Mac OS" "UNIX" "Android"
do
    echo "You have selected $name"
done
```

运行结果：

```shell
What is your favourite OS?
1) Linux
2) Windows
3) Mac OS
4) UNIX
5) Android
#? 4↙
You have selected UNIX
#? 1↙
You have selected Linux
#? 9↙
You have selected
#? 2↙
You have selected Windows
#?^D
```

`#?`用来提示用户输入菜单编号；`^D`表示按下 Ctrl+D 组合键，它的作用是结束 select in 循环。

运行到 select 语句后，取值列表 value_list 中的内容会以菜单的形式显示出来，用户输入菜单编号，就表示选中了某个值，这个值就会赋给变量 variable，然后再执行循环体中的 statements（do 和 done 之间的部分）。

每次循环时 select 都会要求用户输入菜单编号，并使用环境变量 PS3 的值作为提示符，PS3 的默认值为`#?`，修改 PS3 的值就可以修改提示符。

如果用户输入的菜单编号不在范围之内，例如上面我们输入的 9，那么就会给 variable 赋一个空值；如果用户输入一个空值（什么也不输入，直接回车），会重新显示一遍菜单。

> 注意，select 是无限循环（死循环），输入空值，或者输入的值无效，都不会结束循环，只有遇到 break 语句，或者按下 Ctrl+D 组合键才能结束循环。

**完整实例**

select in 通常和 case in 一起使用，在用户输入不同的编号时可以做出不同的反应。修改上面的代码，加入 case in 语句：

```shell
#!/bin/bash
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

用户只有输入正确的编号才会结束循环，如果输入错误，会要求重新输入。

运行结果1，输入正确选项：

```shell
What is your favourite OS?
1) Linux
2) Windows
3) Mac OS
4) UNIX
5) Android
#? 2
Windows是微软开发的个人电脑操作系统，它是闭源收费的。
```

运行结果2，输入错误选项：

```shell
What is your favourite OS?
1) Linux
2) Windows
3) Mac OS
4) UNIX
5) Android
#? 7
输入错误，请重新输入
#? 4
UNIX是操作系统的开山鼻祖，现在已经逐渐退出历史舞台，只应用在特殊场合。
```

运行结果3，输入空值：

```shell
What is your favourite OS?
1) Linux
2) Windows
3) Mac OS
4) UNIX
5) Android
#?
1) Linux
2) Windows
3) Mac OS
4) UNIX
5) Android
#? 3
Mac OS是苹果公司基于UNIX开发的一款图形界面操作系统，只能运行与苹果提供的硬件之上。
```

### 2.35Shell break和continue跳出循环

使用 while、until、for、select 循环时，如果想提前结束循环（在不满足结束条件的情况下结束循环），可以使用 `break` 或者 `continue` 关键字。

在C语言、C++、C#、Python、Java 等大部分编程语言中，break 和 continue 只能跳出当前层次的循环，内层循环中的 break 和 continue 对外层循环不起作用；但是 Shell 中的 break 和 continue 却能够跳出多层循环，`也就是说，内层循环中的 break 和 continue 能够跳出外层循环。`

在实际开发中，break 和 continue 一般只用来跳出当前层次的循环，很少有需要跳出多层循环的情况。

#### 2.35.1break 关键字

Shell break 关键字的用法为：

```shell
break n
```

n 表示跳出循环的层数，如果省略 n，则表示跳出当前的整个循环。break 关键字通常和 if 语句一起使用，即满足条件时便跳出循环。

![image-20230102192129406](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102192129406.png)

【实例1】不断从终端读取用户输入的正数，求它们相加的和：

```shell
#!/bin/bash
sum=0
while read n; do
    if((n>0)); then
        ((sum+=n))
    else
        break
    fi
done
echo "sum=$sum"

运行结果：
10↙
20↙
30↙
0↙
sum=60
```

while 循环通过 read 命令的退出状态来判断循环条件是否成立，只有当按下 Ctrl+D 组合键（表示输入结束）时，`read n`才会判断失败，此时 while 循环终止。除了按下 Ctrl+D 组合键，你还可以输入一个小于等于零的整数，这样会执行 break 语句来终止循环（跳出循环）。

【实例2】使用 break 跳出双层循环。

如果 break 后面不跟数字的话，表示跳出当前循环，对于有两层嵌套的循环，就得使用两个 break 关键字。例如，输出一个 4*4 的矩阵：

```shell
#!/bin/bash
i=0
while ((++i)); do  #外层循环
    if((i>4)); then
        break  #跳出外层循环
    fi
    j=0;
    while ((++j)); do  #内层循环
        if((j>4)); then
            break  #跳出内层循环
        fi
        printf "%-4d" $((i*j))
    done
    printf "\n"
done
运行结果：
1   2   3   4  
2   4   6   8  
3   6   9   12 
4   8   12  16 
```

我们也可以在 break 后面跟一个数字，让它一次性地跳出两层循环，请看下面的代码：

```shell
#!/bin/bash
i=0
while ((++i)); do  #外层循环
    j=0;
    while ((++j)); do  #内层循环
        if((i>4)); then
            break 2  #跳出内外两层循环
        fi
        if((j>4)); then
            break  #跳出内层循环
        fi
        printf "%-4d" $((i*j))
    done
    printf "\n"
done
```

修改后的代码将所有 break 都移到了内层循环里面。读者需要重点关注`break 2`这条语句，它使得程序可以一次性跳出两层循环，也就是先跳出内层循环，再跳出外层循环。

#### 2.35.2continue 关键字

Shell continue 关键字的用法为：

```shell
continue n
```

n 表示循环的层数：

- 如果省略 n，则表示 continue 只对当前层次的循环语句有效，遇到 continue 会跳过本次循环，忽略本次循环的剩余代码，直接进入下一次循环。
- 如果带上 n，比如 n 的值为 2，那么 continue 对内层和外层循环语句都有效，不但内层会跳过本次循环，外层也会跳过本次循环，其效果相当于内层循环和外层循环**同时**执行了不带 n 的 continue。这么说可能有点难以理解，稍后我们通过代码来演示。

continue 关键字也通常和 if 语句一起使用，即满足条件时便跳出循环。

![image-20230102192603136](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230102192603136.png)

【实例1】不断从终端读取用户输入的 100 以内的正数，求它们的和：

```shell
#!/bin/bash
sum=0
while read n; do
    if((n<1 || n>100)); then
        continue
    fi
    ((sum+=n))
done
echo "sum=$sum"

运行结果：
10↙
20↙
-1000↙
5↙
9999↙
25↙
sum=60
```

注意，只有按下 Ctrl+D 组合键输入才会结束，`read n`才会判断失败，while 循环才会终止。

【实例2】使用 continue 跳出多层循环，请看下面的代码：

```shell
#!/bin/bash
for((i=1; i<=5; i++)); do
    for((j=1; j<=5; j++)); do
        if((i*j==12)); then
            continue 2
        fi
        printf "%d*%d=%-4d" $i $j $((i*j))
    done
    printf "\n"
done
运行结果：
1*1=1   1*2=2   1*3=3   1*4=4   1*5=5  
2*1=2   2*2=4   2*3=6   2*4=8   2*5=10 
3*1=3   3*2=6   3*3=9   4*1=4   4*2=8   5*1=5   5*2=10  5*3=15  5*4=20  5*5=25
```

从运行结果可以看出，遇到`continue 2`时，不但跳过了内层 for 循环，也跳过了外层 for 循环。

### 2.36Shell函数详解

`Shell 函数的本质是一段可以重复使用的脚本代码，这段代码被提前编写好了，放在了指定的位置，使用时直接调取即可。`

Shell 中的函数和C++、Java、Python、C# 等其它编程语言中的函数类似，只是在语法细节有所差别。

Shell 函数定义的语法格式如下：

```shell
function name() {
    statements
    [return value]
}
```

对各个部分的说明：

- `function`是 Shell 中的关键字，专门用来定义函数；
- `name`是函数名；
- `statements`是函数要执行的代码，也就是一组语句；
- `return value`表示函数的返回值，其中 return 是 Shell 关键字，专门用在函数中返回一个值；这一部分可以写也可以不写。

由`{ }`包围的部分称为函数体，调用一个函数，实际上就是执行函数体中的代码。

#### 2.36.1函数定义的简化写法

如果你嫌麻烦，函数定义时也可以不写 function 关键字：

```shell
name() {
    statements
    [return value]
}
```

如果写了 function 关键字，也可以省略函数名后面的小括号：

```shell
function name {
    statements
    [return value]
}
```

我建议使用标准的写法，这样能够做到“见名知意”，一看就懂。

#### 2.36.2函数调用

调用 Shell 函数时可以给它传递参数，也可以不传递。如果不传递参数，直接给出函数名字即可：

```shell
name
```

如果传递参数，那么多个参数之间以空格分隔：

```shell
name param1 param2 param3
```

不管是哪种形式，函数名字后面都不需要带括号。

和其它编程语言不同的是，Shell 函数在定义时不能指明参数，但是在调用时却可以传递参数，并且给它传递什么参数它就接收什么参数。

Shell 也不限制定义和调用的顺序，你可以将定义放在调用的前面，也可以反过来，将定义放在调用的后面。

#### 2.36.3示例演示

1)定义一个函数，输出 Shell 教程的地址：

```shell
#!/bin/bash
#函数定义
function url(){
    echo "http://c.biancheng.net/shell/"
}
#函数调用
url
运行结果：
http://c.biancheng.net/shell/
```

你可以将调用放在定义的前面，也就是写成下面的形式：

```shell
#!/bin/bash
#函数调用
url
#函数定义
function url(){
    echo "http://c.biancheng.net/shell/"
}
```

2)定义一个函数，计算所有参数的和：

```shell
#!/bin/bash
function getsum(){
    local sum=0
    for n in $@
    do
         ((sum+=n))
    done
    return $sum
}
getsum 10 20 55 15  #调用函数并传递参数
echo $?

运行结果：
100
```

`$@`表示函数的所有参数，`$?`表示函数的退出状态（返回值）。

### 2.37Shell函数参数

和 C++、C#、Python 等大部分编程语言不同，Shell 中的函数在定义时不能指明参数，但是在调用时却可以传递参数。

函数参数是 `Shell 位置参数`的一种，在函数内部可以使用`$n`来接收，例如，1 表示第一个参数，2 表示第二个参数，依次类推。

除了`$n`，还有另外三个比较重要的变量：

- `$#`可以获取传递的参数的个数；
- `$@`或者`$*`可以一次性获取所有的参数。

【实例1】使用 $n 来接收函数参数。

```shell
#!/bin/bash
#定义函数
function show(){
    echo "Tutorial: $1"
    echo "URL: $2"
    echo "Author: "$3
    echo "Total $# parameters"
}
#调用函数
show C# http://c.biancheng.net/csharp/ Tom
运行结果：
Tutorial: C#
URL: http://c.biancheng.net/csharp/
Author: Tom
Total 3 parameters
```

【实例2】使用 $@ 来遍历函数参数。

定义一个函数，计算所有参数的和：

```shell
#!/bin/bash
function getsum(){
    local sum=0
    for n in $@
    do
         ((sum+=n))
    done
    echo $sum
    return 0
}
#调用函数并传递参数，最后将结果赋值给一个变量
total=$(getsum 10 20 55 15)
echo $total
#也可以将变量省略
echo $(getsum 10 20 55 15)

运行结果：
100
100
```

### 2.38Shell 函数返回值精讲

在 C++、Java、C#、Python 等大部分编程语言中，返回值是指函数被调用之后，执行函数体中的代码所得到的结果，这个结果就通过 return 语句返回。

但是 Shell 中的返回值表示的是函数的退出状态：返回值为 0 表示函数执行成功了，返回值为非 0 表示函数执行失败（出错）了。if、while、for 等语句都是根据函数的退出状态来判断条件是否成立。

`Shell 函数的返回值只能是一个介于 0~255 之间的整数，其中只有 0 表示成功，其它值都表示失败。`

函数执行失败时，可以根据返回值（退出状态）来判断具体出现了什么错误，比如一个打开文件的函数，我们可以指定 1 表示文件不存在，2 表示文件没有读取权限，3 表示文件类型不对。

如果函数体中没有 return 语句，那么使用默认的退出状态，也就是最后一条命令的退出状态。如果这就是你想要的，那么更加严谨的写法为：

```shell
return $?
```

`$?`是一个特殊变量，用来获取上一个命令的退出状态，或者上一个函数的返回值。

#### 2.38.1如何得到函数的处理结果

有人可能会疑惑，既然 return 表示退出状态，那么该如何得到函数的处理结果呢？比如，我定义了一个函数，计算从 m 加到 n 的和，最终得到的结果该如何返回呢？

这个问题有两种解决方案：

- 一种是借助全局变量，将得到的结果赋值给全局变量；
- 一种是在函数内部使用 echo、printf 命令将结果输出，在函数外部使用$()或者``捕获结果。

下面我们具体来定义一个函数 getsum，计算从 m 加到 n 的和，并使用以上两种解决方案。

【实例 1】将函数处理结果赋值给一个全局变量。

```shell
#!/bin/bash
sum=0 #全局变量
function getsum(){
	for((i=$1; i<=$2; i++)); do
		((sum+=i)) #改变全局变量
	done
	return $? #返回上一条命令的退出状态
}
read m
read n
if getsum $m $n; then
	echo "The sum is $sum" #输出全局变量
else
	echo "Error!"
fi

运行结果：
1
100
The sum is 5050
```

这种方案的弊端是：定义函数的同时还得额外定义一个全局变量，如果我们仅仅知道函数的名字，但是不知道全局变量的名字，那么也是无法获取结果的。

【实例 2】在函数内部使用 echo 输出结果。

```shell
#!/bin/bash
function getsum(){
	local sum=0 #局部变量
	for((i=$1; i<=$2; i++)); do
		((sum+=i)) #改变全局变量
	done
	echo $sum
	return $? #返回上一条命令的退出状态
}
read m
read n
total=$(getsum $m $n)
echo "The sum is $total"
#也可以省略 total 变量，直接写成下面的形式
 #echo "The sum is "$(getsum $m $n)
运行结果：
1
100
The sum is 5050
```

这种方案的弊端是：如果不使用\$()，而是直接调用函数，那么就会将结果直接输出到终端上，不过这貌似也无所谓，所以我推荐这种方案。
