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

