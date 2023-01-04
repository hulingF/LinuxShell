# Linux实时任务调度(crontab命令)

## 1.定义

crond 是Linux下用周期性的执行某种任务或者等待处理某些事件的一个守护进程，crond 进程会每分钟定期检查是否有要执行的任务，如果有要执行的任务则自动执行该任务。

## 2.Linux下的任务调度

1. 系统任务调度：系统周期性所要执行的工作，如：写缓存数据到硬盘、清理日志等。系统任务调度的配置文件 /etc/crontab。
2. 用户任务调度：用户定期要执行的工作，比如数据库备份、定时邮件提醒等。所有用户定义的crontab文件都保存在/var/spool/cron目录中。文件名与用户名一致。

## 3.crontab文件的含义

用户所建立的crontab文件中，每一行代表一项任务，每行的每个字段代表一项设置，共分六个字段，前五段是时间设定段，第六段是要执行的命令段。
==minute hour day month week command==

![image-20221231094703319](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231094703319.png)

![image-20221231094759177](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231094759177.png)

```sql
	在以上各个字段中，还可以使用以下特殊字符：
 	
 	星号（*）：代表所有可能的值，例如month字段如果是星号，则表示在满足其它字段的制约条件后每月都执行该命令操作。
 	
 	逗号（,）：可以用逗号隔开的值指定一个列表范围，例如，“1,2,5,7,8,9”
 	
 	中杠（-）：可以用整数之间的中杠表示一个整数范围，例如“2-6”表示“2,3,4,5,6”
 	
 	正斜线（/）：可以用正斜线指定时间的间隔频率，例如“0-23/2”表示每两小时执行一次。同时正斜线可以和星号一起使用，例如*/10，如果用在minute字段，表示每十分钟执行一次
```

## 4.crontab服务

1. service crond start //启动服务
2. service crond stop //关闭服务
3. service crond restart //重启服务
4. service crond reload //重新载入配置
5. service crond status //查看服务状态

`查看服务是否已经运行用 ps -ax | grep cron`

## 5.crontab命令选项

1. -u指定一个用户
2. -l列出某个用户的任务计划
3. -r删除某个用户的任务
4. -e编辑某个用户的任务

## 6.新增任务调度

1. 在命令行输入: crontab -e 然后添加相应的任务，wq存盘退出。
2. 直接编辑/etc/crontab 文件，即vim /etc/crontab，添加相应的任务。

## 7.查看任务调度

```shell
crontab -l //列出当前的所有调度任务
crontab -l -u root   //列出用户root的所有调度任务
```

## 8.删除任务调度

```shell
crontab -r   //删除所有任务调度工作
```

**利用任务调度执行Shell脚本，在Shell脚本中执行PHP文件，可以做到每秒执行一次PHP文件**

## 9.应用案例

![image-20221231100227791](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231100227791.png)