## 操作系统

## 系统

### CPU
```shell
# 获CPU温度
cat /sys/class/thermal/thermal_zone0/temp
```

### 磁盘
- Resize 分区：https://raspberrypi.stackexchange.com/questions/499/how-can-i-resize-my-root-partition

## SSH
- [Remote access](https://www.raspberrypi.com/documentation/computers/remote-access.html)

## UI

### 中文输入法
```shell
sudo apt-get update
sudo apt-get install scim-pinyin
sudo reboot
```

## GPIO
```shell
# echo 223 > /sys/class/gpio/export // 定义使用针脚
# echo out > /sys/class/gpio/gpio223/direction // 定义输出方向
# echo 0 > /sys/class/gpio/gpio223/value //
# echo 1 > /sys/class/gpio/gpio223/value // 输出3.3V
# echo in > /sys/class/gpio/gpio223/direction 
# cat /sys/class/gpio/gpio223/value // 读取输出电压
# echo 223 > /sys/class/gpio/unexport // 卸载
```

参考：
- [Pi4J](http://pi4j.com/)
- [Orange Pi PC: Fan on GPIO](https://forum.armbian.com/topic/1774-orange-pi-pc-fan-on-gpio/)

## SMS
- [使用Python在树莓派上收发短信](https://hristoborisov.com/index.php/projects/turning-the-raspberry-pi-into-a-sms-center-using-python/)
- 商品链接：https://item.taobao.com/item.htm?id=542608922284
- [CH340 Linux驱动使用教程](https://blog.csdn.net/JAZZSOLDIER/article/details/70170466)

### USB SIM800C模块
#### 安装
```shell
# 删除旧版驱动，位置：
/lib/modules/$(uname -r)/kernel/drivers/usb/serial/ch341.ko

# 下载驱动 http://www.wch.cn/download/CH341SER_LINUX_ZIP.html
# 解压缩编译
make

# 需要注意的时，系统内核>=4.11时，需要对源代码做如下修改：
# 页头添加：
include <linux/version.h>
if LINUX_VERSION_CODE >= KERNEL_VERSION(4, 11, 0)
include <linux/sched/signal.h>
endif

# 591行
wait_queue_t --> wait_queue_entry_t 

# 加载驱动：
make load

# 插入CH340硬件，查看日志
dmesg
# [511051.620969] usb 1-1.4.1: new full-speed USB device number 15 using xhci_hcd
# [511051.722225] usb 1-1.4.1: New USB device found, idVendor=1a86, idProduct=7523
# [511051.722229] usb 1-1.4.1: New USB device strings: Mfr=0, Product=2, SerialNumber=0
# [511051.722231] usb 1-1.4.1: Product: USB2.0-Serial
# [511051.723483] ch341 1-1.4.1:1.0: ch341-uart converter detected
# [511051.724433] usb 1-1.4.1: ch341-uart converter now attached to ttyUSB0

# 自动加载驱动
cp ch34x.ko /lib/modules/$(uname -r)/kernel/drivers/usb/serial
```

### minicom
#### 连接设备
```shell
# 查看USB串口设备
ls /dev/ttyUSB*

# 如果是普通串口设备
ls /dev/ttyS*

# 连接设备
minicom --device /dev/ttyUSB0
# 如果连接不成功尝试设置chmod a+x 777 /dev/ttyUSB0
```

#### minicom不能接受键盘输入
```shell
https://blog.csdn.net/David_xtd/article/details/7652120
Ctrl-A -> Z -> E
```

#### Java串口编程
- Minicom代码示例：
   - https://github.com/maximyep/adbb/blob/master/Test/src/test/Minicom.java
- 串口库： 
   - [jSerialComm](https://fazecast.github.io/jSerialComm/)

#### AT 命令
- [树莓派.GPRS.短信接收器](https://mc.dfrobot.com.cn/thread-29044-1-1.html)
- [树莓派使用sim800过程中的一些小结](https://blog.51cto.com/mayuenjkxt/1794487)
- [SIM800C模块TCP透传模式AT设置](https://blog.csdn.net/yuanlaibobo/article/details/79260221)
- [SIM800系列模块GSM/GPRS建立TCP连接到远端服务器过程](https://blog.csdn.net/lushengchu_luis/article/details/62894267)
- [树莓派+Linux+Java驱动SIM800C GPRS模块实现TCP数据传输](https://zhoujianshi.github.io/articles/2016/%E6%A0%91%E8%8E%93%E6%B4%BE+Linux+Java%E9%A9%B1%E5%8A%A8SIM800C%20GPRS%E6%A8%A1%E5%9D%97%E5%AE%9E%E7%8E%B0TCP%E6%95%B0%E6%8D%AE%E4%BC%A0%E8%BE%93/index.html)
- [Reading SMS Messages from a Message Storage Area Using AT Commands (AT+CMGR, AT+CMGL)](https://www.developershome.com/sms/readSmsByAtCommands.asp)

##### 初始化
```shell
# 握手 / SIM卡检测等
AT

# 查询是否检测到SIM卡
AT+CPIN?

# 信号质量测试
# 返回 +CSQ: **,##
# ** 应该在10-31之间，数值越大代表信号越好
# ## 为误码率，0-99之间
AT+CSQ

# 读取SIM的CCID(SIM卡背面20位数字)，可以检测是否有SIM卡或者是否接触良好
AT+CCID

# 检测是否注册网络
AT+CREG?
```

##### 信息查询
```shell
# IMEI
AT+CGSN

# IMSI
AT+CIMI

# 获得SIM卡的标识
# 读取SIM卡上的EF-CCID文件
AT+CCID

# 模块厂商标识
AT+CGMI

# 网络注册状态，返回：+CREG: <mode>, <stat>
# <mode>为0，1，2
# <stat>为0（没有注册网络），1（注册到本地网络），2（找到运营商没有注册到网络），3（注册被拒绝），4（位置），5（漫游注册）
AT+CREG=1
```

##### 短信相关
```shell
# 设置短信指令格式，分2种短信格式： 0-PDU, 1-文本格式
AT+CMGF=1

# 设置短信文本格式
AT+CSDH=1

# 查看所有短信
AT+CMGL="ALL"

# 按序号查看短信
AT+CMGR=<Index>

# 读取未读短信
AT+CMGL="REC UNREAD"

# 读取已读短信
AT+CMGL="REC READ"
```

## 扩展
### Sense HAT
```shell
sudo apt-get install cmake
git clone https://github.com/RPi-Distro/RTIMULib.git
cd Linux/python/
sudo python3 setup.py build
sudo python3 setup.py instal
sudo apt install libopenjp2-7
sudo pip3 install sense-hat
```
参考：
- [Sense HAT](https://pythonhosted.org/sense-hat/)
- https://github.com/astro-pi/python-sense-hat

## Tinkerboard
- GPIO针脚图: https://www.asus.com/us/Single-Board-Computer/Tinker-Board/ 页面最下方
- armbian: https://www.armbian.com/tinkerboard/
   - 稳定配置：烧录后可正常启动，可正常播放视频，HDMI可正常输出音频，可正常进行进行GPU加速
   - [RK3288 Media Script](https://forum.armbian.com/topic/7262-rk3288-media-script-tinkerboard/)