## 系统

### 设备信息
```shell
# 查看内核/操作系统/CPU信息
uname -a
# 查看操作系统版本
head -n 1 /etc/issue
# 查看计算机名
hostname
# 列出所有PCI设备
lspci -tv
# 列出所有USB设备
lsusb -tv
# 列出加载的内核模块
lsmod
# 查看环境变量
env
# CPU信息
cat /proc/cpuinfo
sudo dmidecode | grep -57 "Processor Information$"
# CPU主频
lscpu | grep MHz
sudo dmidecode -t processor | grep Speed
# 设置CPU主频
sudo apt install cpufrequtils
sudo cpufreq-set -f 1700
# 内存信息
sudo dmidecode | grep -A16 "Memory Device$"
# 磁盘信息
sudo hdparm -I /dev/sda | grep -A4 "Model Number"
sudo smartctl -A /dev/sda
sudo blkid /dev/sda
```

### 测试磁盘读写速度
```shell
# 写速度
time dd if=/dev/zero of=test.dbf bs=8k count=300000
# 读速度
time dd if=/dev/sda1 of=/dev/null bs=8k
```

### 资源使用
```shell
# 查看磁盘使用情况
df -lh
# CPU资源占用
top -c
# 查看内存使用情况
free -m
# 最近一分钟，五分钟和十五分钟的系统负载 (0-1)*cpu核数
uptime
```

#### vmstat
磁盘活动
- bi：表示从磁盘每秒读取的块数（blocks/s）。数字越大，表示读磁盘的活动越多。
- bo：表示每秒写到磁盘的块数（blocks/s）。数字越大，表示写磁盘的活动越多。
CPU活动
- us：用户程序使用cpu的时间比例。这个数字越大，表示用户进程越繁忙。
- sy：系统调用使用cpu的时间比例。注意，NFS由于是在内核里面运行的，所以NFS活动所占用的cpu时间反映在sy里面。这个数字经常很大的 话，就需要注意是否某个内核进程，比如NFS任务比较繁重。如果us和sy同时都比较大的话，就需要考虑将某些用户程序分离到另外的服务器上面，以免互相 影响。
- id：cpu空闲的时间比例。
- wa：cpu等待未决的磁盘IO的时间比例。数字越大，表示文件系统活动阻碍cpu的情况越严重，因为cpu在等待慢速的磁盘系统提供数据。wa为0是最理想的。如果wa经常大于10，可能文件系统就需要进行性能调整了。

#### 端口使用
```shell
netstat -a  // 查看已经连接的服务端口 ESTABLISHED
netstat -ap // 查看所有的服务端口（LISTEN，ESTABLISHED）
netstat -ap | grep 8080 // 查看8080端口
lsof -i:8888 // 查看8888端口
sudo lsof -Pnl +M -i4 //ipv4端口使用
sudo lsof -Pnl +M -i6 //ipv6端口使用
```

#### 网络流量统计
```shell
sudo apt-get install ethstatus
sudo ethstatus -i eth0
```

### 性能测试
```shell
 sysbench --test=cpu --num-threads=128 --cpu-max-prime=200000 run
 sudo apt-get install hardinfo
 hardinfo | less
```

### 禁止自动休眠
```shell
sudo systemctl mask sleep.target suspend.target hibernate.target hybrid-sleep.target
```

### 用xset修改DPMS和屏保设定
- [Display Power Management Signaling](https://wiki.archlinuxcn.org/wiki/Display_Power_Management_Signaling)
```shell
# 禁用屏保清空
xset s off
# 将清空时间设置到 1 小时
xset s 3600 3600
# 关闭 DPMS
xset -dpms
# 禁用 DPMS 并阻止屏幕清空
xset s off -dpms
# 立即关闭屏幕
xset dpms force off
# 待机界面
xset dpms force standby
# 休眠界面
xset dpms force suspend
```

### 设定系统时间
```shell
# 选择时区
tzselect

# 设置
cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

# 更新时间
ntpdate time.windows.com

# 新增TZ环境变量（写入 /etc/profile，对所有用户生效）
export TZ='Asia/Shanghai'
# 此时 JDK 获取的时间与本地时间将一致
```

## 远程

### 免密码登陆
在A机下生成公钥/私钥对：
```shell
ssh-keygen -t ed25519
```
把A机下的`id_rsa.pub`复制到B机下的`.ssh/authorized_keys`文件里。`authorized_keys`的权限要是600。A机登录B机，第一次登录是时输入yes，即可无密码登录。

如果遇到如下错误：
```
The authenticity of host <X.X.X.X> can't be established.
```
执行:
```shell
ssh -o StrictHostKeyChecking=no <X.X.X.X>
```
### 禁止用户SSH登录
```shell
vi /etc/ssh/sshd_config

# 禁止用户user1登陆，多个空格分隔
DenyUsers user1
# 禁止用户组group1的所有用户登录，多个空格分隔
DenyGroups group1

# 保存配置后，重启sshd
/etc/rc.d/init.d/sshd restart
```

### 远程唤醒主机
```shell
sudo apt-get install etherwake
# ether-wake MAC-Address-Here
# wakeonlan xx:yy:zz:11:22:33
# etherwake xx:yy:zz:11:22:33
```

## 用户
1. 添加sudo权限
   - `sudo usermod -aG sudo <user>`
1. 禁用用户
   - 修改 /etc/shadow 中的密码栏，加感叹号(!)，带!号为禁用，没有!则为启用
   - 使用 usermod 命令（usermod -L 禁用，usermod -U 启用）
   - 使用 passwd 命令（passowd -l 禁用，passwd -u 启用）

## 异常
### 内核
在/var/log/kern.log每秒钟出现如下一条报错：
```shell
kernel: [ 7008.540708] EDAC MC0: 1 CE error on CPU#0Channel#1_DIMM#0 (channel:1 slot:0 page:0x0 offset:0x0 grain:8 syndrome:0x0)
```
原因是内存出错/内存插座接口出错，需要找到错误的那根内存条，替换或换一个插槽：
```shell
grep "[0-9]" /sys/devices/system/edac/mc/mc*/csrow*/ch*_ce_count
```

### apt-get half-installed 错误情况
错误信息：
```shell
dpkg: error processing libgfortran3:amd64 (--configure):
package libgfortran3:amd64 is not ready for configuration
cannot configure (current status `half-installed')
```
解决方法：
```shell
sudo dpkg --remove --force-remove-reinstreq --dry-run libgfortran3:amd64
```

### System freezes completely with Intel Bay Trail (J2900)
Ubunut Desktop 16.04 在搭载 J2900 CPU 的主板，会随机死机，解决方法如下：
```shell
sudo vi /etc/default/grub
```
修改：
```shell
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash intel_idle.max_cstate=1"
```
然后：
```shell
sudo update-grub
```
重启

### 文件 root 用户不可写
- 通常情况下是由于系统管理不当中了病毒，重装系统吧，**记得所有相关密钥都要废弃**

### 一般问题排查
- 如果系统死机前没有任何的log输出，最有可能是linux系统bug所致，主要原因是当前linux内核/并行版本与主板/CPU不兼容，根据这个方向google问题解决方法
- 硬盘问题：死机时磁盘灯常亮，平时磁盘异响，无法boot等
- 内存问题：不常见
- 主板问题：不常见
- 电源问题：极不常见