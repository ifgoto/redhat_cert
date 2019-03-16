

# 1. 添加磁盘,分区及文件系统

[Cylinder-head-sector](https://en.wikipedia.org/wiki/Cylinder-head-sector)

## 1.1 fdisk

### 此处也可以说明一下2048,2050问题(演示一下)
### fstab
- 再次安利一下vim打开fstab这个文件的时候, 如果写少了一个选项会有个语法检查的
如   
```bash
UUID="da6db982-78ec-4dbc-bd7e-31af4e19c500" /archive            ext4    defaults        0 2
写成了(中间少了个挂载点)
UUID="da6db982-78ec-4dbc-bd7e-31af4e19c500"             ext4    defaults        0 2

```

fstab 第5列 第6列分别为
[fstab详解](https://blog.csdn.net/clozxy/article/details/5603222)
>   fs_dump - 该选项被dump命令使用来检查一个文件系统应该以多快频率进行转储，若不需要转储就设置该字段为0
fs_pass - 该字段被fsck命令用来决定在启动时需要被扫描的文件系统的顺序，根文件系统/对应该字段的值应该为1，其他文件系统应该为2。若该文件系统无需在启动 时扫描则设置该字段为0

### 当在fdisk退格键(backspace)变为^H后,需要按ctrl+退格键才能生效
### 局限性
演于一下fdisk 的局限性.(所以考试时, 如果同一个硬盘中分区数大于4个时,就需要分配逻辑分区之类)
(如只有四个primary, <这个好实现>

```bash
Select (default p):
Using default response p
Partition number (3,4, default 3):
First sector (2149-20971519, default 208896):
Using default value 208896
Last sector, +sectors or +size{K,M,G} (208896-20971519, default 20971519): 1000MB
Value out of range.
Last sector, +sectors or +size{K,M,G} (208896-20971519, default 20971519): +100MB
Partition 3 of type Linux and of size 95 MiB is set

Command (m for help): p

Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9220f3b3

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048        2148          50+  83  Linux
/dev/vdb2            4096      208895      102400   83  Linux
/dev/vdb3          208896      403455       97280   83  Linux

Command (m for help): p

Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9220f3b3

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048        2148          50+  83  Linux
/dev/vdb2            4096      208895      102400   83  Linux
/dev/vdb3          208896      403455       97280   83  Linux

Command (m for help): n
Partition type:
   p   primary (3 primary, 0 extended, 1 free)
   e   extended
Select (default e):
Using default response e
Selected partition 4
First sector (2149-20971519, default 403456): +100MB
Sector 195313 is already allocated
First sector (403456-20971519, default 403456):
Using default value 403456
Last sector, +sectors or +size{K,M,G} (403456-20971519, default 20971519): +1000MB
Partition 4 of type Extended and of size 954 MiB is set

Command (m for help): p

Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9220f3b3

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048        2148          50+  83  Linux
/dev/vdb2            4096      208895      102400   83  Linux
/dev/vdb3          208896      403455       97280   83  Linux
/dev/vdb4          403456     2357247      976896    5  Extended

Command (m for help): n
All primary partitions are in use
Adding logical partition 5
First sector (405504-2357247, default 405504):
Using default value 405504
Last sector, +sectors or +size{K,M,G} (405504-2357247, default 2357247): +100MB
Partition 5 of type Linux and of size 95 MiB is set

Command (m for help): p

Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x9220f3b3

   Device Boot      Start         End      Blocks   Id  System
/dev/vdb1            2048        2148          50+  83  Linux
/dev/vdb2            4096      208895      102400   83  Linux
/dev/vdb3          208896      403455       97280   83  Linux
/dev/vdb4          403456     2357247      976896    5  Extended
/dev/vdb5          405504      600063       97280   83  Linux

```

单分区不能大于2TiB<这个有点难>, 除非结合kvm挂空盘)


### parted
补充除课本外的第三个工具,parted <br>
man parted所得到的信息比infoparted 的信息会少, <br>
有时gnu系列的产品如果用man看到的信息不足时, 要想到info
```bash
parted 
help 

(parted) mkpart p2 xfs 3200MB 3400MB
Warning: The resulting partition is not properly aligned for best performance.
Ignore/Cancel? I
(parted) print list
Model: Virtio Block Device (virtblk)
Disk /dev/vdb: 10.7GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name              Flags
 1      1049kB  2149MB  2147MB  xfs          Linux filesystem
 2      3000MB  3100MB  99.6MB               p1
 3      3200MB  3400MB  200MB                p2


Model: Virtio Block Device (virtblk)
Disk /dev/vda: 10.7GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags:

Number  Start   End     Size    Type     File system  Flags
 1      1049kB  10.7GB  10.7GB  primary  xfs          boot


(parted) quit
Information: You may need to update /etc/fstab.


```

### gparted
当然还可以演示应急盘gparted(这个可以放vm的iso中演示一下)<br>
<此时需要修改vmware的光驱, 挂到iso,并且开机的时候要按F2改为光驱优先启动><br>
对应的windows的分区也是有的[](https://product.pconline.com.cn/itbk/diy/harddisk/1802/10842789.html)

## 1.2 gdisk
gdisk不能列出分区?还是我没有找到合适的选项??
只能用`fdisk -l` ???
````bash
[root@server0 ~]# gdisk -l
GPT fdisk (gdisk) version 0.8.6

Problem opening -l for reading! Error is 2.
The specified file does not exist!
[root@server0 ~]# fdisk -l

Disk /dev/vda: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disk label type: dos
Disk identifier: 0x00013f3e

   Device Boot      Start         End      Blocks   Id  System
/dev/vda1   *        2048    20970332    10484142+  83  Linux

Disk /dev/vdb: 10.7 GB, 10737418240 bytes, 20971520 sectors
Units = sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


````


## 1.3 swap分区的挂载

- 对于挂载的优先级说明
man swapon搜-p
man 2 swapon 
>    Priority
>      Each  swap  area  has  a priority, either high or low.  The default priority is low.  Within the low-priority areas, newer areas are even lower priority
>      than older areas.
>      All priorities set with swapflags are high-priority, higher than default.  They may have any nonnegative value chosen by  the  caller.   Higher  numbers
>      mean higher priority.
>      Swap pages are allocated from areas in priority order, highest priority first.  For areas with different priorities, a higher-priority area is exhausted
>      before using a lower-priority area.  If two or more areas have the same priority, and it is the highest priority available, pages  are  allocated  on  a
>      round-robin basis between them.


## 1.4 学会处理当lab过不了的时候, 要慢慢学会看lab脚本, 并初步具备一些判断力
lab RH134P190 最终lab disk grade时有时会报错, <br>
主要是由于swapon -a时,排序可能会不固定. <br>
在教学的过程中可以演示一下排错.

# 2. LVM

## 2.0 从用户场景引出逻辑卷出现的必要性.
可以从平时我们某个盘的大小不够用.如何扩展???提出,再通过zoomit画图

## 2.1 逻辑卷的基本概念
通过画图解释
- 组成pv的 有可能是本地磁盘,有可能是RAID, 有可能是SAN,
- pv
- vg
- lv
还有一系列的各种命公
XXscan
XXcreate
xxdisplay

## 2.2 创建逻辑卷

- 由于PE是最小单位, 所以考试理论上只要误差不要差太多(理论上是PE的大小),容量上就算是对的.
- 要注意 `lvcreate`时 -L -l大小写的重大区别
- lvcreate时, vg的名字是可以补全出来的

## 2.3 扩展逻辑卷
- lvextent -r可以省去再grow那一步

如果把lvreduce弄得比实际文件小的话, 那么再挂上可能会报错, 把再其lvextent之后好像可以挂上了
```bash
[root@server0 ~]# umount /xx
[root@server0 ~]# mount /dev/ff/mylv /xx
mount: wrong fs type, bad option, bad superblock on /dev/mapper/ff-mylv,
       missing codepage or helper program, or other error

       In some cases useful info is found in syslog - try
       dmesg | tail or so.
[root@server0 ~]# lvextend -n /dev/ff/mylv -L400M
  Extending logical volume mylv to 400.00 MiB
  Logical volume mylv successfully resized
[root@server0 ~]# mount /dev/ff/mylv /xx

```
- xfs 只增不减, ext4可增可减
## 2.4 RAID
- raid0
- raid1
- raid5

有空可以说这个, 虽然考试不用考

# 3. 利用NFS访问其它机器的存储

NOTE:与挂载一起讲的可以说一下平时我们操作的记录输出日志, 那样可以就出问题时,有据可查, 也可以在实验失败之后,<br>
多次重复copy命令省去一些时间(当然最快的是history)

## 3.1 挂载NFS

### 挂载时几个安全参数的区别

- none
- sys
- krb5
- krb5i
- krb5p

### 挂载的三个基本步骤

也可以用`rpcinfo -p localhost`去查一下端口
####  识别`showmount -e  server0`(此时要注意服务器的防火墙是否关了,可用iptable -F进行验证)
```bash
[root@desktop0 ~]# rpcinfo -p server0
rpcinfo: can't contact portmapper: RPC: Remote system error - No route to host
#此时在server0执行iptables -F
[root@desktop0 ~]# rpcinfo -p server0
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100024    1   udp  57267  status
    100024    1   tcp  56235  status
    100005    1   udp  20048  mountd
    100005    1   tcp  20048  mountd
    100005    2   udp  20048  mountd
    100005    2   tcp  20048  mountd
    100005    3   udp  20048  mountd
    100005    3   tcp  20048  mountd
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049  nfs_acl
    100003    3   udp   2049  nfs
    100003    4   udp   2049  nfs
    100227    3   udp   2049  nfs_acl
    100021    1   udp  49010  nlockmgr
    100021    3   udp  49010  nlockmgr
    100021    4   udp  49010  nlockmgr
    100021    1   tcp  41083  nlockmgr
    100021    3   tcp  41083  nlockmgr
    100021    4   tcp  41083  nlockmgr
    100011    1   udp    875  rquotad
    100011    2   udp    875  rquotad
    100011    1   tcp    875  rquotad
    100011    2   tcp    875  rquotad
[root@desktop0 ~]#

```

#### 挂载点 在本机建立相关目录, 注意权限不能太低(mkdir)

#### 挂载(mount) 
+ 手动 mount -t nfs -o sync server0:/share /mountpoint(其中-t并未严格要求, 但-o的话要根据实际情况而定)
+ 自动(写在/etc/fstab)

#### 用umount取消挂载

/etc/mtab 与 mount的输出大体相同
```bash
[root@desktop0 ~]# ls -l /etc/mtab
lrwxrwxrwx. 1 root root 17 May  7  2014 /etc/mtab -> /proc/self/mounts
```
RH134P232实验的时候, 有几个点(如果 /etc/fstab 中/shares/public写成了/share/public<少了个s,也就是路径写错了>会卡死(有时)>)
如果server0不能被showmount -e server0(在desktop0中运行),那么需要在server0用`iptables -F`关掉防火墙(当然也可能关得更细致)

iptables 中的-F解释
```bash
  -F, --flush [chain]
              Flush the selected chain (all the chains in the table if none is given).  This is equivalent to deleting all the rules one by one.

```
如果desktop由于mount卡死了,那么请reset一下desktop,注意,只需要reset, desktop即可(或者有谁有更精准的解决方案也麻烦提出一下)

## 3.2 自动挂载
装了autofs后,可以看出超时值为
````bash
[root@desktop0 ~]# grep -i ^timeout /etc/sysconfig/autofs
TIMEOUT=300

````

RH134P240的 练习中如果ssh  ldapuser0@localhost登陆不成功,可能是key下载时错了.
像我就试过wget时, 把大写的O写成了小写的o
那么本来存 key 的krb5.keytab的内容变成了下面这个样子
````bash
cat krb5.keytab
--2019-03-07 05:07:08--  http://classroom.example.com/pub/keytabs/desktop0.keytab
Resolving classroom.example.com (classroom.example.com)... 172.25.254.254
Connecting to classroom.example.com (classroom.example.com)|172.25.254.254|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1258 (1.2K)
Saving to: ‘desktop0.keytab’

     0K .                                                     100%  101M=0s

2019-03-07 05:07:08 (101 MB/s) - ‘desktop0.keytab’ saved [1258/1258]

````

```bash
遇到这种情况的时候只能reset(根据修改server0:/etc/exportfs, 去掉sec=krb5p之后发现可以挂载, 证实是krb5p的问题)
mount -vv可以看得更清楚
[root@desktop0 ~]# mount -t nfs -o sec=krb5p,rw,sync  server0:/shares/public /mnt/public -vv
mount.nfs: timeout set for Thu Mar  7 07:47:33 2019
mount.nfs: trying text-based options 'sec=krb5p,vers=4,addr=172.25.0.11,clientaddr=172.25.0.10'
mount.nfs: mount(2): Permission denied
mount.nfs: access denied by server while mounting server0:/shares/public


```

这个练习与之前的挂载练习一起做, 有时会出现上面说的permission denied, 这时一般需要重置,再单独做

## 3.3 lab
有一个需要注意的, 这些挂载, 考试时, 服务器都会给你设好, 我们一定是在客户端(desktop)去做的,不要搞错了

# 4. 访问SAMBA共享

与cmd net share一致
## 4.1 访问具体SMB的网络存储


需要安装
cifs-utils
yum -y install samba-client.x86_64


### 三个步骤
- 识别
- 确定挂载点, 并创建挂载点(空目录)
- 挂载网络文件系统.

>识别共享
>	[root@desktop0 ~]# smbclient -L 172.25.0.11 -U user01
>	Enter user01's password: 
>	Domain=[MYGROUP] OS=[Unix] Server=[Samba 4.1.1]
>
>		Sharename       Type      Comment
>		---------       ----      -------
>		rhce            Disk      
>	创建挂载点
>	[root@desktop0 ~]# mkdir /mnt/samba
>
>	挂载
>	[root@desktop0 ~]# mount -o username=user01,password=redhat //172.25.0.11/rhce /mnt/samba/
>
>	[root@desktop0 ~]# tail -1 /etc/fstab
>	//172.25.0.11/rhce	/mnt/samba	cifs	defaults,username=user01,password=redhat	0 0
>
>	写不了
>
>	[root@desktop0 samba]# touch 1.txt
>	touch: cannot touch ‘1.txt’: Permission denied
>
>	原因：
>	（1）写权限？
>
>	[root@server0 ~]# ll -d /samba/
>	drwxr-xrwx. 2 root root 18 Aug  6 16:31 /samba/
>	[root@server0 ~]# 
>	（2）SELinux？
>	[root@server0 ~]# chcon -R -t samba_share_t /sa>mba
>	[root@server0 ~]# 
>	[root@server0 ~]# getenforce 
>	Enforcing
>	[root@desktop0 samba]# touch 3.txt
>

## 4.2 RH134P246练习

`credentials=/secure/student.smb`
考试时,credentials这个单词应能写出

## 4.3 LAB RH134P248

这种挂载的实验一般是开头先要重置desktop,server,之后运行lab XXX setup,最后在desktop中lab XXX grade

注意, 自动挂载前,可以先手动挂载验证, 

自动挂载文件中 为://server0,而手动挂载时, 不用":"

### 一些疑问
对于那些对接的挂载文件不一定非要放在/etc/auto.master.d/下面,也可以直接写在/etc/auto.master, 而且需要在课堂上演示一下, 修改需要通过
`systemctl restart autofs`重启服务才行

也可以显式地指定密码. smbclient -L ip -U username%password

