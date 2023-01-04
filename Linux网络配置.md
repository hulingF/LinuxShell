# Linux网络配置

## 一.网络地址管理

### 1.1网络地址查看–ifconfig

命令格式：`ifconf` 或 `ifconfig +网卡名`
主要参数信息：

![image-20221231132645907](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231132645907.png)

### 1.2网络配置修改

临时修改IP地址：`ifconfig +网卡 +更改后的IP地址`

```shell
[root@xiayan ~]# ifconfig ens33 192.168.48.10
[root@xiayan ~]# ifconfig ens33 
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.48.10  netmask 255.255.255.0  broadcast 192.168.48.255
        inet6 fe80::3ab8:991b:a38a:e6bd  prefixlen 64  scopeid 0x20<link>
        ether 00:0c:29:4f:79:cd  txqueuelen 1000  (Ethernet)
```

临时启动与关闭网卡:`ifconfig +网卡 up  #打开` `ifconfig +网卡 down #关闭`

永久修改IP地址:网卡配置文件存放在 `/etc/sysconfig/network-scripts/ifcfg-ens33`,可以对网卡编辑进行修改。

```tex
TYPE=Ethernet                             #网卡类型，Ethernet为以太网
BOOTPROTO=static                          #网络配置方式。static为静态dhcp为动态 
DEVICE=ens33                              #网络接口名称                  NAME=ens33                                #网络接口名称
UUID=09fb2b87-8a2a-4f57-b4cf-9cd8040c9c   #网卡地址
ONBOOT=yes                                #是否开机自启动
IPADDR=192.168.48.6                       #网络接口IP地址
GATEWAY=192.168.48.2                      #网络接口默认网关
NETMASK=255.255.255.0                     #网络接口子网掩码
DNS1=114.114.114.114                      #域名解析服务器地址
```

修改配置文件后，需要重启网络服务：`systemctl restart network`

### 1.3网络虚拟接口设置

`ifconfig 网卡：序号 +IP地址`

![image-20221231133630800](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231133630800.png)

### 1.4虚拟机NAT模式

![image-20221231143418301](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231143418301.png)

![image-20221231143439102](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231143439102.png)

当虚拟机与宿主机进行通信时：其实就是10.0.0.1/24与10.0.0.132/24这两个地址进行通信；
当虚拟机与外网进行通信时：虚拟机先把数据发送到网关10.0.0.2/24，然后再通过NAT服务器把地址转换为192.168.1.100/24，然后再与外网进行通信；
如果把Vmnet8这块虚拟网卡禁用，还是不影响虚拟机访问互联网，只是宿主机与虚拟机的通信会受到影响，从上面的图示中不难看出。

## 二.路由表设置

### 2.1路由表查看–route

路由表:Linux操作系统中的路由表决定着从本机向其他主机、其他网络发送数据的去向，是排除网络故障的关键信息。

> lo, local的简写，一般指本地环回接口。一般回环接口的ip v4地址为:127.0.0.1，子网掩码：255.255.255.0。假如包是由一个本地进程为另一个本地进程产生的, 它们将通过外出链的'lo'接口,然后返回进入链的'lo'接口
>
> virbr0 是 KVM 默认创建的一个 Bridge，其作用是为连接其上的虚机网卡提供 NAT 访问外网的功能。virbr0 默认分配了一个IP 192.168.122.1，并为连接其上的其他虚拟网卡提供 DHCP 服务。

![image-20221231135219325](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231135219325.png)

![image-20221231134704293](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231134704293.png)

![image-20221231135851235](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231135851235.png)

![image-20221231135912249](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231135912249.png)

命令格式：`route`     route -n 将路由记录中的地址显示为数字形式

![image-20221231133840532](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231133840532.png)

直接执行“route"命令可以查看当前主机中的路由表信息

### 2.2路由表设置

#### 2.2.1添加指定网段到路由表

`route add -net 网段地址 gw IP地址`

![image-20221231140155299](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231140155299.png)

#### 2.2.2删除指定的网段

`route del -net 网段地址`

![image-20221231140237794](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231140237794.png)

#### 2.2.3添加默认路由到路由表

`route add default gw 网关地址`

![image-20221231140332263](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231140332263.png)

#### 2.2.4从路由表中删除默认网关

`route del default gw IP地址`

![image-20221231140414158](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231140414158.png)

## 三.网络连接测试

### 3.1测试网络连通性–ping

命令格式：`ping 【选项】 目标主机名或IP`

| 选项 | 功能             |
| ---- | ---------------- |
| -c   | 指定发包次数     |
| -i   | 指定发包间隔时间 |
| -w   | 超时时间间隔     |

![image-20221231140515009](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231140515009.png)

若看到"`Destination Host Unrelachable`"的反馈信息，则表示目的主机不可达，可能`目标地址不存在或者主机已经关闭`。
若看到"`Network is unreachable`" 的反馈信息，则表示`没有可用的路由记录`(如默认网关)，无法达到目标主机所在的网络。
当`目标主机有严格的防火墙限制`时，或者当网络中存在影响通信过程稳定性的因素(如网卡故障、病毒或网络攻击等)时，可能收到"`Request timeout`"的反馈结果

### 3.2跟踪数据包路径–traceroute

traceroute命令能够比ping命令更加准确得定位网络连接故障点。
命令格式：`traceroute +目标主机名或IP`

![image-20221231140728842](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231140728842.png)

## 四.域名解析

### 4.1域名解析–nslookup

![image-20221231143250430](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231143250430.png)

通过网站地址，解析出对方的IP地址
`nslookup 目标主机地址 [DNS服务器地址]` #测试DNS域名解析

![image-20221231140842446](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231140842446.png)

或使用`dig`命令

![image-20221231140919860](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231140919860.png)

### 4.2DNS设置

更改DNS两种方法
方法一：`vim /etc/resolv.conf`配置文件实时生效

![image-20221231141034787](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231141034787.png)

方法二：`vim /etc/sysconfig/network-scripts/ifcfg-ens33` 修改网卡信息中的DNS, 修改配置文件后，需要重启网络服务：`systemctl restart network`

![image-20221231141137696](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231141137696.png)

### 4.3本地主机映射

默认情况下，系统首先从hosts 文件查找解析记录，hosts文件只对当前的主机有效，hosts文件可减少DNS查询过程，从而加快访问速度。host文件位置：`/etc/hosts`
添加格式：`主机IP IP地址`

不更改host直接ping百度:

![image-20221231141244074](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231141244074.png)

`vim /etc/hosts`

![image-20221231141325667](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231141325667.png)

更改过hosts再ping百度:

![image-20221231141351113](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231141351113.png)

## 五.端口检查

### 5.1netstat命令查看

命令格式：`nststat 【选项】`

![image-20221231141436136](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231141436136.png)

查看系统正在运行的TCP端口信息

![image-20221231141509536](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231141509536.png)

查看TCP协议的80端口

![image-20221231141528322](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231141528322.png)

### 5.2ss命令查看

命令格式：`ss 【选项】`
ss常用选项

![image-20221231142847751](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231142847751.png)

查看ssh端口状态

![image-20221231142918792](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231142918792.png)

### 5.3lsof命令

命令格式： `lsof -i：+端口号`
查看22端口使用

![image-20221231142954330](C:\Users\lan\AppData\Roaming\Typora\typora-user-images\image-20221231142954330.png)