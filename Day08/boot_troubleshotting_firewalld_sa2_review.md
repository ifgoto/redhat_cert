plan
![](res/schedule.png)

# 1. 启动过程中的故障排查

## 1.1 ???


## 1.2 RHEL7启动过程

- BIOS/UEFI (玩技术也存在站队,,当然,如果把原理给学过去后, 也有会转得比较快, 也弄到一个广度,深度的治学问题的怪圈中)
作为两种技术, 新旧交替, 此时可以发散对于 

- IPv4/ipv6 

- grub/lilo

> [What is the difference between LILO and GRUB?](https://unix.stackexchange.com/questions/6498/what-is-the-difference-between-lilo-and-grub)
>LILO has a simpler interface and is easier to wrap your head around.
>GRUB is more featured and handles odd configurations better.
>The LILO bootstrap process involves locating the kernel by in essence (it's more complicated than this) pointing to the first logical-sector of the Kernel file. The GRUB bootstrap process is more filesystem aware and can locate a kernel file in a filesystem without having to specify a logical-sector.
>There is a reason nearly everyone is using GRUB these days, and that's because it's less fragile and handles edge-cases better.

### 1.2.1 加电自检(POST)

### 1.2.2 搜索启动设备
- (此时可以先给vm做一些镜像, 之后做实验, 让学员们去调整启动顺序等)
- MBR的概念也可以在这里进行介绍
- 对于VMware模拟的bios可以按F2进入bios,也可以用ESC键临时调整启动顺序??
[VMware workstation设置开机引导等待时间的一个小技巧](https://blog.csdn.net/solaraceboy/article/details/80286930)
关闭vmware后, 打开虚拟机的vmx文件在后面加入 bios.bootDelay = "20000" 表示停20秒
<br>
(再次打开,可能这行就不在最后面,需要搜索)

### 1.2.3 系统固件读取磁盘中的boot loader
在RHEL7,bootloader是grub2 

### 1.2.4 grub从磁盘加载其它东西

- /etc/grub.d
- /etc/default/grub
- /boot/grub2/grub.cfg

### 1.2.5 与用户交互(或超时)

- initramfs是经过gzip的cpio归档,其中包含启动时所必要的硬件(内核模块,初始化脚本), 在rhel7中initramfs包含了整个系统.

### 1.2.6 grub把控制权交给了initramfs
配置同1.2.4, 但是不同的阶段

### 1.2.7  initramfs-->systemd
```bash
[kiosk@foundation0 ~]$ ls -lrt /sbin/init
lrwxrwxrwx. 1 root root 22 Jan 23  2016 /sbin/init -> ../lib/systemd/systemd

```

### 1.2.8 initrd.target /sysroot的真正root

`/etc/fstab`

### 1.2.9 root归位

### 1.2.10 根据默认的target选择相应的选项(按需,并行启动)

## 1.3 启动, 重启, 关闭

正确的姿势应是
systemctl reboot 之类,但为了向下兼容, 所以就保留了一部分

`````bash
[kiosk@foundation0 ~]$ ls -l $(which reboot)
lrwxrwxrwx. 1 root root 16 Jan 23  2016 /usr/sbin/reboot -> ../bin/systemctl
[kiosk@foundation0 ~]$ ls -l $(which poweroff)
lrwxrwxrwx. 1 root root 16 Jan 23  2016 /usr/sbin/poweroff -> ../bin/systemctl
[kiosk@foundation0 ~]$ ls -l $(which halt)
lrwxrwxrwx. 1 root root 16 Jan 23  2016 /usr/sbin/halt -> ../bin/systemctl
[kiosk@foundation0 ~]$
`````
note: halt不直接关机,还要手动来一下

## 1.4 各个启动级别

各个级别所依赖是不同的,一般从上到下减下
````bash
[kiosk@foundation0 ~]$ systemctl list-dependencies  graphical.target |grep target
graphical.target
└─multi-user.target
  ├─basic.target
  │ ├─paths.target
  │ ├─slices.target
  │ ├─sockets.target
  │ ├─sysinit.target
  │ │ ├─cryptsetup.target
  │ │ ├─local-fs.target
  │ │ └─swap.target
  │ └─timers.target
  ├─getty.target
  └─remote-fs.target
[kiosk@foundation0 ~]$ systemctl list-dependencies  multi-user.target |grep target
multi-user.target
├─basic.target
│ ├─paths.target
│ ├─slices.target
│ ├─sockets.target
│ ├─sysinit.target
│ │ ├─cryptsetup.target
│ │ ├─local-fs.target
│ │ └─swap.target
│ └─timers.target
├─getty.target
└─remote-fs.target
[kiosk@foundation0 ~]$ systemctl list-dependencies  rescue.target |grep target
rescue.target
└─sysinit.target
  ├─cryptsetup.target
  ├─local-fs.target
  └─swap.target
[kiosk@foundation0 ~]$ systemctl list-dependencies  emergency.target |grep target
emergency.target
[kiosk@foundation0 ~]$
````
通过 `systemctl isolate `进行切换
如在graphic切到multi-user  就可以用`systemctl isolate multi-user`<br>

(这个命令在foundation时灵时不灵, 在server中进行实验倒是挺灵的,,,<br>
其实也与之前的init 5(startx 图型界面), init 4(字符界面) init 6重启, init 0 关闭)

### 1.4.1 默认的启动目标
之前在搭环境时,就列出过相关, 现在正式来讲一下
````bash
#别名了之后不会了补全, 也挺让人心烦的,,不知如何是好
[kiosk@foundation0 ~]$ sctl get-default
multi-user.target
[kiosk@foundation0 ~]$ systemctl get-default
multi-user.target

systemctl set-default XXX.target

````
当然也可能在启动时, 在引导内核所在行(一般以linux16开始),的最后面加上`systemd.unit=rescue.target``去临时改变启动的目标.
其实也可以直接在行未加空格后,写上emergency进入该模式,


另外,由于7有不少常用的系统命令已进行了不少的更改, 下面的网页可以列出修改
[Common administrative commands in Red Hat Enterprise Linux 5, 6, and 7](https://access.redhat.com/articles/1189123)

## 1.5 root密码恢复

### 不能直接进行单用户模式了

之前直接在内核中加空格, 加小写s(或数字1)进行单用户模式那招在rhel7已失效了

可按书本上修改
(如不remount,会发现/sysroot/etc/shadow下面是都没有权限的,)

但有一个细节, 如果发现前面有`rhgb quiet` rd.break加入前, 请把上面的字眼删掉,以防万一,
另外如不加入rd.break,那么可以加入 `init=/bin/sh`试一下,如果考试遇上失败的重置场景的话
有时还不行的话再加入`console=tty1`

下面是一系列参考链接
[RHEL 7 Root Password Recovery](https://access.redhat.com/discussions/1243493?tour=8)
[12 Steps to Password Recovery for RHEL, CentOS 7 Linux](https://spr.com/password-recovery-for-rhel-centos-7-linux/)
[How to recover forgotten root password](https://rhel7tutorial.wordpress.com/how-to-recover-forgotten-root-password/)




