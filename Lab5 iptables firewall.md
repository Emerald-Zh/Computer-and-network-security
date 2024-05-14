

参考：
[iptables实验](https://www.cnblogs.com/Qi-Lin/p/12221904.html)
[WinSSHD安装配置](https://www.linuxidc.com/Linux/2017-07/145984.htm)


## 1. Lab Overview

**Project goal: configure iptables, which has been installed in Linux kernel.**

**Lab details:** No required material. You may materials on the internet.

**Your Tasks: ( You must learn iptables by yourselves)**

Tasks:
一. Part A

* Configure NAT in m2 using NAT table with POSTROUTING chain:
MASQUERADE packets so that internal IP addresses are hidden from external network
From m1 and m3, only allow ssh to external network
* This NAT configuration will remain in force throughout the lab

二. Part B

Write Rules for packets originating from or terminating at m2 (the gateway) in order to achieve the following goals:

1. Allow ssh connections originating from m2 and destined to m2.

2. Allow pings originating from m2 and destined to m2.

3. Block all other traffic to or from m2.

4. Hint: Part B requires INPUT and OUTPUT chains but no FORWARD chain

三. Part C

* Flush filter table rules from Part B.
* Allow only m1 (and not m3) to initiate an ssh session to hosts in the external network
* Reject all other traffic
* Hint: Part C requires FORWARD, INPUT and OUTPUT chains

## 2. Environment

![输入图片说明](/imgs/2024-05-11/9usS0FSmKE5WZycr.png)
M1 内网 192.168.184.132
M2 网关  外网：172.20.10.11 内网：192.168.184.128
M3 内网  192.168.184.131
M4 外网 172.20.10.12

### 2.1 具体环境配置
1. M2 添加网络适配器，分别为桥接模式和仅主机模式。
查看ip为：![输入图片说明](/imgs/2024-05-11/EcpOLO9vGEm6Ovfl.png)
<![endif]-->

外网为:172.20.10.11
内网为 :192.168.184.128
`` gedit /ect//sysctl.conf``
取消IPV4部分注释
![输入图片说明](/imgs/2024-05-11/3KTVrvvgbw4z7XTS.png)
运行使配置生效
``sysctl -p``

2. 配置主机m1和m3，操作相同
![输入图片说明](/imgs/2024-05-11/hculs1dwaT0Mam6U.png)

修改路由为m2
`` sudo route add default gw 192.168.184.128  ``
![输入图片说明](/imgs/2024-05-11/eAUeXHFlcMAfni0O.png)

3. 配置m4
更改为桥接模式，更改网关为m2
`` sudo route add default gw 172.20.10.11  ``

![输入图片说明](/imgs/2024-05-11/GLlhDz2K6ALd4xzy.png)

### 2.2 测试配置
（1）外网之间
m2 -> m4
![输入图片说明](/imgs/2024-05-11/qwOOb2eiKe9jzRgS.png)![输入图片说明](/imgs/2024-05-11/VG2kEnyefPLubClu.png)

（2）内网之间
m1->m3![输入图片说明](/imgs/2024-05-11/AW1JNcoy4OaXGDHC.png)
m1->m2
![输入图片说明](/imgs/2024-05-11/aqG3cyT2JNvFMLk2.png)
（3）内网和外网之间
m1->m4
![输入图片说明](/imgs/2024-05-11/eIahHxke2LfTCtQP.png)
m4->m3
![输入图片说明](/imgs/2024-05-11/eXdxjL0oebR8Jy2z.png)
m4->m1![输入图片说明](/imgs/2024-05-11/IvEXDPP44JBVKoBC.png)

（4）综上，外网之间，内网之间，内外网之间均能通信，网络配置完成。

## 3 Task A
### 3.1

允许ssh进行流量转发：允许来自 192.168.184.0/24 网络、通过ens37接口进入、目标端口为22（SSH）且状态为 NEW（新建立）、ESTABLISHED（已建立）或RELATED（已已建立连接相关的）的TCP数据包，通过ens33接口转发出去；第二条命令同理。
```
sudo iptables -A FORWARD -p tcp --sport 22 -m state --state ESTABLISHED,RELATED -s 192.168.184.0/24 -i ens33 -o ens37 -j ACCEPT

sudo iptables -A FORWARD -p tcp --dport 22 -m state --state NEW,ESTABLISHED,RELATED -s 192.168.184.0/24 -i ens37 -o ens33 -j ACCEPT
```

隐藏内部IP地址
```
sudo iptables -t nat -A POSTROUTING -s 192.168.184.0/24 -o ens33 -j MASQUERADE
```

修改FORWARD链默认策略为DROP（丢弃）并展示iptables规则
```
sudo iptables -P FORWARD DROP

sudo iptables -L
```
![输入图片说明](/imgs/2024-05-11/tN7oRBcAbTki57Ee.png)
### 3.2测试

发现m1 ping不通m4，也不能建立ssh连接。
m1向m4 ping，由于没有找到合适的规则，并且转发策略默认丢弃，因此ping不通。
![输入图片说明](/imgs/2024-05-11/c283IUrtfLZPJxNc.png)

## 4 Task B
### 4.1
删除之前的规则
``` iptables -D FORWARD 1``
修改INPUT和OUTPUT链的默认策略为DROP
![输入图片说明](/imgs/2024-05-11/hRnw4vZz59cqdHbU.png)


```
# iptables -A INPUT -p tcp --dport 22 -i lo -j ACCEPT
# iptables -A INPUT -p tcp --sport 22 -i lo -j ACCEPT
# iptables -A OUTPUT -p tcp --dport 22 -o lo -j ACCEPT
# iptables -A OUTPUT -p tcp --sport 22 -o lo -j ACCEPT
```
允许m2 ping 自身：接收和发送本地回环icmp协议流量。
```
# iptables -A INPUT -p icmp -i lo -j ACCEPT
# iptables -A OUTPUT -p icmp -o lo -j ACCEPT
```


### 4.2 测试
M2可以ping通自己（外网和内网），但是不能ssh![输入图片说明](/imgs/2024-05-11/dHzU5VCuzjuKSkBf.png)连接。

![输入图片说明](/imgs/2024-05-11/tCMRFO7hpiXTAyqM.png)

m2->m1
![输入图片说明](/imgs/2024-05-11/GYsApM1eNZgPvxRL.png)
m4->m2
![输入图片说明](/imgs/2024-05-11/fxjgVi6LBK9K0HXh.png)

## 5 Task C
### 5.1
删除所有的规则
```
# iptables -F
# iptables -L
```

![输入图片说明](/imgs/2024-05-11/czBehycZOFUDjuhh.png)

允许m1向m2发起ssh连接。
```
$ sudo iptables -A OUTPUT -p tcp -s 192.168.184.132 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
$ sudo iptables -A INPUT -p tcp --sport 22 -d 192.168.184.132 -m state --state ESTABLISHED -j ACCEPT```
```

允许m1向外网发起ssh连接,设置FORWARD默认策略为DROP
```
$ sudo iptables -A FORWARD -p tcp --sport 22 -d 192.168.184.132 -m state --state ESTABLISHED -j ACCEPT
$ sudo iptables -A FORWARD -p tcp -s 192.168.184.132 --dport 22 -m state --state NEW,ESTABLISHED -j ACCEPT
sudo iptables -P FORWARD DROP```

![输入图片说明](/imgs/2024-05-11/g8nuG6NARgj5j20D.png)
![输入图片说明](/imgs/2024-05-11/niYy4WqfcJ7QNDgu.png)

### 5.2 测试
m1ssh 连接m4 ，可以成功
![输入图片说明](/imgs/2024-05-11/JoRSBl6zH7NRcBRf.png)
m1ping m4，不能成功
![输入图片说明](/imgs/2024-05-11/8Yt3syDtGD7GwnWD.png)
m3 ssh 连接m4，不能成功。
![输入图片说明](/imgs/2024-05-11/gzw9FYZJ4nwT3tl0.png)
m4->m1也不能成功。

## 6.总结

Iptables命令格式：

``iptables [-t** **表名] [-A 链名] [-p 网络协议名] [-j 规则名] [-s 源ip地址] [-dport 目的端口] [-sport 源端口]``

![输入图片说明](/imgs/2024-05-11/xAZefnnj9UHNI3EC.png)
