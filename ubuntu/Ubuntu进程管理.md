## 常用命令
```shell
command &   //将进程放在后台执行
ctrl-z      //暂停当前进程 并放入后台
jobs        //查看当前后台任务
bg          //将任务转为后台执行
fg          //将任务调回前台
kill        //杀掉任务
jps | grep <Java主类名称> | awk '{print $1}' | xargs kill -9
```

## 后台运行
如果我们在终端中直接运行一GUI程序，一般情况下，终端就会被当前进程占用了。如果我们想把它放到后台运行有两种方法：
```shell
command &
```
在运行的命令后加一个&号，就会后台运行命令
```shell
ctrl-z
bg %i
```
在终端中按ctrl-z 会将当前任务暂停并转入后台。利用jobs命令可以查看当前后台的任务，如果在jobs命令后增加 -l 参数那么就会显示详细信息，可以发现终止的进程状态为Stopped。此时通过bg %i （i为进程的标号）命令可以将其转为运行。

### 将任务切换回前台
```shell
fg %i
```

### 结束任务
```shell
kill %i
```

## ps 命令
名称：ps
* 使用权限：所有使用者
* 使用方式：`ps [options] [--help]`
* 说明：显示瞬间行程 (process) 的动态
* 参数：ps的参数非常多, 在此仅列出几个常用的参数并大略介绍含义
   * -A    列出所有的进程 
   * -w    显示加宽可以显示较多的资讯 
   * -au    显示较详细的资讯 
   * -aux    显示所有包含其他使用者的行程

