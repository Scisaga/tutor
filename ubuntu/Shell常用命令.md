## 基础

### 常用
```shell
# 进程查询
ps aux | grep java

# 寻找特定文件
find -name my.cnf

# 查找文件内容
grep -rl "string" /path

# 立即关机
shutdown -h 00

# .sh执行权限
chmod u+x *.sh

# 查看环境变量
echo $JAVA_HOME
```

### 环境变量
系统环境变量配置文件：
- `/etc/profile`：在登录时，操作系统定制用户环境时使用的第一个文件，此文件为系统的每个用户设置环境信息，当用户第一次登录时，该文件被执行。
- `/etc/environment`：在登录时操作系统使用的第二个文件，系统在读取你自己的profile前，设置环境文件的环境变量。
- `~/.profile`：在登录时用到的第三个文件是.profile文件，每个用户都可使用该文件输入专用于自己使用的shell信息，当用户登录时，该文件仅仅执行一次。
- `/etc/bashrc`：为每一个运行bash shell的用户执行此文件。当bash shell被打开时，该文件被读取。
- `~/.bashrc`：该文件包含专用于你的bash shell的bash信息，当登录时以及每次打开新的shell时，该文件被读取。

```shell
# 打印环境变量
printenv
printenv | less
printenv | more
```

### Shell脚本
- 格式：`#!/bin/bash`，文件名后缀 `.sh`
- 变量赋值：`` path=`pwd` ``，其中=两边不能有空格，包裹pwd的符号是~下面的符号
- 引用变量用 `$path`
- 特殊符号需要转义，例如 `.` `,` `;` `~` `` `  `` `|`

### 一行执行多条命令
- `;`
   - 如果被分号(;)所分隔的命令会连续的执行下去，就算是错误的命令也会继续执行后面的命令。
- `&&`
   - 如果命令被 && 所分隔，那么命令也会一直执行下去，但是中间有错误的命令存在就不会执行后面的命令，没错就直行至完为止。
- `||`
   - 如果每个命令被双竖线 || 所分隔，那么一遇到可以执行成功的命令就会停止执行后面的命令，而不管后面的命令是否正确与否。如果执行到错误的命令就是继续执行后一个命令，一直执行到遇到正确的命令为止

### 输入/输出重定向
在Linux命令行模式中，如果命令所需的输出不是来自键盘，而是来自指定的文件，这就是输入重定向。同理，命令的输出也可以不显示在屏幕上，而是写入到指定文件中，这就是输出重定向。

```shell
# 输出重定向
who > users
cat users

# 输出追加
command >> file

# 输入重定向
wc -l < users // 不会输出文件名，只知道从标准输入中读取内容

# 一般格式
command1 < infile > outfile

# stderr 重定向到 file
command 2>file

# 将 stdout 和 stderr 合并后追加方式重定向到 file
command >> file 2>&1

# 屏蔽 stdout 和 stderr
command > /dev/null 2>&1
```

扩展：
- 一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件： 
   - 标准输入文件(stdin)：stdin的文件描述符为0，Unix程序默认从stdin读取数据。 
   - 标准输出文件(stdout)：stdout 的文件描述符为1，Unix程序默认向stdout输出数据。 
   - 标准错误文件(stderr)：stderr的文件描述符为2，Unix程序会向stderr流中写入错误信息。 
- 默认情况下，command > file 将 stdout 重定向到 file，command < file 将stdin 重定向到 file。

#### Here Document
Here Document 是 Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 Shell 脚本或程序，将两个 delimiter 之间的内容(document) 作为输入传递给 command。
- 结尾的delimiter 一定要顶格写，前面不能有任何字符，后面也不能有任何字符，包括空格和 tab 缩进。
- 开始的delimiter前后的空格会被忽略掉。

```shell
# 创建文件示例
cat <<EOF > /etc/wireguard/wg0.conf
[Interface]
...
[Peer]
...
EOF
```

#### nohup 后台执行
```shell
# 只输出错误信息到日志文件
nohup ./program >/dev/null 2>log &
# 什么信息也不要
nohup ./program >/dev/null 2>&1 &
```

### 管道
利用Linux所提供的管道符“|”将两个命令隔开，管道符左边命令的输出就会作为管道符右边命令的输入。连续使用管道意味着第一个命令的输出会作为第二个命令的输入，第二个命令的输出又会作为第三个命令的输入，依此类推。
```shell
# 管道将rpm -qa命令的输出（包括系统中所有安装的RPM包）作为grep命令的输入，从而列出带有licq字符的RPM包来
rpm -qa | grep licq

