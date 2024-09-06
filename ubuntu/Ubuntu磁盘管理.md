## 查看磁盘

```shell
fdisk -l
hdparm -I /dev/sda
```
## 分区

### 一般分区
```shell
fdisk -S 56 /dev/xvdb
根据提示，依次输入“n”，“p”“1”，两次回车，“wq”，分区就开始了，很快就会完成。
```

### GPT方式分区
```shell
parted /dev/sdb
mklabel gpt
mkpart primary 2048s 100%
```
* [http://rainbow.chard.org/2013/01/30/how-to-align-partitions-for-best-performance-using-parted/ How to align partitions for best performance using parted]

## 格式化
```shell
mkfs -t ext4 /dev/sdb1

# 亦可
mkfs.ext4 /dev/xvdb1

# 添加分区信息
echo '/dev/xvdb1   /data ext4    defaults    0  0' >> /etc/fstab
# 挂载
mount -a
```
### GPT方式

`sudo blkid`获取磁盘uuid


`vi /etc/fstab`，添加
```
UUID=735535dc-946a-4455-acc0-f34eab619bc8       /sd/sd1        ext4    defaults        0       1
UUID=a6abb1d7-247b-4283-bb4a-1231b37c86c5       /sd/sd2        ext4    defaults        0       1
UUID=f2e05ddb-5c6e-47e4-82a8-b2ed33f6e561       /sd/sd3        ext4    defaults        0       1
UUID=c8fa8d89-4ec2-4a20-b7a6-3f57b49cce79       /sd/sd4        ext4    defaults        0       1
```

### 低级格式化
低格就是用0或1去覆盖整个硬盘，速度很慢，你可以自己不在的时候执行
```shell
dd if=/dev/zero of=/dev/sda
sudo dd if=/dev/zero of=/dev/sda && sudo shutdown -h +2
```

## LVM 管理
### 扩容
```shell
lvextend -L 10G /dev/mapper/ubuntu--vg-ubuntu--lv      //增大或减小至19G
lvextend -L +10G /dev/mapper/ubuntu--vg-ubuntu--lv     //增加10G
lvreduce -L -10G /dev/mapper/ubuntu--vg-ubuntu--lv     //减小10G
lvresize -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv   //按百分比扩容

resize2fs /dev/mapper/ubuntu--vg-ubuntu--lv            //执行调整
```

## SMART 测试
```shell
# 查看磁盘是否支持SMART
smartctl -i /dev/sda
# 查看简要测试结果
smartctl -H /dev/sda
# 查看信息
smartctl -a /dev/sda
```

**衷心奉劝**
* 如果出现C5当前待映射的扇区数/05重映射扇区，需要立即更换硬盘，没必要进行坏道检测
* 如果出现如：print_req_error: I/O error, dev sda, sector 5617835656等错误，需要立即更换硬盘，没必要进行坏道检测

参考：
* [硬盘SMART检测参数详解](https://www.cnblogs.com/xqzt/p/5512075.html)

## 坏道检测
```shell
badblocks -s -v -o /root/badblocks.log /dev/sda
```

参考：
* https://www.runoob.com/linux/linux-comm-badblocks.html
* [Linux磁盘坏道的检测及修复](https://blog.51cto.com/netpro/515141 )

## 速度测试
```shell
hdparm -Tt /dev/sda
```

## 创建RAID
```shell
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=4 /dev/sda /dev/sdc /dev/sdd /dev/sde
```
查看进度
```shell
cat /proc/mdstat
```
结果示例
```shell
Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10]
md0 : active raid5 sde[4] sdd[2] sdc[1] sda[0]
8790402048 blocks super 1.2 level 5, 512k chunk, algorithm 2 [4/3] [UUU_]
[>....................]  recovery =  0.2% (7558168/2930134016) finish=402.6min speed=120977K/sec
bitmap: 0/22 pages [0KB], 65536KB chunk

unused devices: <none>
```
格式化创建好的磁盘组
```shell
sudo mkfs.ext4 -F /dev/md0
```
挂载
```shell
sudo mount /dev/md0 /mnt
```
查看空间
```shell
df -h -x devtmpfs -x tmpfs
```
这一步感觉不需要
```shell
sudo mdadm --detail --scan | sudo tee -a /etc/mdadm/mdadm.conf
update the initramfs
update-initramfs -u
```
保存/etc/fstab
```shell
echo '/dev/md0 /mnt ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
```
参考：
* https://www.digitalocean.com/community/tutorials/how-to-create-raid-arrays-with-mdadm-on-ubuntu-18-04
* https://raid.wiki.kernel.org/index.php/Recovering_a_failed_software_RAID
* https://www.cnblogs.com/ariclee/p/6403396.html

## 增加swap交换空间
查看系统 swap 分区大小：
```shell
free -m
```
创建 swapfile，大小为1GB
```shell
mkdir /swap
cd /swap
sudo dd if=/dev/zero of=swapfile bs=1024 count=1024000
```
把生成的文件转换成 swap 文件
```shell
mkswap swapfile
```
激活 swapfile
```shell
sudo swapon swapfile
```
卸载 swapfile
```shell
sudo swapoff swapfile
```
如果需要一直保持这个 swap，写入 /etc/fstab
```shell
/swap/swapfile  /swap swap  defaults 0 0
```