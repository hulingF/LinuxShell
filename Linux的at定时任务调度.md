# Linux的at定时任务调度

`与crond不同的是，at任务调度是一次性的，而crond是重复性的。`

## 1.基本介绍

1. at命令是一次性定时计划任务，at的守护进程atd会以后台模式运行，检查作业队列来运行。
2. 默认情况下，atd守护进程每60s检查作业队列，有作业时，会检查作业运行时间，如果时间与当前时间匹配，则运行此作业。
3. at命令是一次性定时计划任务，执行完一个任务后就不再执行这个任务了。
4. 在使用at命令的时候，一定要保证atd进程的启动，可以使用相关指令查看：ps -ef | grep atd 可以检查atd是否在运行。

![image-20221231102023296](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231102023296.png)

## 2.at命令格式

at [选项] [时间]

按两次 `ctrl+d` 结束at命令的输入

## 3. 选项

![image-20221231101724050](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231101724050.png)

## 4.at指令的时间定义

1. 当天的hh:mm（小时：分钟），假如这个时间已经过去，那么就第二天的这个时间执行。例如04:00。
2. 模糊的词语，例如midnight、noon、teatime（下午茶时间，16:00左右）。
3. 采用 12 小时计时制，即在时间后面加上 AM（上午）或 PM（下午）来说明是上午还是下午。
4. 指定执行命令的具体日期，格式为month dat（月 日）或者mm/dd/yy或dd.mm.yy，指定的日期必须跟着写在在指定时间的后面，例如：04:00 2021-3-1就是2021年3月1日凌晨4点整执行。
5. 相对计时法，指定格式为now + count time-units，now就是当前时间，time-units是时间单位，可以是minutes、hours、days、weeks。count是时间的数量，例如：now + 5 minutes。
6. 直接用today、tomorrow来指定完成命令的时间。

## 5.应用案例

案例 1：2 天后的下午 5 点执行 /bin/ls /home

![image-20221231102354332](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231102354332.png)

案例 2：atq 命令来查看系统中没有执行的工作任务

![image-20221231102413655](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231102413655.png)

案例 3：明天 17 点钟，输出时间到指定文件内 比如 /root/date100.log

![image-20221231102530564](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231102530564.png)

案例 4：2 分钟后，输出时间到指定文件内 比如 /root/date200.log

![image-20221231102645440](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231102645440.png)

2分钟后再次编辑任务：

![image-20221231102922564](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231102922564.png)