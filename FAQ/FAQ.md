# 环境类
## 在desktop server中运行lab相关命令报`unable to download``


![](res/lab_cannot_download.png)


如上图,这种情况请检查classroom是不是没有启动.
<br>
可用此命令
```bash
rht-vmctl status classroom
```

>你可以看一下classroom是不是正常的,(用以下命令)
>在foundation0中打开个终端输入
>
`rht-vmctl status classroom`
>
>  2019/3/1 21:59:32
>如果未启动可以用下面的命令启动(报missing)
>rht-vmctl start classroom
>
>  2019/3/1 22:00:59
>如果已启动, 那么可能出了点小问题,建议重置desktop server
<br>
`rht-vmctl reset desktop`
<br>
`rht-vmctl reset server`
>
>如果还是不行把classroom也重置了,
>
<br>
`rht-vmctl reset classroom`
<br>

>
>  2019/3/1 22:01:07
>如果还不行,,那我再想想其它办法

## 总复习的字母不清
ch16   的总复习键入的是
`lab sa1-review setup`  注意是数字1而不是字母l

## 在server或desktop中运行rht-vmctl命令
这两个命令是在foundation0中运行的, 在这两个机器中运行会无效的.

## RH124P136 ricky可以删除lfile1 lfile2
有同学对此条有疑问, 理论上只要他对这个目录有w权限,他就可以删掉
<br>
下面的例子进行说明

```bash
 for USER in user{1..4}; do useradd ${USER};done
[root@server0 ~]# mkdir testGroupOwner
[root@server0 ~]# ls
a  anaconda-ks.cfg  b  testGroupOwner  testUGO
[root@server0 ~]# groupadd group1
[root@server0 ~]# usermod -aG group1 user1
[root@server0 ~]# usermod -aG group1 user3
[root@server0 ~]# chown user1:group1 testGroupOwner/
[root@server0 ~]# chmod 775 testGroupOwner/
[root@server0 ~]# cd testGroupOwner/
[root@server0 testGroupOwner]# touch lfile1
[root@server0 testGroupOwner]# chown user2:user2 lfile1
[root@server0 testGroupOwner]# touch lfile2
[root@server0 testGroupOwner]# chown user2:group1 lfile2
[root@server0 testGroupOwner]# ll
total 0
-rw-r--r--. 1 user2 user2  0 Mar  3 12:34 lfile1
-rw-r--r--. 1 user2 group1 0 Mar  3 12:34 lfile2
[root@server0 testGroupOwner]# chmod g+w lfile1
[root@server0 testGroupOwner]# chmod o+w lfile2
[root@server0 testGroupOwner]# ll
total 0
-rw-rw-r--. 1 user2 user2  0 Mar  3 12:34 lfile1
-rw-r--rw-. 1 user2 group1 0 Mar  3 12:34 lfile2
[root@server0 ~]# mv testGroupOwner /tmp
[root@server0 testGroupOwner]# sudo -i -u user1
[user1@server0 ~]$ id
uid=1008(user1) gid=1008(user1) groups=1008(user1),10502(group1) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
[user1@server0 ~]$ cd /tmp
[user1@server0 tmp]$ ls -lrt testGroupOwner/
total 0
-rw-rw-r--. 1 user2 user2  0 Mar  3 12:34 lfile1
-rw-r--rw-. 1 user2 group1 0 Mar  3 12:34 lfile2
[user1@server0 tmp]$ ls -lrtd testGroupOwner/
drwxrwxr-x. 2 user1 group1 32 Mar  3 12:34 testGroupOwner/
[user1@server0 tmp]$ cd testGroupOwner/
[user1@server0 testGroupOwner]$ rm lfile1 lfile2
rm: remove write-protected regular empty file ‘lfile1’? y
rm: remove write-protected regular empty file ‘lfile2’? y
[user1@server0 testGroupOwner]$ ls
[user1@server0 testGroupOwner]$

```


