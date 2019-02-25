plan
![](res/schedule.png)

# 1. 配置,加固openSSH服务

## 1.1 通过SSH远程登陆

下面部分内容摘自维基百科[SSH中文相关页面](https://zh.wikipedia.org/wiki/Secure_Shell)

### 名字由来
SSH是(Secure SHell)的缩写.

Secure Shell（安全外壳协议，简称SSH）是一种加密的网络传输协议，可在不安全的网络中为网络服务提供安全的传输环境。<br>
SSH通过在网络中创建安全隧道来实现SSH客户端与服务器之间的连接。虽然任何网络服务都可以通过SSH实现安全传输，SSH最常见的用途是远程登录系统，<br>
人们通常利用SSH来传输命令行界面和远程执行命令。使用频率最高的场合类Unix系统，但是Windows操作系统也能有限度地使用SSH。<br>
2015年，微软宣布将在未来的操作系统中提供原生SSH协议支持.

### 替代品
在设计上，SSH是Telnet和非安全shell的替代品。Telnet和Berkeley rlogin、rsh、rexec等协议采用明文传输，使用不可靠的密码，<br>
容易遭到监听、嗅探和中间人攻击。SSH旨在保证非安全网络环境（例如互联网）中信息加密完整可靠。

### 没有绝对的安全
不过，SSH也被指出有被嗅探甚至解密的漏洞。早在2011年，中国的互联网审查机构已经有能力针对SSH连线的刺探及干扰。<br>
而后爱德华·斯诺登泄露的文件也指出，美国国家安全局有时能够把SSH协议传输的信息解密出来，从而读出SSH会话的传输内容。<br>
2017年7月6日，非营利组织维基解密确认美国中央情报局已经开发出能够在Windows或Linux操作系统中窃取SSH会话的工具。

### 通过ssh远程登陆

有时可能我们的实验环境不是干净的, 因此, 为了得到相一致的效果, 建议我们在实验前先把我们的 desktop, server都重置一下,<br>
如对我们环境 foundation, desktop, server还不太清楚, 可以查看[相关环境搭建的章节](../preparation/preparation.md)<br>

#### 登陆前常见错误
desktop 机器都未启动我们就想去登陆,,一般会出现下面的报错, 我们可以用ping这个检测网络是否连通的命令去验证(当然,如有防火墙把icmp包给丢了就另当别论)
```bash
[kiosk@foundation0 ~]$ ssh student@172.25.0.10
ssh: connect to host 172.25.0.10 port 22: No route to host
[kiosk@foundation0 ~]$ ping 172.25.0.10
PING 172.25.0.10 (172.25.0.10) 56(84) bytes of data.
From 172.25.0.250 icmp_seq=1 Destination Host Unreachable
From 172.25.0.250 icmp_seq=2 Destination Host Unreachable
From 172.25.0.250 icmp_seq=3 Destination Host Unreachable
From 172.25.0.250 icmp_seq=4 Destination Host Unreachable
^C
--- 172.25.0.10 ping statistics ---
6 packets transmitted, 0 received, +4 errors, 100% packet loss, time 5000ms
pipe 4
[kiosk@foundation0 ~]$

```
当然,此时如果查看desktop这虚拟机的状态也是处于Missing中
````bash
[kiosk@foundation0 ~]$ rht-vmctl status desktop
error: failed to connect to the hypervisor
error: no valid connection
error: Failed to connect socket to '/var/run/libvirt/libvirt-sock': No such file or directory
desktop MISSING
````
#### 重置环境
同时把desktop, server给重置了
```bash
[kiosk@foundation0 ~]$ rht-vmctl reset desktop
Are you sure you want to reset desktop? (y/n) y
Powering off desktop.
Resetting desktop.
Creating virtual machine disk overlay for rh124-desktop-vda.qcow2
Creating virtual machine disk overlay for rh124-desktop-vdb.qcow2
Starting desktop.
[kiosk@foundation0 ~]$ rht-vmctl reset server
Are you sure you want to reset server? (y/n) y
Powering off server.
Resetting server.
Creating virtual machine disk overlay for rh124-server-vda.qcow2
Creating virtual machine disk overlay for rh124-server-vdb.qcow2
Starting server.
[kiosk@foundation0 ~]$

```

由于重置后会默认把被重置的机器拉起, 稍等片刻后, 两机器应处于运行状态.
````bash
[kiosk@foundation0 ~]$ ping 172.25.0.10
PING 172.25.0.10 (172.25.0.10) 56(84) bytes of data.
64 bytes from 172.25.0.10: icmp_seq=1 ttl=64 time=1.45 ms
^[k64 bytes from 172.25.0.10: icmp_seq=2 ttl=64 time=0.600 ms
64 bytes from 172.25.0.10: icmp_seq=3 ttl=64 time=1.16 ms
^C
--- 172.25.0.10 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 2002ms
rtt min/avg/max/mdev = 0.600/1.072/1.453/0.354 ms
[kiosk@foundation0 ~]$ ping 172.25.0.11
PING 172.25.0.11 (172.25.0.11) 56(84) bytes of data.
64 bytes from 172.25.0.11: icmp_seq=1 ttl=64 time=1.30 ms
64 bytes from 172.25.0.11: icmp_seq=2 ttl=64 time=2.73 ms
^C
--- 172.25.0.11 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1002ms
rtt min/avg/max/mdev = 1.303/2.018/2.734/0.716 ms
````

此时我时我们先登陆到desktop中再从desktop用ssh登到server中,(因为之前在foundation中,已进行了一些设置(我们下面会详细说明),
<br>
因此我们直接登陆到了desktop中去了.)


```bash
[kiosk@foundation0 ~]$ ssh student@172.25.0.10
[student@desktop0 ~]$
```

在desktop中我们再正常演示一下平时我们的登陆的情况
注:两机器的用户名与密码均为student
```bash
[student@desktop0 ~]$ ssh student@172.25.0.11
The authenticity of host '172.25.0.11 (172.25.0.11)' can't be established.
ECDSA key fingerprint is eb:24:0e:07:96:26:b1:04:c2:37:0c:78:2d:bc:b0:08.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.25.0.11' (ECDSA) to the list of known hosts.
student@172.25.0.11's password:
[student@server0 ~]$
```

此时我们可以在新开的另一个终端窗口(下面简称B窗)中foundation登上server,之后用`w -f`命令查看,确认有用户远程登陆到server中去了
```bash
[student@server0 ~]$ w -f
 22:21:48 up 14 min,  3 users,  load average: 0.04, 0.05, 0.08
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
student  pts/0    desktop0.example 22:15    2:12   0.03s  0.03s -bash
student  pts/1    172.25.0.250     22:21    4.00s  0.04s  0.01s w -f
```
从FROM中可以看到 pts/0 是从desktop0.example过来的,如果你真的不确认, 
此时我们可以在之前从desktop登陆到server的那个窗口(下面简称A窗)sleep一下
```bash
[student@server0 ~]$ sleep 1000
```

之后再回到从foundation登陆到server的那个窗口, 再次运行`w -f`此时发现之前的-bash命令, 变成了sleep,<br>
 那么对于那个登陆者就是从desktop过来的这个事实就更确认无疑了.
```bash

[student@server0 ~]$ w -f
 22:24:36 up 17 min,  3 users,  load average: 0.00, 0.03, 0.07
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
student  pts/0    desktop0.example 22:15   12.00s  0.03s  0.00s sleep 1000
student  pts/1    172.25.0.250     22:21    4.00s  0.05s  0.02s w -f
```

### 远程登陆后运行完命令就退出
之前是登陆后, 就一直在远程机器,进行交换式操作, 但有时我们到远程机器只是需要进行简单的操作,<br>
因此, 我们可以在原来登陆后交互操作的命令后面加入需要在远程操作的命令后就会有新的效果, 如下:

例如, 我们可以在A窗运行,写入desktop到此一游
```bash
[student@desktop0 ~]$ ssh student@172.25.0.11 'echo "desktop had beed here">~/notice'
student@172.25.0.11's password:
```
注意, 'echo "desktop had beed here">~/notice' 作为ssh登陆后的一个整体命令, 需要用单引号引住, 不然至此一游的文件还是在本地.切记




<br>
此时回到B窗确认一下是不是真的有一个叫notice的文件,里面写着到此一游的..
```bash
[student@server0 ~]$ cat notice
desktop had beed here
```
的确如此

### ssh的known_hosts

如果不是第一次登陆到新远端机器中, 就不会有以下提示及应答

```bash 
The authenticity of host '172.25.0.11 (172.25.0.11)' can't be established.
ECDSA key fingerprint is eb:24:0e:07:96:26:b1:04:c2:37:0c:78:2d:bc:b0:08.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.25.0.11' (ECDSA) to the list of known hosts.
```
那是因为自动第一次ssh登陆到远程机器中后, 客户端(desktop,就会把server的公钥,及远程机器存起来)
如下面 在A窗运行

```bash
[student@desktop0 ~]$ cat .ssh/known_hosts
172.25.0.11 ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHX+o9KAnlfw2dE7CsmM4hqfv1udM79a5NWC2BuWlmfKSwfYLptPQMJF8bnqaz0EjDlxCxRu/aito+GphPLzp/k=
```

在B窗确认一下server的公钥
```bash
[student@server0 ~]$ cat /etc/ssh/ssh_host_ecdsa_key.pub
ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBHX+o9KAnlfw2dE7CsmM4hqfv1udM79a5NWC2BuWlmfKSwfYLptPQMJF8bnqaz0EjDlxCxRu/aito+GphPLzp/k=
[student@server0 ~]$
```

我们把A窗的中.ssh/known_hosts中相应的记录删去(当然,更极端的话, 就不只是删掉某条记录, 而是整个文件给删去)
再去登陆的话, 之前提示会再次出现, yes后, 登进去, 再退出,再登陆就不会再提示了.


## 1.2 以密钥形式登陆
在上一小节我们演示了, 通过用户名与密码登陆到远程的机器中去.
而ssh其实还是可以通过密钥登陆的.

下面就演示在desktop,生成密钥后,传输密钥后再不用输入密码远程登陆到server

先在desktop生成公钥及私钥.
```bash
[student@desktop0 ~]$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/student/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/student/.ssh/id_rsa.
Your public key has been saved in /home/student/.ssh/id_rsa.pub.
The key fingerprint is:
6c:47:97:6c:c4:d4:6d:ec:d7:d5:11:4d:76:ae:d3:c8 student@desktop0.example.com
The key's randomart image is:
+--[ RSA 2048]----+
|           oo. *O|
|           o...oO|
|          . =  o+|
|       . . o . ++|
|        S .   E o|
|       . .     . |
|                 |
|                 |
|                 |
+-----------------+
```
把公钥传到要登陆的服务器中
```bash
[student@desktop0 ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub student@172.25.0.11
/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
student@172.25.0.11's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'student@172.25.0.11'"
and check to make sure that only the key(s) you wanted were added.
```


之后就可以免密登陆了
```bash
[student@desktop0 ~]$ ssh student@172.25.0.11
Last login: Mon Feb 25 23:41:57 2019 from desktop0.example.com
[student@server0 ~]$
```

如果我们在生成key的时候,用了密码保存,那么登陆时,需要不用输入student的密码,可是还是要输入保护key的密码,那样还是要交换,要手动输入, 不太方便, 
<br>
但是"安全".

#### 如果私钥用密码进行保护后, 还是需要输入密码, 只不过是另一个密码罢了.
可以这样重复实验
```bash
[student@desktop0 .ssh]$ rm id_rsa id_rsa.pub
```
再重复生成密钥, 此时用密码保护私钥, 那么就算是传输密钥之后, 我们登陆还是会提示
```bash
[student@desktop0 ~]$ ssh student@172.25.0.11
Enter passphrase for key '/home/student/.ssh/id_rsa':
Last failed login: Mon Feb 25 23:40:53 CST 2019 from desktop0.example.com on ssh:notty
There were 2 failed login attempts since the last successful login.
Last login: Mon Feb 25 23:07:41 2019 from desktop0.example.com
[student@server0 ~]$ exit
```

## 1.3 修改sshd相关配置后,限制登陆
注意是
/etc/ssh/sshd_config
不是
/etc/ssh/ssh_config

```bash
[root@server0 ssh]# egrep "^PermitRoot|^Password" sshd_config
PermitRootLogin no
PasswordAuthentication no
```
在B窗
修改 PasswordAuthentication 为no后,
再重启sshd服务
```bash
[root@server0 ssh]# systemctl restart sshd
```

再切回A窗, 发现已不能用密码登陆了
(当然由于之前的student用户已设置了密钥登陆, 因此我们需要临时改立一个新的用户去验证这个事)
```bash
[kk@desktop0 ~]$ ssh student@172.25.0.11
The authenticity of host '172.25.0.11 (172.25.0.11)' can't be established.
ECDSA key fingerprint is eb:24:0e:07:96:26:b1:04:c2:37:0c:78:2d:bc:b0:08.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '172.25.0.11' (ECDSA) to the list of known hosts.
Permission denied (publickey,gssapi-keyex,gssapi-with-mic).
[kk@desktop0 ~]$ exit
```

同理, 修改了
PermitRootLogin no
就会让root不能直接ssh登陆.



# 2. 分析,存储日志

## 系统日志架构

各种系统日志
![](res/overview_of_system_log_files.png)

<br>

## 查看系统日志文件
![](res/overview_of_syslog_priorities.png)

监控安全方面的日志文件
在B窗打开安全方面的日志文件(用 tail -f 打开实时查看变化)

```bash
[root@server0 log]# tail -f /var/log/secure
Feb 25 10:58:21 localhost unix_chkpwd[31087]: password check failed for user (root)
Feb 25 10:58:21 localhost sshd[31085]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=desktop0.example.com  user=root
Feb 25 10:58:21 localhost sshd[31085]: pam_succeed_if(sshd:auth): requirement "uid >= 1000" not met by user "root"
Feb 25 10:58:23 localhost sshd[31085]: Failed password for root from 172.25.0.10 port 36440 ssh2
Feb 25 10:58:45 localhost sshd[31085]: Connection closed by 172.25.0.10 [preauth]
Feb 25 10:59:03 localhost sshd[31083]: Received signal 15; terminating.
Feb 25 10:59:03 localhost sshd[31103]: Server listening on 0.0.0.0 port 22.
Feb 25 10:59:03 localhost sshd[31103]: Server listening on :: port 22.
Feb 25 10:59:07 localhost sshd[31106]: Accepted password for root from 172.25.0.10 port 36441 ssh2
Feb 25 10:59:07 localhost sshd[31106]: pam_unix(sshd:session): session opened for user root by (uid=0)
...

随着A窗root登陆到server,  出错会打出下面的信息, 最后一次为密码正确的登陆


Feb 25 11:10:30 localhost sshd[31106]: Received disconnect from 172.25.0.10: 11: disconnected by user
Feb 25 11:10:30 localhost sshd[31106]: pam_unix(sshd:session): session closed for user root
Feb 25 11:10:38 localhost unix_chkpwd[31350]: password check failed for user (root)
Feb 25 11:10:38 localhost sshd[31348]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=desktop0.example.com  user=root
Feb 25 11:10:38 localhost sshd[31348]: pam_succeed_if(sshd:auth): requirement "uid >= 1000" not met by user "root"
Feb 25 11:10:40 localhost sshd[31348]: Failed password for root from 172.25.0.10 port 36442 ssh2
Feb 25 11:10:46 localhost unix_chkpwd[31351]: password check failed for user (root)
Feb 25 11:10:46 localhost sshd[31348]: pam_succeed_if(sshd:auth): requirement "uid >= 1000" not met by user "root"
Feb 25 11:10:48 localhost sshd[31348]: Failed password for root from 172.25.0.10 port 36442 ssh2
Feb 25 11:10:48 localhost sshd[31348]: Connection closed by 172.25.0.10 [preauth]
Feb 25 11:10:48 localhost sshd[31348]: PAM 1 more authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=desktop0.example.com  user=root
Feb 25 11:10:52 localhost sshd[31360]: Accepted password for root from 172.25.0.10 port 36443 ssh2
Feb 25 11:10:52 localhost sshd[31360]: pam_unix(sshd:session): session opened for user root by (uid=0)
^C
```

### logrotate
当日志过多时, 是超量归类,还是固定几个文件重复写入?
总之可以避免单个日志过大, 或者尽可能避免因日志导致磁盘满了.

### 系统日志相关格式
![](res/syslog_format.png)

### 给系统日志发消息
```bash
[root@server0 log]# logger -p local7.notice "test system log"
[root@server0 log]# cd /var/log
[root@server0 log]# grep -r "test system log" *
boot.log:Feb 25 11:35:30 localhost student: test system log
messages:Feb 25 11:35:30 localhost student: test system log

```
用这个命令重新系统日志服务
```bash
[root@server0 log]# systemctl restart rsyslog
````

RH124P220 练习


## 查看systemd日志条目

### 通过journalctl 查找事件

`journalctl` 这样列出太多, 可以加入下面的限制更精确去查看日志.

-n 可看最近十条, -n 5 最后五条, -p err,可以看err级别的日志

```bash
[root@server0 log]# journalctl -n
-- Logs begin at Mon 2019-02-25 22:07:35 CST, end at Tue 2019-02-26 00:40:01 CST.
Feb 26 00:30:01 server0.example.com CROND[31589]: (root) CMD (/usr/lib64/sa/sa1 1
Feb 26 00:32:58 server0.example.com student[31670]: sadfasfj
Feb 26 00:32:59 server0.example.com student[31670]: asdfasdf
Feb 26 00:35:30 server0.example.com student[31690]: test system log
Feb 26 00:38:02 server0.example.com systemd[1]: Stopping System Logging Service..
Feb 26 00:38:02 server0.example.com systemd[1]: Starting System Logging Service..
Feb 26 00:38:02 server0.example.com systemd[1]: Started System Logging Service.
Feb 26 00:40:01 server0.example.com systemd[1]: Starting Session 34 of user root.
Feb 26 00:40:01 server0.example.com systemd[1]: Started Session 34 of user root.
Feb 26 00:40:01 server0.example.com CROND[31775]: (root) CMD (/usr/lib64/sa/sa1 1
[root@server0 log]# journalctl -n 5
-- Logs begin at Mon 2019-02-25 22:07:35 CST, end at Tue 2019-02-26 00:40:01 CST.
Feb 26 00:38:02 server0.example.com systemd[1]: Starting System Logging Service..
Feb 26 00:38:02 server0.example.com systemd[1]: Started System Logging Service.
Feb 26 00:40:01 server0.example.com systemd[1]: Starting Session 34 of user root.
Feb 26 00:40:01 server0.example.com systemd[1]: Started Session 34 of user root.
Feb 26 00:40:01 server0.example.com CROND[31775]: (root) CMD (/usr/lib64/sa/sa1 1
lines 1-6/6 (END)
[root@server0 log]# journalctl -p err
-- Logs begin at Mon 2019-02-25 22:07:35 CST, end at Tue 2019-02-26 00:40:01 CST.
Feb 25 22:07:36 localhost kernel: Failed to access perfctr msr (MSR c1 is 0)
Feb 25 22:07:37 localhost rpcbind[171]: rpcbind terminating on signal. Restart wi
Feb 25 22:07:43 localhost smartd[504]: Problem creating device name scan list
Feb 25 22:07:43 localhost smartd[504]: In the system's table of devices NO device
Feb 25 22:07:49 localhost systemd[1]: Failed to start LSB: Starts the Spacewalk D
Feb 25 22:07:50 localhost libvirtd[1031]: libvirt version: 1.1.1, package: 29.el7
Feb 25 22:07:50 localhost libvirtd[1031]: Module /usr/lib64/libvirt/connection-dr
Feb 25 22:07:53 server0.example.com systemd[1]: Failed to start /etc/rc.d/rc.loca
Feb 25 23:54:06 server0.example.com systemd[1]: Failed to mark scope session-25.s
lines 1-10/10 (END)
```

-f 类似 tail -f 可以实时动态输出

--since --until 作为时间范围查找

也可 
```bash
journalctl _SYSTEMD_UNIT=sshd.service _PID=1182
```
_UID=..
_PID=..
这样更精确地列出

### 永久转存systemd日志
```bash
[root@server0 log]# mkdir  /var/log/journal
[root@server0 log]# chown root:systemd-journal /var/log/journal/
[root@server0 log]# chmod 2755 /var/log/journal
[root@server0 log]# killall -USR1 systemd-journald
[root@server0 log]# ls /var/log/journal/
946cb0e817ea4adb916183df8c4fc817
[root@server0 log]# ls /var/log/journal/946cb0e817ea4adb916183df8c4fc817/
system.journal
[root@server0 log]#

```

### 保存精确的时间

#### 设置本地时间及时区

```bash

[root@server0 log]# timedatectl
      Local time: Tue 2019-02-26 00:52:03 CST
  Universal time: Mon 2019-02-25 16:52:03 UTC
        RTC time: Mon 2019-02-25 16:52:04
        Timezone: Asia/Shanghai (CST, +0800)
     NTP enabled: yes
NTP synchronized: no
 RTC in local TZ: no
      DST active: n/a

```

```bash
[root@server0 log]# timedatectl list-timezones
Africa/Abidjan
Africa/Accra
Africa/Addis_Ababa
Africa/Algiers
Africa/Asmara
Africa/Bamako
Africa/Bangui
Africa/Banjul
...




[root@server0 log]# timedatectl set-timezone Asia/Shanghai
[root@server0 log]# timedatectl set-time 0:55
[root@server0 log]# timedatectl set-ntp true


```

#### 配置和监控chronyd

修改 /etc/chrony.conf
```bash

[root@server0 etc]# head chrony.conf
# Use public servers from the pool.ntp.org project.
# Please consider joining the pool (http://www.pool.ntp.org/join.html).
server classroom.example.com iburst

# Ignore stratum in source selection.
stratumweight 0

# Record the rate at which the system clock gains/losses time.
driftfile /var/lib/chrony/drift

```
重启服务并对时

```bash


[root@server0 etc]# systemctl restart chronyd.service


[root@server0 etc]# chronyc sources -v
210 Number of sources = 1

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current synced, '+' = combined , '-' = not combined,
| /   '?' = unreachable, 'x' = time may be in error, '~' = time too variable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||                                                /   xxxx = adjusted offset,
||         Log2(Polling interval) -.             |    yyyy = measured offset,
||                                  \            |    zzzz = estimated error.
||                                   |           |
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* classroom.example.com         8   6    17    30    -21us[ -104us] +/- 1439us
[root@server0 etc]#


```


