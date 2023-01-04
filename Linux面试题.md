# Linux面试题

1.分析日志 t.log(访问量)，将各个 ip 地址截取，并统计出现次数,并按从大到小排序(腾

讯)

```tex
http://192.168.200.10/index1.html
http://192.168.200.10/index2.html
http://192.168.200.20/index1.html
http://192.168.200.30/index1.html
http://192.168.200.40/index1.html
http://192.168.200.30/order.html
http://192.168.200.10/order.html
```

![image-20230101191741603](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101191741603.png)

2.统计连接到服务器的各个 ip 情况，并按连接数从大到小排序 (腾讯)

![image-20230101191956479](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101191956479.png)

![image-20230101192222458](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101192222458.png)

3.统计 ip 访问情况，要求分析 nginx 访问日志(access.log)，找出访问页面数量在前 2 位的 ip(美团)

![image-20230101194025046](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101194025046.png)

![image-20230101194152041](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101194152041.png)

4.使用 tcpdump 监听本机, 将来自 ip 192.168.80.131，tcp 端口为 22 的数据，保存输出到tcpdump.log , 用做将来数据分析(美团)

![image-20230101194735938](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101194735938.png)

5.如果你是系统管理员，在进行 Linux 系统权限划分时,应考虑哪些因素?（腾讯）

1)首先阐述 Linux 权限的主要对象

![image-20230101200334229](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101200334229.png)

2)根据自己实际经验谈考虑因素

1. 注意权限分离，比如: 工作中，Linux 系统权限和数据库权限不要在同一个部门
2. 权限最小原则(即:在满足使用的情况下最少优先)
3. 减少使用 root 用户，尽量用普通用户+sudo 提权进行日常操作
4. 重要的系统文件，比如/etc/passwd, /etc/shadow etc/fstab，/etc/sudoers 等,日常建议使用 chattr(change attribute)锁定, 需要操作时再打开[比如: 锁定 /etc/passwd 让任何用户都不能随意 useradd,除非解除锁定]
5. 可以利用工具，比如 chkrootkit/rootkit hunter 检测 rootkit 脚本

6.权限操作思考题

1)用户 tom 对目录 /home/test 有执行 x 和读 r 写 w 权限，/home/test/hello.java 是只读文件，问 tom 对 hello.java 文件能读吗(ok)? 能修改吗(no)？能删除吗?(ok)

2)用户tom 对目录 /home/test 只有读写权限，/home/test/hello.java 是只读文件，问tom对 hello.java文件能读吗(no)? 能修改吗(no)？能删除吗(no)?

3)用户 tom 对目录 /home/test 只有执行权限 x，/home/test/hello.java 是只读文件，问 tom 对 hello.java 文件能读吗(ok)?能修改吗(no)？能删除吗(no)?

4)用户 tom 对目录 /home/test 只有执行和写权限，/home/test/hello.java 是只读文件，问 tom 对 hello.java 文件能读吗(ok)? 能修改吗(no)？能删除吗(ok)?

7.说明 Centos7 启动流程，并说明和 CentOS6 相同和不同的地方(腾讯)

![image-20230101203308240](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101203308240.png)

![image-20230101203320238](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101203320238.png)

8.Linux 查看内存、io 读写、磁盘存储、端口占用、进程查看命令是什么?(瓜子)

```shell
top, iotop, df -lh , netstat -tunlp , ps -aux | grep sshd
```

9.使用 Linux 命令计算 t2.txt 第二列的和并输出 (美团)

![image-20230101204140082](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101204140082.png)

```shell
cat t2.txt | awk -F " " '{sum+=$2} END {print sum}'
```

10.Shell 脚本里如何检查一个文件是否存在？并给出提示(百度)

```shell
if [ -f 文件名 ] then echo “存在” else echo “不存在” fi
```

11.用 shell 写一个脚本，对文本 t3.txt 中无序的一列数字排序, 并将总和输出(百度)

```shell
sort -nr t3.txt | awk '{sum+=$1; print $1} END {print "和="sum}
```

12.请用指令写出查找当前文件夹（/home）下所有的文本文件内容中包含有字符 “

cat”的文件名称(金山)

```shell
grep -r "cat" /home | cut -d ":" -f 1
```

13.请写出统计/home 目录下所有文件个数和所有文件总行数的指令(在金山面试题扩展)

```shell
find /home/test -name "*.*" | wc -l
find /home/test -name "*.*" | xargs wc -l
```

14.每天晚上 10 点 30 分，打包站点目录/var/spool/mail 备份到/home 目录下（每次备份按时间生成不同的备份包 比如按照 年月日时分秒）(滴滴)

![image-20230101210746398](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101210746398.png)

![image-20230101210800778](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20230101210800778.png)

15.如何优化 Linux 系统， 说出你的方法 (瓜子)

