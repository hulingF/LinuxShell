# Linux磁盘分区挂载

## Linux分区

### 原理介绍

Linux 来说无论有几个分区，分给哪一目录使用，它归根结底就只有一个根目录，一个独立且唯一的文件结构 , Linux 中每个分区都是用来组成整个文件系统的一部分。Linux 采用了一种叫“载入”的处理方法，它的整个文件系统中包含了一整套的文件和目录，且将一个分区和一个目录联系起来。这时要载入的一个分区将使它的存储空间在一个目录下获得。

![image-20221231112445004](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231112445004.png)

### 硬盘说明

Linux 硬盘分 IDE 硬盘和 SCSI 硬盘，目前基本上是 SCSI 硬盘。

对于 IDE 硬盘，驱动器标识符为“hdx~”,其中“hd”表明分区所在设备的类型，这里是指 IDE 硬盘了。“x”为盘号（a 为基本盘，b 为基本从属盘，c 为辅助主盘，d 为辅助从属盘）,“~”代表分区，前四个分区用数字 1 到 4 表示，它们是主分区或扩展分区，从5开始就是逻辑分区。例，hda3 表示为第一个 IDE 硬盘上的第三个主分区或扩展分区,hdb2表示为第二个 IDE 硬盘上的第二个主分区或扩展分区。

对于 SCSI 硬盘则标识为“sdx~”，SCSI 硬盘是用“sd”来表示分区所在设备的类型的，其余则和 IDE 硬盘的表示方法一样。

![image-20221231112633189](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231112633189.png)

### 查看所有设备挂载情况

命令 ：`lsblk` 或者 `lsblk -f`

![image-20221231114158080](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231114158080.png)

![image-20221231112729097](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231112729097.png)

![image-20221231112748470](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231112748470.png)

## 挂载的经典案例

下面我们以**增加一块硬盘**为例来熟悉下磁盘的相关指令和深入理解磁盘分区、挂载、卸载的概念。

### 如何增加一块硬盘

1) 虚拟机添加硬盘
2) 分区
3) 格式化
4) 挂载
5) 设置可以自动挂载

### 虚拟机增加硬盘步骤 1

在【虚拟机】菜单中，选择【设置】，然后设备列表里添加硬盘，然后一路【下一步】，中间只有选择磁盘大小的地方需要修改，至到完成。然后重启系统（才能识别）！

![image-20221231112952597](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231112952597.png)

### 虚拟机增加硬盘步骤 2

分区命令 `fdisk /dev/sdb` 开始对/sdb 分区。

m 显示命令列表

p 显示磁盘分区 同 fdisk –l

n 新增分区

d 删除分区

w 写入并退出

![image-20221231114303901](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231114303901.png)

> 说明： 开始分区后输入 n，新增分区，然后选择 p ，分区类型为主分区。两次回车默认剩余全部空间。最后输入 w写入分区并退出，若不保存退出输入 q。

![image-20221231113120832](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231113120832.png)

### 虚拟机增加硬盘步骤 3

格式化磁盘:分区命令:mkfs -t ext4 /dev/sdb1 其中 ext4 是分区类型。

![image-20221231114351312](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231114351312.png)

### 虚拟机增加硬盘步骤 4

挂载: 将一个分区与一个目录联系起来， mount 设备名称 挂载目录 例如： mount 

/dev/sdb1 /newdisk。

> 老师注意: 用命令行挂载,**重启后会失**效

### 虚拟机增加硬盘步骤 5

永久挂载: 通过修改`/etc/fstab` 实现挂载 添加完成后 执行 mount –a 即刻生效.

![image-20221231113513049](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231113513049.png)

## 磁盘情况查询

### 查询系统整体磁盘使用情况

基本语法:`df -h`

![image-20221231114513698](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231114513698.png)

![image-20221231113621358](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231113621358.png)

### 查询指定目录的磁盘占用情况

基本语法:`du -h`

![image-20221231114552136](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231114552136.png)

查询指定目录的磁盘占用情况，默认为当前目录

-s 指定目录占用大小汇总

-h 带计量单位

-a 含文件

--max-depth=1 子目录深度

-c 列出明细的同时，增加汇总值

案例:查询 /opt 目录的磁盘占用情况，深度为 1

![image-20221231113729648](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231113729648.png)

## 磁盘情况-工作实用指令

![image-20221231114618629](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231114618629.png)

- 统计/opt 文件夹下文件的个数

```shell
ls -l /opt | grep "^-" | wc -l
```

- 统计/opt 文件夹下目录的个数

```shell
ls -l /opt | grep "^d" | wc -l
```

- 统计/opt 文件夹下文件的个数，包括子文件夹里的

```shell
ls -lR /opt | grep "^-" | wc -l
```

- 统计/opt 文件夹下目录的个数，包括子文件夹里的

```shell
ls -lR /opt | grep "^d" | wc -l
```

- 以树状显示目录结构 tree 目录,注意，如果没有 tree ,则使用 **yum install tree** 安装

![image-20221231114002071](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231114002071.png)