### Head标头
```shell
USER    用户名
UID    用户ID（User ID）
PID    进程ID（Process ID）
PPID    父进程的进程ID（Parent Process id）
SID    会话ID（Session id）
%CPU    进程的cpu占用率
%MEM    进程的内存占用率
VSZ    进程所使用的虚存的大小（Virtual Size）
RSS    进程使用的驻留集大小或者是实际内存的大小，Kbytes字节。
TTY    与进程关联的终端（tty）
STAT    进程的状态：进程状态使用字符表示的（STAT的状态码）
R 运行    Runnable (on run queue)            正在运行或在运行队列中等待。
S 睡眠    Sleeping                休眠中, 受阻, 在等待某个条件的形成或接受到信号。
I 空闲    Idle
Z 僵死    Zombie（a defunct process)        进程已终止, 但进程描述符存在, 直到父进程调用wait4()系统调用后释放。
D 不可中断    Uninterruptible sleep (ususally IO)    收到信号不唤醒和不可运行, 进程必须等待直到有中断发生。
T 终止    Terminate                进程收到SIGSTOP, SIGSTP, SIGTIN, SIGTOU信号后停止运行运行。
P 等待交换页
W 无驻留页    has no resident pages        没有足够的记忆体分页可分配。
X 死掉的进程
< 高优先级进程                    高优先序的进程
N 低优先    级进程                    低优先序的进程
L 内存锁页    Lock                有记忆体分页分配并缩在记忆体内
s 进程的领导者（在它之下有子进程）；
l 多进程的（使用 CLONE_THREAD, 类似 NPTL pthreads）
+ 位于后台的进程组
  START    进程启动时间和日期
  TIME    进程使用的总cpu时间
  COMMAND    正在执行的命令行命令
  NI    优先级(Nice)
  PRI    进程优先级编号(Priority)
  WCHAN    进程正在睡眠的内核函数名称；该函数的名称是从/root/system.map文件中获得的。
  FLAGS    与进程相关的数字标识

```
参考
* [ps命令详解](http://www.cnblogs.com/wangkangluo1/archive/2011/09/23/2185938.html)

## Linux系统运行级别

### 运行级别(runlevel)

* 运行级别0：系统停机状态，系统默认运行级别不能设为0，否则不能正常启动
* 运行级别1：单用户工作状态，root权限，用于系统维护，禁止远程登陆
* 运行级别2：多用户状态(没有NFS)
* 运行级别3：完全的多用户状态(有NFS)，登陆后进入控制台命令行模式
* 运行级别4：系统未使用，保留
* 运行级别5：X11控制台，登陆后进入图形GUI模式
* 运行级别6：系统正常关闭并重启，默认运行级别不能设为6，否则不能正常启动

### 运行级别原理 
* 在目录/etc/rc.d/init.d下有许多服务器脚本程序，一般称为服务(service)
* 在/etc/rc.d下有7个名为rcN.d的目录，对应系统的7个运行级别
* rcN.d目录下都是一些符号链接文件，这些链接文件都指向init.d目录下的service脚本文件，命名规则为K+nn+服务名或S+nn+服务名，其中nn为两位数字。
* 系统会根据指定的运行级别进入对应的rcN.d目录，并按照文件名顺序检索目录下的链接文件
  ** 对于以K开头的文件，系统将终止对应的服务
  ** 对于以S开头的文件，系统将启动对应的服务
* 查看运行级别用：runlevel
* 进入其它运行级别用：init N
* 另外init 0为关机，init 6为重启系统

## 开机自启动

### 使用特定用户启动
```shell
sudo vi /etc/rc.local
# 加入
su - <user> -c 命令
```

### 程序崩溃或系统重启后自动重新执行
* https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-1-practical-examples
* https://www.reddit.com/r/SS13/comments/38yjys/creating_an_autorestarting_server_in_ubuntu/
* http://serverfault.com/questions/611525/standard-or-best-way-to-keep-alive-process-started-by-init-d
* [管理Ubuntu的自启动程序](http://yaxin-cn.github.io/Linux/manage-startup-program-on-ubuntu-server.html)
* [Linux系统下/etc/init/和/etc/init.d/的区别](http://liaolushen.github.io/2015/09/11/Linux%E7%B3%BB%E7%BB%9F%E4%B8%8B-etc-init-%E5%92%8C-etc-init-d-%E7%9A%84%E5%8C%BA%E5%88%AB/)

### Shell脚本实现守护进程
```shell
# #! 不是注释符，而是指定脚本由哪个解释器来执行，
# #! 后面有一个空格，空格后面为解释器的全路径且必须正确。
#! /bin/sh

PRO_PATH=""

# testpro 为要守护的可执行程序，即保证它是一直运行的
PROGRAM="testpro"


# 此脚本一直不停的循环运行，while <条件> 与 do 放在一行上要在条件后加分号
# if、then、while、do等关键字或命令是作为一个新表达式的开头，
# 一个新表达式之前的表达式必须以换行符或分号(；)来结束
# 如果条件不是单个常量或变量而是表达式的话，则要用[]括起来
# while、until与for循环皆以do开始以done结束构成循环体
while true ; do


# 休息10秒以确保要看护的程序运行起来了，这个时间因实际情况而定
    sleep 10

# 单引号''中的$符与\符没有了引用变量和转义的作用，但在双引号""中是可以的！
# 单引号中如果还有单引号，则输出时全部的单引号都将去掉，单引号括住的内容原样输出。
# 例：echo 'have 'test'' --> have test
# ps aux --> a 为显示其他用户启动的进程；
#                 u 为显示启动进程的用户名与时间；
#                 x 为显示系统属于自己的进程；
# ps aux | grep 可执行程序名 --> 在得到的当前启动的所有进程信息文本中，
#                                            过滤出包含有指定文本(即可执行程序名字)的信息文本行
＃ 注：假设 ps aux | grep 可执行程序名 有输出结果，但输出不是一条信而是两条，
# 一个为查找到的包含有指定文本(即可执行程序名字)的信息文本行(以换行符0x10结尾的文本为一行)，
# 一个为 grep 可执行程序名 ，即把自己也输出来了，
# 所这条信息是我们不需要的，因为我们只想知指定名字的可执行程序是否启动了
# grep -v 指定文本 --> 输出不包含指定文本的那一行文本信息
# wc -l --> 输出文件中的行数(-l --> 输出换行符统计数)
# ps aux | grep $PROGRAM | grep -v grep | wc -l --> 如果有指定程序名的程序启动的话，结果大于壹
    PRO_NOW=`ps aux | grep "$PROGRAM" | grep -v grep | wc -l`


# 整数比较：-lt -> 小于，-le -> 小于等于，-gt -> 大于，-ge -> 大于等于，-eq ->等于，-ne -> 不等于
# if [条件] 与 then 放在一行上要在条件后加分号
# 如果当前指定程序启动的个数小于壹的话
    if [ $PRO_NOW -lt 1 ]; then


# 0 -> 标准输入，1 -> 标准输出，2 - > 标准错误信息输出
# /dev/null --> Linux的特殊文件，它就像无底洞，所有重定向到它的信息数据都会消失！
# 2 > /dev/null --> 重定向 stderr 到 /dev/null，1 >& 2 --> 重定向 stdout 到 stderr，
# 直接启动指定程序，且不显示任何输出
# 可执行程序后面加空格加&，表示要执行的程序为后台运行
        ./$PROGRAM 2>/dev/null 1>&2 &

# date >> ./tinfo.log --> 定向输出当前日期时间到文件，添加到文件尾端，如果没有文件，则创建这个文件
        date >> ./tinfo.log

# echo "test start" >> ./tinfo.log --> 定向输出 test start 添加到文件尾端
        echo "test start" >> ./tinfo.log

# if 分支结构体结束
    fi


# 基本与上面的相同，就是多了一个 grep T，其结果为过滤出含 T 字符的信息行
# T --> 进程已停止，D --> 不可中断的深度睡眠，R --> 进程运行或就绪，S --> 可接收信号的睡眠，
# X --> 已完全死掉，Z --> 已完全终止
    PRO_STAT=`ps aux|grep "$PROGRAM" |grep T|grep -v grep|wc -l`


# 如果指定进程状态为已停止的信息大于零的话
    if [ $PRO_STAT -gt 0 ] ; then


# killall --> 用名字方式来杀死进程，-9 --> 即发给程序一个信号值为9的信号，即SIGKILL(非法硬件指令)
# 也可以不指定信号，默认为SIGTERM，即信号值为15
        killall -9 $PROGRAM
        sleep 2
        ./$PROGRAM 2>/dev/null 1>&2 &
        date >> ./tinfo.log
        echo "test start" >> ./tinfo.log
    fi


# while、until与for循环皆以do开始以done结束构成循环体
done


# exit 用来结束脚本并返回状态值，0 - 为成功，非零值为错误码，取值范围为0 ~ 255。
exit 0
```

## 计划任务 crontab

```
*　　*　　*　　*　　*　　command
分　 时　 日　 月　 周　 命令
```

- 第1列表示分钟1～59 每分钟用*或者 */1表示
- 第2列表示小时1～23（0表示0点）
- 第3列表示日期1～31
- 第4列表示月份1～12
- 第5列标识号星期0～6（0表示星期天）
- 第6列要运行的命令

### crontab文件示例

| 命令    | 说明   |
| ------ | ------ |
|30 21 * * * /usr/local/etc/rc.d/lighttpd restart      |''每晚的21:30重启lighttpd'' |
|45 4 1,10,22 * * /usr/local/etc/rc.d/lighttpd restart |''每月1、10、22日的4 : 45重启lighttpd'' |
|10 1 * * 6,0 /usr/local/etc/rc.d/lighttpd restart     |''每周六、周日的1 : 10重启lighttpd'' |
|0,30 18-23 * * * /usr/local/etc/rc.d/lighttpd restart |''每天18 : 00至23 : 00之间每隔30分钟重启 |lighttpd''
|0 23 * * 6 /usr/local/etc/rc.d/lighttpd restart       |''每星期六的11 : 00 pm重启lighttpd'' |
|* */1 * * * /usr/local/etc/rc.d/lighttpd restart      |''每一小时重启lighttpd'' |
|* 23-7/1 * * * /usr/local/etc/rc.d/lighttpd restart   |''晚上11点到早上7点之间，每隔一小时重启lighttpd'' |
|0 11 4 * mon-wed /usr/local/etc/rc.d/lighttpd restart |''每月的4号与每周一到周三的11点重启lighttpd'' |
|0 4 1 jan * /usr/local/etc/rc.d/lighttpd restart      |''一月一号的4点重启lighttpd'' |

## 僵尸进程
查找僵尸进程
```shell
ps -A -ostat,ppid,pid,cmd | grep -e '^[zZ]'
```

## sudoers
通过设定sudoers，可以定制特定用户/用户组的程序执行权限。
```shell
sudo visudo
<user list> <host list> = <operator list> <tag list> <command list>
www-data ALL=(ALL) NOPASSWD: /usr/bin/java,/bin/kill,/bin/bash
```

如果只是希望当前用户在输出sudo后不用继续输入密码：
```shell
%sudo   ALL=(ALL:ALL) NOPASSWD: ALL
```
当前用户应该在sudo group

参考：
* [Sudoers](https://help.ubuntu.com/community/Sudoers)
* [Running scripts as root from PHP](http://codebin.co.uk/blog/running-scripts-as-root-from-php/)