# 利用第一个管道将cat命令（显示passwd文件的内容）的输出送给grep命令，grep命令找出含有“/bin/bash”的所有行；第二个管道将grep的输入送给wc命令，wc命令统计出输入中的行数。这个命令的功能在于找出系统中有多少个用户使用bash。
cat /etc/passwd | grep /bin/bash | wc -l
```

### 命令替换
在Linux命令行模式下，当遇到一对“`” (上分割符)时，将首先执行“`”中间包含的命令，然后将其输出结果作为参数代入命令行中，这就是命令替换了。它类似于输入输出的重定向功能，但区别在于命令替换是将一个命令的输出作为另外一个命令的参数。下面来看它的实际应用。

```shell
# date +%Y%m%d%k%M%S命令将首先执行，它将按指定格式输出当前的时间。然后，这个时间将被作为touch命令的参数，其结果是建立了一个以当前时间为文件名的文件。
touch `date +%Y%m%d%k%M%S`.txt

# 该命令将杀掉sshd的所有进程。这里用pidof这个命令给出进程号，因为kill是针对进程号进行操作的。两者通过命令替换，实现了只用一条命令就杀掉sshd所有进程的功能。
kill `/sbin/pidof sshd`

# 使用了三个管道和一次命令替换，另外使用了grep、tr和awk三个与字符操作相关的命令
kill -9 `ps -ef | grep smbd | tr -s ' ' | awk -F' ' '{print $2}'`
```

### Exit Code
- 0 for successful executions and 1 or higher for failed executions.

```shell

cat <<EOF > ./tmp.sh
#!/bin/bash
touch /root/test
EOF

./tmp.sh 
# touch: cannot touch '/root/test': Permission denied

$ echo $?
# 1
```

## 文件

### 文件查找
```shell
全盘查找大于1G的文件
find / -type f -size +1024000k -exec du -h {} \;

查找当前目录下大于10MB的文件
find . -type f -size +10000k -exec ls -lh {} \; | awk ‘{ print $8 “: ” $5 }’

# 查看当前文件夹所有子文件夹大小
du -lh --max-depth=1
```

### less 文件浏览
```shell
less <文件名>
less +100g xx.log # 直接定位到第100行
less +GG xx.log # 定位到最后一行
less +100P xx.log # 定位到第100个字节的位置
```
命令：
- j 向前移动一行
- k 向后移动一行
- PageUp
- PageDown
- G 文件末尾
- =：显示当前行信息，如行号、字节位置等（可能要计算，毕竟没有加载整个信息！）
- / 使用一个模式进行搜索，并定位到下一个匹配的文本
- n 向前查找下一个匹配的文本
- N 向后查找前一个匹配的文本

### sed 文件内容替换
文件开头添加内容
```shell
sudo sed -i "1i\auth_param basic program /usr/lib/squid/basic_ncsa_auth /etc/squid/squid_passwd\n \
acl ncsa_users proxy_auth REQUIRED\n \
http_access allow ncsa_users\n" /etc/squid/squid.conf
```
文本内容替换
```shell
sudo sed -i "s/.*http_port 3128.*/http_port 3228/" /etc/squid/squid.conf
```
文件末尾添加内容
```shell
sudo sed -i '$a\forwarded_for off \
...
request_header_access All deny all\n' /etc/squid/squid.conf
```

### 快速清空文件
```shell
: > filename #其中的 : 是一个占位符, 不产生任何输出.
> filename
echo "" > filename
echo /dev/null > filename
echo > filename
cat /dev/null > filename
cp /dev/null filename
```

## tar 压缩/解压缩

### pigz
```shell
# 压缩
# pigz：用法-9是压缩比率比较大，-p是指定cpu的核数
tar cvf - 目录名 | pigz -9 -p 24 > file.tgz

