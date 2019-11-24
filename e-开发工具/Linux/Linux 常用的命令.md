# Linux 常用的命令

## 关于内存CPU的命令

### free命令详解

free 命令显示系统内存的使用情况，包括物理内存、交换内存(swap)和内核缓冲区内存。
![Snipaste_2019-11-24_11-20-31.jpg](https://i.loli.net/2019/11/24/Hru7Wy5qGJZk1UE.jpg)
如果加上 -h 选项，输出的结果会友好很多，
![Snipaste_2019-11-24_11-21-19.jpg](https://i.loli.net/2019/11/24/QbtST1mhjaUqzsA.jpg)

有时我们需要持续的观察内存的状况，此时可以使用 -s 选项并指定间隔的秒数：

`free -h -s 3`

![Snipaste_2019-11-24_11-21-19.jpg](https://i.loli.net/2019/11/24/QbtST1mhjaUqzsA.jpg)

**参数解释**：

**Mem** 行是内存的使用情况

**Swap** 行是交换空间的使用情况

**total** 列显示系统总的可用物理内存和交换空间大小

**used** 列显示已经被使用的物理内存和交换空间

**free** 列显示还有多少物理内存和交换空间可用使用

**shared** 列显示被共享使用的物理内存

**buff/cache** 列显示被 buffer 和 cache 使用的物理内存大小

### ps 命令

ps命令是Process Status的缩写。ps命令用来列出系统中当前运行的那些进程。ps命令列出的是当前那些进程的快照，就是执行ps命令的那个时刻的那些进程，如果想要动态的显示进程信息，就可以使用top命令。

`ps -aux`

![Snipaste_2019-11-24_15-21-51.jpg](https://i.loli.net/2019/11/24/5gMaGJIB4Rb2UcP.jpg)


- -a 显示一个终端的所有进程，除了会话引线
- -u uid or username 选择有效的用户id或者是用户名
- -x 显示没有控制终端的进程，同时显示各个命令的具体路径。dx不可合用。

> USER：    用户名
> PID：    进程ID（Process ID）
> %CPU：    进程的cpu占用率
> %MEM：    进程的内存占用率
> VSZ：    进程所使用的虚存的大小（Virtual Size）
> RSS：    进程使用的驻留集大小或者是实际内存的大小，Kbytes字节。
> TTY：    与进程关联的终端（tty）
> STAT：    进程的状态：进程状态使用字符表示的（STAT的状态码）
> TIME：    进程使用的总cpu时间
> COMMAND：    正在执行的命令行命令

`ps -ef`

![Snipaste_2019-11-24_15-24-54.jpg](https://i.loli.net/2019/11/24/uH8wv4T3WJUtk2A.jpg)

>UID：    用户ID（User ID）
>PID：    进程ID（Process ID）
>PPID：    父进程的进程ID（Parent Process id）
>STIME：    启动时间
>TTY：    与进程关联的终端（tty）
>TIME：    进程使用的总cpu时间
>CMD：    正在执行的命令行命令

### top 命令

## 网络相关

### Linux 中查看IP地址的方式

**ip addr**

![Snipaste_2019-11-24_15-33-02.jpg](https://i.loli.net/2019/11/24/vnSx328hGbJjDOX.jpg)

**ifconfig**

![Snipaste_2019-11-24_15-32-54.jpg](https://i.loli.net/2019/11/24/uFYcTHV7NKmjblL.jpg)

### route 命令

