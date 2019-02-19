# plan
|上课时间|课程模块|课程内容|重点项目案例|
|:---|:---|:---|:---|
|上午|访问命令行| 本地访问控制台<br> 在桌面上运行命令行<br> 在bash shell中执行命令行| 实战案例1:本地控制台访问权限术语<br> 实战案例2:用GNOME 3桌面环境<br> 实战案例3:命令和键盘快捷方式<br> 实战案例4:运行命令行<br> |


# 访问命令行

## 本地访问控制台(Accessing the Command Line Using the Local Console)
### 拿什么来管理您,我的OS
早上的考试, 内容在RH124,RH134, 说人话,就是大家手上的第一二册, 两本书. 
<br>
如果大家不小心通过了,会有一张RHCSA的证书
<br>
![](res/RHCSA.png)
<br>
惨啦, 一不小心暴露了姓名,反正自我绍也忘了,正好,
<br>我叫朱XX,在IT这行中混了十多年,大家可以叫我老朱.
毕竟在IT界中,,见面就给你贴个工程师,叫X工的, 我被叫`朱工`时,总有时会被联想到"公猪",之类..不太好.<br>
so, just call me 老朱, pls.
<br>
RHCSA是(Red Hat Certified System Administrator), 也就是系统管理员.

<br>
那么我们凭什么去管理系统呢?不能每次都这样子吧.(现在再看来也挺佛系的)
<br>

![](res/pray.png)
<br>
我们需要的不仅仅是双手高举过头,
<br>
而更需要的是双手好好地去键盘上,熟练地,有时甚至是机械地去键入一个个运维指令,去管理系统.
<br>

![](res/stroke-key-and-the-screen.png)
唉..这个图没有选好, 请不要关注前面的屏幕,,,
<br>
我们这个教室不是在教大家写代码, 教大家 javascript21天从入门到放弃
<br>
后面那个对焦不准的才是敲命令的...
<br>所以..那招用力按鼠标左键可能在我们这边不太好使.
<br>
`好吧..我就是来教大家敲命令的^_^`

### hello world
什么叫传统,就是大家差不多都这么做的事情,
<br>
例如电影开拍,开机时, 要上个烧猪拜上几拜.
<br>又例如, IT界中什么什么大都从`hello world`开始,
<br>据说一退休码农在家中学书法,一开始就用毛笔写了1024个楷书的`hello world`.

<br>
因此我们也不能免俗,不用敲1024个,一个即可.

#### echo hello world
```bash
[kiosk@foundation0 ~]$ echo "hello world"
hello world
[kiosk@foundation0 ~]$

```

> 思考题<br>
此处可以给大家留个思考题, 上面这里用双引号及用单引号有没有区别, 在这种命令环境中单双引号有什么区别?

同时为了纪念这个这么有仪式感的时刻,我又迫不及地打个卡

```bash
[kiosk@foundation0 ~]$ date
Wed Feb 20 08:25:18 CST 2019
```

#### usage中括号及尖括号等说明
可是我的英文不太好, 上面的时间看得不太明白.

于是想换个输出的格式, 但人老了,,记性比英文更差, 于是就要求助了

> note:下面的输出有所选择地输出.

```bash
[kiosk@foundation0 ~]$ date --help
Usage: date [OPTION]... [+FORMAT]
  or:  date [-u|--utc|--universal] [MMDDhhmm[[CC]YY][.ss]]
Display the current time in the given FORMAT, or set the system date.
...$a)
FORMAT controls the output.  Interpreted sequences are:
...
  %d   day of month (e.g., 01)
...
  %m   month (01..12)
...
  %T   time; same as %H:%M:%S
...
  %Y   year
....

Examples:
Convert seconds since the epoch (1970-01-01 UTC) to a date
  $ date --date='@2147483647'
  ...

For complete documentation, run: info coreutils 'date invocation'

```

上面usage中,一般国际惯例是, 
- []中是可选择的,,,可加可不加, 虽然有时加了与不加差异挺大的查看之后,把命令调整为
- <>中的文本表示变量数据,  如<filename>表示此处插入您要使用的文件名.
```bash
[kiosk@foundation0 ~]$ date '+%Y%m%d %T'
20190220 08:28:50
[kiosk@foundation0 ~]$

```
这样看我就比较清楚时间了.


#### 变成root修改时间
上面时间的格式是合我意了, 但好像时间不太正确
<br>
(可能是虚拟机没有设置好, 或者时区没设好, 关于`时区的设置`, 可以参看后继课程)
<br>修改时间, 不是随随便便的人也能改的, 要改也只有随便起来不是人的root可以修改.
<br>用下面命令切换到root用户,关于`用户`这个概念后继再详细讲解,暂时你可以理解就一个个马甲好了.

```bash
[kiosk@foundation0 ~]$ su -
Password:
Last login: Wed Feb 20 08:52:59 CST 2019 on pts/1
[root@foundation0 ~]#
[root@foundation0 ~]#
```
最后的提示符从原来的一般用户的`$`变成了root的`#`, 这个是比较大的区别,
另外也可以从提示符中看到有差别
```bash
[root@foundation0 ~]# echo $PS1
[\u@\h \W]\$
```
上面环境变量PS1就表示这个提示符由\u用户名, \h主机名 \W当前工作目录 组成
<br>
什么?是环境变量?什么是主机名?当前工作目录又是什么鬼???好吧...后面细说现在只能这么说.

<br>
总之就是当你是root时, 会尽可能地让你感到你不再是随随便便的人了..于是给你换个提示符,
<br>
生产上有些机器不光只换提示符的最后一位, 也会随之把提示符的颜色也改成了红色,以示警戒.

<br>
好吧既然是root了, 就改个时间吧, 改快一分左右


TODO: 修改时间的方法好像还有其它的, 可以再找找其它的写法
```bash
[root@foundation0 ~]# date
Wed Feb 20 01:01:15 CST 2019
[root@foundation0 ~]# date 0220010319
Wed Feb 20 01:03:00 CST 2019
[root@foundation0 ~]# date
Wed Feb 20 01:03:01 CST 2019
```
### 命令的三大组成部分

### 124P5的 practise需要演示一下


## 在桌面上运行命令行