# 解压
pigz -d file.tgz
# 快速解压
pigz -dc target.tar.gz | tar xf -
```

### tar 参数
命令参数
- -c: 建立压缩档案 
- -x：解压 
- -t：查看内容 
- -r：向压缩归档文件末尾追加文件 
- -u：更新原压缩包中的文件
可选参数
- -z：有gzip属性的
- -j：有bz2属性的
- -Z：有compress属性的
- -v：显示所有过程
- -O：将文件解开到标准输出
- -C：接文件夹路径，解压缩到特定文件夹

### 压缩
```shell
tar -cvf jpg.tar *.jpg //将目录里所有jpg文件打包成tar.jpg
tar -czf jpg.tar.gz *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用gzip压缩，生成一个gzip压缩过的包，命名为jpg.tar.gz
tar -cjf jpg.tar.bz2 *.jpg //将目录里所有jpg文件打包成jpg.tar后，并且将其用bzip2压缩，生成一个bzip2压缩过的包，命名为jpg.tar.bz2
tar -cZf jpg.tar.Z *.jpg   //将目录里所有jpg文件打包成jpg.tar后，并且将其用compress压缩，生成一个umcompress压缩过的包，命名为jpg.tar.Z
rar a jpg.rar *.jpg //rar格式的压缩，需要先下载rar for linux
zip jpg.zip *.jpg //zip格式的压缩，需要先下载zip for linux
```

### 解压
```shell
tar -xvf file.tar //解压 tar包
tar -xzvf file.tar.gz //解压tar.gz
tar -xjvf file.tar.bz2   //解压 tar.bz2
tar -xZvf file.tar.Z   //解压tar.Z
unrar e file.rar //解压rar
unzip file.zip //解压zip
```

### 文件格式一览
- *.tar 用 tar -xvf 解压
- *.gz 用 gzip -d 或者 gunzip 解压
- *.tar.gz 和 *.tgz 用 tar -xzf 解压
- *.bz2 用 bzip2 -d 或者用 bunzip2 解压
- *.tar.bz2 用 tar -xjf 解压
- *.Z 用 uncompress 解压
- *.tar.Z 用tar -xZf 解压
- *.rar 用 unrar e解压
- *.zip 用 unzip 解压

## vi

### 查找
```shell
# 向下查找pattern匹配字符串
/pattern<Enter>
#  向上查找pattern匹配字符串
?pattern<Enter>
#  使用了查找命令之后，使用如下两个键快速查找：
n // 按照同一方向继续查找
N // 按照反方向查找
```

### 删除
```shell
dG // 这是删除光标所在行到最后一行的内容（包括光标所在行的内容）
# 删除1到n行
:1,10d
# 删除10行到光标所在行
:10,.d
```
### 全选
```shell
ggVG
```

解释是：
- gg 让光标移到首行，在vim才有效，vi中无效 
- V 是进入Visual(可视）模式 
- G 光标移到最后一行 
- 选中内容以后就可以其他的操作了，比如： 
   - d 删除选中内容 
   - y 复制选中内容到0号寄存器
   - "+y 复制选中内容到＋寄存器，也就是系统的剪贴板，供其他程序用

## 数据传输
### 下载
```shell
# 下载并打印
wget -qO- http://example.com

# 使用代理
wget -c -r -np -k -L -p -e "http_proxy=<host>:<port>" http://ddns.oray.com/checkip

# 下载并执行
bash <(wget -qO- https://domain/tmp.sh)

# 多线程
axel -a -n [Num_of_Thread] link1 link2 link3 ...
```

### 传输

#### scp
```shell
# 本地到远程
scp /home/space/music/1.mp3 root@www.cumt.edu.cn:/home/root/others/music
scp /home/space/music/1.mp3 root@www.cumt.edu.cn:/home/root/others/music/001.mp3 // 文件的重命名
 
// 将本地 music 目录复制到远程 others 目录下
scp -r /home/space/music/ root@www.cumt.edu.cn:/home/root/others/

# 远程到本地
scp -r root@www.cumt.edu.cn:/var/lib/mysql/sae_rela/ /var/lib/mysql/
```
参数：
- -v 显示进度
- -C 使能压缩选项
- -P 选择端口
- -4 强行使用IPV4地址
- -6 强行使用IPV6地址
- 4 备份命令
- 
#### 压缩/加密传输/解压一体
```shell
time tar -c <local_folder> | lz4 | ssh -c aes128-ctr -o "MACs umac-64@openssh.com" root@<remote_host> "lz4 -d | tar -xC <remote_parent_folder>"
```
参考：
- [Benchmarking SSH connection: What is the fastest cipher algorithm for RPi?](https://blog.twogate.com/entry/2020/07/30/benchmarking-ssh-connection-what-is-the-fastest-cipher)
- [使用tar+lz4/pigz+ssh更快的数据传输](https://www.orczhou.com/index.php/2013/11/tranfer-data-faster-on-the-fly/)