To lab or not to lab, that is the question.

参考：
[配置snortIDS并创建规则](https://cn.linux-console.net/?p=17541)
[实验 snort安装配置与NIDS规则编写](https://www.cnblogs.com/lasgalen/p/4512755.html)
[开源入侵检测系统—Snort的配置与检测规则编写](https://blog.csdn.net/negnegil/article/details/120836558)
[snort的安装配置和使用](https://blog.csdn.net/qq_37865996/article/details/85088090?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522162331041516780366540025%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162331041516780366540025&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-85088090.first_rank_v2_pc_rank_v29&utm_term=snort&spm=1018.2226.3001.4187)

## 1. Lab Overview
No required material. You may refer to materials on the internet.
your tasks：
1. Installing Snort. You can use any Linux flavor you want or windows platform.

2. Writing Snort Rules

You run snort in the NIDS mode.

Now you write snore rules in the following two scenarios, respectively
(1) Write a snore rule that will display an alert whenever it detects that your computer is accessed by other computers via HTTP protocol. Note that your Apache server is working at TCP port 80.
写一个snort规则，当它检测到你的电脑被其他的电脑通过HTTP协议访问时，会发出一个报警。你的Apache server 在TCP 80端口。
(2) Write a snore rule that will display an alert when someone logins your computer as root outside your computer via any method (such as ftp, or ssh , or telenet).
写一个snort规则，当有人在除了你的电脑之外的地方通过任意协议使用管理员权限登录时，会发出一个警报。
The report must include:
studentID
For each scenario, you describe the steps and give snapshot of the snort rule and alert.


## 2.Tasks 

### 2.0 Environment

主机1：ubuntu 32bit 观察机 192.168.188.133
主机2：ubuntu 32bit 攻击机


**1、安装依赖包**
```
apt-get install flex
apt-get install bison
apt-get install libpcap-dev
```
获取DAQ源码包。
```
wget https://www.snort.org/downloads/snort/daq-2.0.7.tar.gz
tar xvzf daq-2.0.7.tar.gz
cd daq-2.0.7
./configure && make && sudo make install
```

**2.安装snort依赖包以及snort包**
```
 apt-get install libpcre3-dev
 apt-get install libdumbnet-dev
```
```
wget https://www.snort.org/downloads/snort/snort-2.9.20.tar.gz
tar xvzf snort-2.9.20.tar.gz
cd snort-2.9.20
./configure --disable-open-appid && make && sudo make install
```

**3.配置snort**
```
#创建snort目录
mkdir /etc/snort
mkdir /etc/snort/rules
mkdir /etc/snort/rules/iplists
mkdir /etc/snort/preproc_rules
mkdir /usr/local/lib/snort_dynamicrules
mkdir /etc/snort/so_rules


#创建储存规则文件
touch /etc/snort/rules/iplists/black_list.rules
touch /etc/snort/rules/iplists/white_list.rules
touch /etc/snort/rules/local.rules
touch /etc/snort/sid-msg.map

#创建日志目录
mkdir /var/log/snort
mkdir /var/log/snort/archived_logs


#修改文件权限
chmod -R 5775 /etc/snort
chmod -R 5775 /var/log/snort
chmod -R 5775 /var/log/snort/archived_logs
chmod -R 5775 /etc/snort/so_rules
chmod -R 5775 /usr/local/lib/snort_dynamicrules


#修改文件属主
chown -R snort:snort /etc/snort
chown -R snort:snort /var/log/snort
chown -R snort:snort /usr/local/lib/snort_dynamicrules


#将配置文件从源文件复制到/etc/snort/中
cd /snort-2.9.18.1/etc/    # (进入snort安装目录，每个人可能不同)

cp *.conf* /etc/snort
cp *.map /etc/snort
cp *.dtd /etc/snort

cd /root/snort-2.9.18.1/src/dynamic-preprocessors/build/usr/local/lib/snort_dynamicpreprocessor
cp * /usr/local/lib/snort_dynamicpreprocessor/
```

**4.编辑snort配置文件**

注释掉Snort导入默认规则文件集的行
使用PulledPork管理规则集注释掉snort.conf中引用的规则文件：`sed -i "s/include \$RULE\_PATH/#include \$RULE\_PATH/" /etc/snort/snort.conf`

用“ip addr”查看ubuntu4.1的网卡信息，得到ip地址为aaa.aaa.aaa.aaa/24

在当前目录下使用gedit命令修改snort.conf配置，
(1)将HOME_NET替换为本机（aaa.aaa.aaa.0/24）
```
Setup the network addresses you are protecting
ipvar HOME_NET aaa.aaa.aaa.0/24 #45行左右
```
(2)修改conf配置路径
```
var RULE_PATH /etc/snort/rules           # 104行左右  
var SO_RULE_PATH /etc/snort/so_rules        # 105行左右  
var PREPROC_RULE_PATH /etc/snort/preproc_rules   # 106行左右  
```
（3）注释掉黑白名单的过滤规则。
```
#var WHITE_LIST_PATH ../rules    #119行左右
#var BLACK_LIST_PATH ../rules
```

**5.测试snort**
使用主机2ping主机1，并在主机1启动snort。


### 2.1 Task1 

**1.编写规则**
在ubuntu4.1中进入root用户，并进入根目录的/etc/snort/rules在local.rules文件中写入我们自定义的规则。**这条 Snort 规则用于检测 TCP 流量中与目标 IP 地址为 192.168.188.133，目标端口为 80 的通信。**
```
cd /etc/snort/rules
gedit local.rules
```
```
 alert tcp ![192.168.188.133/32] any -> 192.168.188.133/32 80 (logto:”task1”; msg:”this is task 1”; sid:1000001)
 ```


 - snort规则分为两部分，括号之前是规则头，括号内是规则选项部分。规则头包含规则的动作、协议、源和目标ip地址与网络掩码，以及源和目标端口信息；规则选项部分包含报警消息内容和要检查的包的具体部分。

 - alert 表示这是一个警告。tcp表示要检测所有使用tcp协议的包，因为http协议是tcp/ip协议的一部分。接下来的一部分表示源IP地址，其中！表示除了后面IP的所有IP，因此! 192.168.188.133/32表示的就是除了本机之外的所有主机。再后面的一个表示端口，any表示源IP地址任何一个端口，也就是说源IP地址的主机不管哪个端口发送的包都会被检测。->表示检测的包的传送方向，表示从源IP传向目的IP。下面的一个字段表示目的IP，在这里表示主机。后面的字段表示端口号，经过查阅相关资料，80端口在winxp中作为IIS对主机的访问端口。

 - 括号中的规则选项部分，logto表示将产生的信息记录到文件，msg表示在屏幕上打印一个信息，sid表示一个规则编号，如果不在规则中编写这个编号，则执行过程中会出错，而且这个编号是唯一的能够标识一个规则的凭证，1000000以上用于用户自行编写的规则。

 - 综合起来，这条规则的意思是：当检测到从除了目标 IP 地址是 192.168.188.133 外的任何地址到 192.168.188.133 的 TCP 流量，目标端口是 80 时，生成一个警报，日志名称为 "task1"，并显示消息 "this is task 1"。
 
**2. 编写网站**
安装apache.
` apt-get install apache2`
验证apache运行状态
`sudo systemctl status apache2`
`echo "task1111" > /var/www/html/index.html`
运行snort，用主机2访问主机1 ip，可以看到报错，有异常网络数据包但是没有对应的策略。


### 2.2 TASK2
**1.编写规则** 

重新写入规则，它们的作用是检测来自除了目标 IP 地址是 192.168.188.133 外的任何源地址到目标地址是 192.168.188.133 的流量，并生成一个警报，记录日志到名为 "task2" 的位置，同时显示消息 "this is task 2"。
```
alert tcp ![192.168.188.133/32] any -> 192.168.188.133/32 any (logto:”task2”;  msg:”this is task 2”; sid:1000002)
alert udp ![192.168.188.133/32] any -> 192.168.188.133/32 any (logto:”task2”;  msg:”this is task 2”; sid:1000003)
alert icmp ![192.168.188.133/32] any -> 192.168.188.133/32 any (logto:”task2”;  msg:”this is task 2”; sid:1000004)
alert ip ![192.168.188.133/32] any -> 192.168.188.133/32 any (logto:”task2”;  msg:”this is task 2”; sid:1000005)
```
**2.ssh连接**
使用ssh连接主机1
` ssh 192.168.188.133`
在主机1观察到报警。


## 3 实验总结
Snort 是一个用于网络监控的开源入侵检测系统 (IDS)。通过对 Snort 的实验，可以学习到网络流量监测与分析的基本原理，掌握规则编写与优化技巧，提高对网络安全威胁的识别能力。实验还培养了快速响应与处理安全事件的能力，加深对网络攻击手段和防御策略的理解。

