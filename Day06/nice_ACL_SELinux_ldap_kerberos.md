plan
![](res/schedule.png)

# 1. 管理Linux进程优先级

## 1.1 优先级和"nice"概念

### Linux进程调度和多任务

#### 插一个nohup的演示

```firefox &``` 这样运行后 ps 能看到当前firefox,把当前这个terminal关闭后, <br>
firefox也会被关闭<br>
当然,为了闭免这个问题, 我们可以用<br>
`nohup firefox &`去运行.<br>

`ps aux`
a mean all 
u mean user
x mean extension

当然也可以换成
ps eux 会更详细

当然上面的是bsd语法, 也可以用标准语法
ps -ef



```
#也可以用下面的方法去取特定的列
[kiosk@foundation0 ~]$ ps au
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root       797  0.5  0.5 207360 40124 tty1     Ssl+ 21:59   0:11 /usr/bin/Xorg :0 -background none -verbose -auth /run/gdm/auth-for-gdm-epzdFq/database -seat seat0 -nolisten tcp vt1
kiosk     2282  0.0  0.0 116256  2840 pts/2    Ss+  22:21   0:00 bash
kiosk     3062  0.5  0.0 116136  2796 pts/1    Ss   22:36   0:00 -bash
kiosk     3120  0.0  0.0 123356  1368 pts/1    R+   22:36   0:00 ps au
[kiosk@foundation0 ~]$ ps -o %cpu
%CPU
 0.2
 0.0

[kiosk@foundation0 ~]$ ps exf -o pid,%cpu,comm --no-header
 3060  0.0 sshd
 3062  0.0  \_ bash
 3151  0.0      \_ ps
 1058  0.0 gnome-session
 1154  0.0  \_ ssh-agent
 1276  0.0  \_ gnome-settings-
 1410  5.7  \_ gnome-shell
 1583  0.0  \_ seapplet
 1599  0.0  \_ tracker-miner-f
 1609  0.0  \_ rhsm-icon
 1618  0.0  \_ abrt-applet
 2203  0.1 gnome-terminal-
 2206  0.0  \_ gnome-pty-helpe
 2282  0.0  \_ bash
 1643  0.0 gvfsd-metadata
 1572  0.0 gvfsd-trash
 1555  0.0 tracker-store
 1550  0.0 gconfd-2
 1541  0.0 evolution-addre
 1537  0.0 evolution-calen
 1512  0.0 nautilus
 1479  0.0 evolution-sourc
 1473  0.0 mission-control
 1471  0.0 gnome-shell-cal
 1432  0.0 pulseaudio
 1420  0.0 gsd-printer
 1407  0.0 gvfs-gphoto2-vo
 1399  0.0 goa-daemon
 1396  0.0 gvfs-goa-volume
 1392  0.0 gvfs-mtp-volume
 1387  0.0 gvfs-afc-volume
 1377  0.0 gvfs-udisks2-vo
 1364  0.0 dconf-service
 1291  0.0 gnome-keyring-d
 1275  0.0 spice-vdagent
 1272  0.0 gvfsd-fuse
 1251  0.0 gvfsd
 1233  0.0 at-spi2-registr
 1221  0.0 at-spi-bus-laun
 1229  0.0  \_ dbus-daemon
 1143  0.0 dbus-daemon
 1107  0.0 dbus-launch

```

```
也可以用下面的方法查找某个程序的pid
[kiosk@foundation0 ~]$ pidof bash
3062 2282 754
[kiosk@foundation0 ~]$ man pidof
[kiosk@foundation0 ~]$ echo $$
3062
```

```bash

nice -n 13 firefox 2>&1 >/dev/null &

kiosk@foundation0 ~]$ pstree $(echo $$)
bash─┬─firefox───39*[{firefox}]
     └─pstree

```

总的来说, 我们这个只是一个分时系统.(据说有一些实时系统用于航天,CPU主频很低,很慢,但这个系统很皮厚,能应付各种太空中的射线)<br>

