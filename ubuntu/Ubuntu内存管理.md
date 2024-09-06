## 查看内存使用情况
```shell
free -m
-------------------------------------
total used free shared buffers cached
Mem: 515588 295452 220136 0 2060 64040
-/+ buffers/cache: 229352 286236
Swap: 682720 112 682608
```

其中：
* total：总物理内存
* used：已使用内存，一般情况这个值会比较大，因为这个值包括了cache+应用程序使用的内存
* free：未被使用的内存
* shared：应用程序共享内存
* buffers：缓存，主要用于目录方面，inode值等（ls大目录可看到这个值增加）
* cached：缓存，用于已打开的文件
* total = used + free
* -buffers/cache = used - buffers - cached
* +buffers/cache = free + buffers + cached

参考：
* [Tips for Optimizing Linux Memory Usage](http://www.linuxjournal.com/article/2770 )

## 查看内存频率
```shell
sudo dmidecode | grep -A16 "Memory Device"|grep 'Speed'
```

## ps 内存指标说明
- RSS
   - RSS is the Resident Set Size and is used to show how much memory is allocated to that process and is in RAM. It does not include memory that is swapped out. It does include memory from shared libraries as long as the pages from those libraries are actually in memory. It does include all stack and heap memory.
- VSZ
   - VSZ is the Virtual Memory Size. It includes all memory that the process can access, including memory that is swapped out and memory that is from shared libraries.

## 释放缓存
```shell
sync
echo 3 > /proc/sys/vm/drop_caches
# 或
sudo sh -c "sync; echo 3 > /proc/sys/vm/drop_caches"
```

### 使用 crontab 周期执行释放缓存命令
```shell
sudo su
crontab -e
# 增加条目
*/15 * * * * sudo sh -c "sync; echo 3 > /proc/sys/vm/drop_caches"
# 重启cron
service cron restart
```

### 参考
* [Linux 内核的文件 Cache 管理机制介绍](https://www.ibm.com/developerworks/cn/linux/l-cache/)
* [How to Clear RAM Memory Cache, Buffer and Swap Space on Linux](http://www.tecmint.com/clear-ram-memory-cache-buffer-and-swap-space-on-linux/)
