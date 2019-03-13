

# 1. 添加磁盘,分区及文件系统

## 1.1 fdisk
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