也就是我们实际运行的线程数远远大于我们现在cpu core的数量. 不可能"一心一意",因此就有了进程调试.
<br>
当然,就进程调试来说,作为一整门操作系统的大课来说也是说不完的,我们在此仅仅是通过一些实验,来窥探一下linux系统的冰山一角而己.
<br>

### 相对优先级
注意,我们说的优先级仅仅是在同一个时点,我们需要的core的数量小于我们进程需要core的数量,
<br>
进程与进程之间存在竞争关系,这个时候,优先级的意义才体现出来.

打个不太恰当的比方就是, 我们整个操作系统就是一个到火车站买票的过程, 如果平时我们买票的人多于火车站买票的窗口,那么就必须在各窗口中<br>
进行排队,那么假设每个窗口是一样的(事实上不是如此),而排队的过程有点混乱,如个一个人比较强状,那么他的优先级会比较高,那么他可能从排队中<br>
更容易往前挤,,但是,,,如果某个时间点,,,有10个窗口,但只有五个人买票,那么这个时候是否强状其实都不太管用...因为这个时候根本不用抢....
而操作系统很很空闲时,就有点这个情况...因此讨论这个优先级时,也不能太太绝对.

<br>
当然,大多数时候还是需要排队的,而事实上,不一定是强状就可以的(也就是不一定nice值高就OK的)就如我们平时排队买票的时候也有分军人专列,<br>
老弱病残孕专列(当然,,会有人说自己脑残之后钻了空子去这个地方)....对应操作系统就是有一些系统级的相关进程生来就是优先级高,本身就是能得到<br>
更多的系统资源的.


几种调度策略
```bash
man sched_setscheduler
...
Currently, Linux supports the following "normal" (i.e., non-real-time) scheduling policies:

       SCHED_OTHER   the standard round-robin time-sharing policy;

       SCHED_BATCH   for "batch" style execution of processes; and

       SCHED_IDLE    for running very low priority background jobs.

       The  following  "real-time"  policies  are  also  supported, for special time-critical applications that need precise control over the way in which
       runnable processes are selected for execution:

       SCHED_FIFO    a first-in, first-out policy; and

       SCHED_RR      a round-robin policy.

       The semantics of each of these policies are detailed below.

....

```

进程的真实优先级= 进程的初始优先级+nice值

而优行级值越小, 分到CPU的分片就越多,

普通用户设置的nice值只能是正的,也就是nice值越来越大,也就是只能放弃自己的CPU...
而只有root用户才可以把nice值设小那么才有可能真正地提高优先级

其实初始优先级还是可以修改的(
```
chrt 1 cat /dev/zero >/dev/null
```
)

### 分时系统的一些说明
就举个例子我们听音乐, 其实播放音乐的进程不用一直用着CPU只是需要在每隔一段时间用一下, 当然,这个时间比起我们正常人以0.1秒左右的感知说,
<br>已经很长了.但类似[视觉暂留](http://kingdarling.blogspot.com/2013/02/blog-post_4013.html)(海狮加皮球,),听觉其实也有类似的.因此我们进程在一边放音乐,一边在处理着后台的一系列事情.<br>
所以如果有CPU高占时我们能感觉到卡是从鼠标不跟手.声音断断续续这些各种表像进行(当然,也可以从CPU风扇的声音及温度来说).


## 用nice 设定优先级并启动程序

设定这个值是单向的,, 不能让这个进程得到更多的时间片(不然就需要内核程序去维护一个进程超初优先级的列表)
```bash
[kiosk@foundation0 ~]$ nice -n 13 firefox &
[1] 4409
[kiosk@foundation0 ~]$ renice -n 10 $(pidof firefox)
[kiosk@foundation0 ~]$ renice -n 15 $(pidof firefox)
4409 (process ID) old priority 13, new priority 15
[kiosk@foundation0 ~]$ renice -n 14 $(pidof firefox)
renice: failed to set priority for 4409 (process ID): Permission denied

ps axo pid,comm,nice --sort=nice

```

上面这些都是静态的, 我们可以用top去动态显示(当然还有图型界面那种)
另外用top也是可以的,按r
同时也可以按h演于一下各种功能



## 发现进程优先级实验
用`lscpu`可以看到cpu的个数
也可以用 `grep -c '^processor' /proc/cpuinfo`

## 关闭某个cpu
据说在
`/sys/devices/system/cpu/cpu0`
中加入offline文件从而关闭某个cpu
与此相对的还有cpu亲和性设置

## 利用top及renice可以看一下进程对cpu的占用
cat /dev/zero > /dev/null &
cat /dev/zero > /dev/null &

renice -n 15 -p 1653


这些实验及练习都很有必要做, 一个个去做吧 


# 2.使用访问控制列表(ACL)控制对文件的访问

## 2.1 ACL(访问控制列表)

ext4对ACL的支持需要额外的挂载选项.

### 举一个传统UGO不能实列的例子

```bash
 for USER in user{1..4}; do useradd ${USER};done
 [root@server0 tmp]# mkdir ugo
[root@server0 tmp]# chown user1:user1 ugo
[root@server0 tmp]# chmod 751 ugo
[root@server0 tmp]# ls -ld ugo
drwxr-x--x. 2 user1 user1 6 Mar  3 12:15 ugo
[root@server0 tmp]# usermod -aG user1 user2
[root@server0 tmp]# grep '^user1' /etc/group
user1:x:1008:user2
[root@server0 tmp]#
[root@server0 tmp]# setfacl -m u:user4:- ugo

 
```

测试
````bash
[kiosk@foundation0 ~]$ ssh root@server0
root@server0's password:
Last login: Sun Mar  3 11:53:52 2019 from 172.25.0.250
[root@server0 ~]# sudo -i -u user4
[user4@server0 ~]$ cd /tmp/ugo
-bash: cd: /tmp/ugo: Permission denied
[user4@server0 ~]$
[user4@server0 ~]$ logout
[root@server0 ~]# sudo -i -u user3
[user3@server0 ~]$ cd /tmp/ugo
[user3@server0 ugo]$ ls
ls: cannot open directory .: Permission denied
[user3@server0 ugo]$ exit
logout
[root@server0 ~]# sudo -i -u user2
[user2@server0 ~]$ cd /tmp/ugo
[user2@server0 ugo]$ ls
efg
[user2@server0 ugo]$ echo "abc">mmk
-bash: mmk: Permission denied
[user2@server0 ugo]$ umask
0002
[root@server0 ~]# sudo -i -u user1
[user1@server0 ~]$ cd /tmp/ugo/
[user1@server0 ugo]$ ls
[user1@server0 ugo]$ echo "mmq">efg
[user1@server0 ugo]$ ll
total 4
-rw-rw-r--. 1 user1 user1 4 Mar  3 12:20 efg
[user1@server0 ugo]$ ls -lrt
total 4
-rw-rw-r--. 1 user1 user1 4 Mar  3 12:20 efg
[user1@server0 ugo]$ logout

````
### 常规演示
`````
[student@desktop0 ~]$ mkdir tmp2
[student@desktop0 ~]$ getfacl tmp2                                                                                                                                                                          
# file: tmp2
# owner: student                                                                                                                                                                                            
# group: student
user::rwx                                                                                                                                                                                                   
group::rwx
other::r-x                                                                                                                                                                                                  

[student@desktop0 ~]$ ls -ld tmp2                                                                                                                                                                           
drwxrwxr-x. 2 student student 6 5��  31 23:09 tmp2
[student@desktop0 ~]$ man setfacl                                                                                                                                                                           
[student@desktop0 ~]$ sudo useradd u3
[sudo] password for student: 
[student@desktop0 ~]$ setfacl -m u:u3:rwx tmp2
[student@desktop0 ~]$ ll
total 20
-rw-------. 1 root    root    8619 5��  29 22:10 anaconda-ks.cfg
-rw-r--r--. 1 root    root    1981 5��  29 22:09 kickstart.cfg
drwxrwxr-x. 2 student student   26 5��  29 23:02 tmp
drwxrwxr-x+ 2 student student    6 5��  31 23:09 tmp2
[student@desktop0 ~]$ getfacl tmp2
# file: tmp2
# owner: student
# group: student
user::rwx
user:u3:rwx
group::rwx
mask::rwx
other::r-x

`````
-m d 是默认权限
-x d 是取消默认权限, 对已有的权限不受影响

### setfacl用了getfacl的输出
```bash
[root@server0 ~]# touch a
[root@server0 ~]# touch b
[root@server0 ~]# getfacl a|setfacl --set-file=- b
```



### 优先级的一个示例
当给某个用户设置某个特定权限时, 他就不再属于other了,也不属于group,
<br>
但onwer例外, 还是可以走owner的路线
<br>
可以在/tmp/tmp2 加个文件abc测试一下, 加个用户u3




# 3.管理SELinux安全性

说文解字SELinux(Sercurity Enhanced Linux)
在原来DAC的基础上多加了一层,MAC..sandbox

## 3.1 启用并监控SELinux
- 简单地理解为一个安全层, 在原来的基制被突破后,还有另一道防线.
- 一系列安全规制的集合. 
- 安全上下文(context)
- 安全标签(四类:用户,角色,类型,敏感度)
可从下面的输出看出几个类型
(u,r,t,s)
现在一系列命令的选项中加入大写(有些会小写)Z后查得与SELinux相关的东西
```bash
[kiosk@foundation0 ~]$ ls -Z
drwxrwxr-x. kiosk kiosk unconfined_u:object_r:user_home_t:s0 a
-rw-rw-r--. kiosk kiosk unconfined_u:object_r:user_home_t:s0 ClassPrep.txt
-rw-rw-r--. kiosk kiosk unconfined_u:object_r:user_home_t:s0 ClassroomReset.txt

```

我们把级别改为targeted之后, type(类型)这个应用得最多

### 临时启动或停用SElinux
- setenforce
man setenforce
```bash
setenforce(8)                                                                   SELinux Command Line documentation                                                                  setenforce(8)

NAME
       setenforce - modify the mode SELinux is running in

SYNOPSIS
       setenforce [Enforcing|Permissive|1|0]

DESCRIPTION
       Use Enforcing or 1 to put SELinux in enforcing mode.
       Use Permissive or 0 to put SELinux in permissive mode.

       If SELinux is disabled and you want to enable it, or SELinux is enabled and you want to disable it, please see selinux(8).

AUTHOR
       Dan Walsh, <dwalsh@redhat.com>

SEE ALSO
       selinux(8), getenforce(8), selinuxenabled(8)

dwalsh@redhat.com                                                                          7 April 2004                                                                             setenforce(8)

```
- getenforce
man getenforce是我见过一个相对比较简单的man

### 永久改变SELinux
```bash
[root@server0 ~]# cat /etc/selinux/config

# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=enforcing
# SELINUXTYPE= can take one of these two values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted

```

```bash
man selinux_config

...
    If the config file is missing or corrupt, then no SELinux policy is loaded (i.e. SELinux is disabled).
....

```

### SELinux 开关
可以查看man,再结合讲解
```bash
 getsebool -a
 setsebool 
```

RH134P128中,,vim /etc/selinux/config中修改为permissive时, 可以用vim的补全技巧,,可以演示一下
其中的grep也可以复习一下grep相应的正则表达式.


## 更改 SELinux上下文

### 初始SELinux上下文

如果 ls /var/www 报错说没有该目录,请安装httpd
```bash
[root@server0 ~]# ls /var/www
ls: cannot access /var/www: No such file or directory

[root@server0 ~]# yum -y install httpd
```

下面的例子可以说明, 上下文一般随着创建时的目录, 但有一些命令在操作的过程中会改变(如mv, cp -a)
<此处可以扩展一下>
man cp 找-a的用法,,,,

![](res/cp-a_manual.png)

原来此部分被隐藏作为作业的一部分

```bash

[root@server0 ~]# ls -Z /var/www
drwxr-xr-x. root root system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 html
[root@server0 ~]# ls -Z ~
-rw-------. root root system_u:object_r:admin_home_t:s0 anaconda-ks.cfg
[root@server0 ~]#
[root@server0 ~]# cd ~
[root@server0 ~]# touch abc
[root@server0 ~]# ls -Z ~
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 abc
-rw-------. root root system_u:object_r:admin_home_t:s0 anaconda-ks.cfg
[root@server0 ~]# cp abc /var/www/abc_cp
[root@server0 ~]# mv abc /var/www/abc_mv
[root@server0 ~]# cd /var/www
[root@server0 www]# touch efg
[root@server0 www]# ls -Z /var/www
-rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 abc_cp
-rw-r--r--. root root unconfined_u:object_r:admin_home_t:s0 abc_mv
drwxr-xr-x. root root system_u:object_r:httpd_sys_script_exec_t:s0 cgi-bin
-rw-r--r--. root root unconfined_u:object_r:httpd_sys_content_t:s0 efg
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 html

```
### 更改SELinux上下文
- chcon (我们经常加-t)
- restorecon (-v)(-Rv) (推荐用这个)

此时可以安利moba_xterm的透明属性

以下是反面例子
```bash
[root@server0 ~]# mkdir /virtual
[root@server0 ~]# ls -Zd /virtual/
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /virtual/
[root@server0 ~]# chcon  -t httpd_sys_content_t /virtual
[root@server0 ~]# ls -Zd /virtual/
drwxr-xr-x. root root unconfined_u:object_r:httpd_sys_content_t:s0 /virtual/
[root@server0 ~]# restorecon -v /virtual/
restorecon reset /virtual context unconfined_u:object_r:httpd_sys_content_t:s0->unconfined_u:object_r:default_t:s0
[root@server0 ~]# ls -Zd /virtual/
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /virtual/

```

### 定义SELinux默认上下文规则

- semanage
    - -l
    - -a -t httpd_sys_content_t

其中'(/.*)?'(可选的)匹配/后跟任意数量的字符.
以下是正面例子

```bash
[root@server0 ~]# rmdir -p /virtual
[root@server0 ~]# mkdir -p /virtual
[root@server0 ~]# touch /virtual/index.html
[root@server0 ~]# ls -Zd /virtual/
drwxr-xr-x. root root unconfined_u:object_r:default_t:s0 /virtual/
[root@server0 ~]# ls -Z /virtual/
-rw-r--r--. root root unconfined_u:object_r:default_t:s0 index.html
[root@server0 ~]# semanage fcontext -a -t httpd_sys_content_t '/virtual(/.*)?'
[root@server0 ~]# restorecon -RFvv /virtual
restorecon reset /virtual context unconfined_u:object_r:default_t:s0->system_u:object_r:httpd_sys_content_t:s0
restorecon reset /virtual/index.html context unconfined_u:object_r:default_t:s0->system_u:object_r:httpd_sys_content_t:s0
[root@server0 ~]# ls -Zd /virtual/
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /virtual/
[root@server0 ~]# ls -Z /virtual/
-rw-r--r--. root root system_u:object_r:httpd_sys_content_t:s0 index.html
[root@server0 ~]#

```

### RH134P125 更改SELinux上下文练习
- (输入/custom时可以ctrl+x ctrl+f 路径补全)
- (该实验中打开server0的firefox需要在foundation中 ssh -X 登陆)
- (如果一时间httpd_sys_content_t这个类型记不住,可以semanage fcontext -l|grep /var/wwww来借以查看类型)
- (如果对httpd.conf相关配置文件的配置完成后,可以用`httpd -t  -f httpd.conf`这个命令对配置文件进行语法检测)
- (另外也可以不用firefox这个图型界面,我们可以用curl <url>得出我们需要请求的文件,如 curl http://localhost/file3.html)



### 更改SELinux的逻辑布尔值

#### getsebool
- getsebool -a (列出所有)
- getsebool httpd_enable_homedirs(只列出其中一个)

#### setsebool

#### semanage boolean -l -C (列出修改过的值(也就是与默认值不同的配置))


## 对SElinux进行故障排查

- 排查的4个步骤,可以参考RH134P136
- 没事可看日志
    - /var/log/audit/audit.log
    - /var/log/messages
    
- sealert -l <UUID> 可查出对应错误所应采取的具体操作措施
    - -l <UUID> 可查出对应错误所应采取的具体操作措施
    - -b 打开浏览器显示
    - -a 分析文件(可能会有点慢)
    - -f (自动修复)(未试过)







# 4. 连接到网络定义的用户和组

## 4.1 使用身份管理服务

###  回顾一下本地登陆的过程

#### /etc/passwd

#### /etc/group

#### /etc/shadow

### 集中式身份管理的需求

- 相同,相似或相似身份的用户, 管理N台机器
(如果是公用的, 是不是N台机器都要建N个用户???还是有其它解决方案)

结合现在desktop server两台机器都有student用户这个例子举例说明

- SSO(Single Sign On单点登录)

### 用authconfig-tui演示

#### 如报libssn_ldap.so不存在
![](res/need_libnss_ldap.so.png)

### RH134 P156-157练习由于yum install的包不够老是会做失败

下面给出我在字符界面的做法

先安装比书本写得要多的包
````bash
yum -y install pam_krb5 nss-pam-ldapd sssd authconfig-tui  krb5-workstation.x86_64 nss.i686

````

根据书本信息在tui中进行填写, 并在最后一页时, 会提示您下载证书,

![](res/download_cert.png)

此时切换到另外一个终端,用wget下载证书

(注意,未到最后一步, 这个目录不会被创建)
```bash
cd /etc/openldap/cacerts
wget http://classroom.example.com/pub/example-ca.crt

```

 确认是否创建成功
````bash
getent passwd ldapuser0
````


需要提前创建 
```bash
mkdir -p /home/guests/ldapuser0
chown -R ldapuser0:ldapuser0 /home/guests/ldapuser0
```


此进可运行
```bash
ssh ldapuser0@localhost

密码为
kerberos

```
## 4.2 将系统连接到IPA服务器
IPA(identify, policy, audit) 身份,策略, 审核

主要利用客户端
ipa-client进行安装
(可以按课后的lab进行演示)

RH134P161
--no-ntp这个参数好像没有生效

````bash
 sudo ipa-client-install -N --domain=server0.example.com --mkhomedir
Discovery was successful!
Hostname: desktop0.example.com
Realm: SERVER0.EXAMPLE.COM
DNS Domain: server0.example.com
IPA Server: server0.example.com
BaseDN: dc=server0,dc=example,dc=com

Continue to configure the system with these values? [no]: yes
User authorized to enroll computers: admin
Synchronizing time with KDC...
Unable to sync time with IPA NTP server, assuming the time is in sync. Please check that 123 UDP port is opened.


````

另外初次登陆时也没有提示原密码为password, 要更改密码的说法,与原书有所出入.

## 4.3  加入Active Directory
(加入域后, 可以用域用户进行域登陆)

## 红帽官方参考资料

[SYSTEM-LEVEL AUTHENTICATION GUIDE](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/openldap)

[LINUX DOMAIN IDENTITY, AUTHENTICATION, AND POLICY GUIDE](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html-single/linux_domain_identity_authentication_and_policy_guide/)

