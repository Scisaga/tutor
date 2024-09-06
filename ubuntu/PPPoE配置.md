
## 服务端
下载安装rp-pppoe
```shell
apt install libc6-dev gcc make ppp
wget https://www.roaringpenguin.com/files/download/rp-pppoe-3.12.tar.gz
tar xzvf rp-pppoe-3.12.tar.gz
./rp-pppoe-3.12/go
```

vi /etc/ppp/options
```shell
ms-dns 8.8.8.8
ms-dns 8.8.4.4
asyncmap 0
auth
crtscts
lock
hide-password
modem
-pap
+chap
lcp-echo-interval 30
lcp-echo-failure 4
noipx
```
其中-pap和+chap配置代表选择chap验证，pap验证方式无法在XP以上系统使用。

vi /etc/ppp/pppoe-server-options
```shell
require-chap
  auth
  lcp-echo-interval 10
  lcp-echo-failure 2
  ms-dns 8.8.8.8
  ms-dns 8.8.4.4
  logfile /var/log/pppd.log
```
添加用户信息 vi /etc/ppp/chap-secrets
```shell
# client server secret  IP address
"test" * "test" *
```
启动PPPoE服务
```shell
pppoe-server -I eth0 -L 10.10.10.1 -R 10.10.10.100 -N 100
```
其中：
* -I 后加 监听PPPOE服务的网卡
* -L后加 PPPOE虚拟服务器IP（随意设置）
* -R后加 PPPOE网络IP起始（随意设置）
* -N后加可同时连接数

设置IPv4转发和NAT规则
```shell
echo "net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1" > /etc/sysctl.d/wg.conf && \
sysctl --system \
iptables -t nat -A POSTROUTING -s 10.10.10.0/24 -o eth0 -j MASQUERADE
```

参考：
* [Ubuntu pppoe server](http://jyhshin.pixnet.net/blog/post/47469696-ubuntu-pppoe-server)
* [How to create a PPPoE Server on Ubuntu?](https://poundcomment.wordpress.com/2011/03/30/pppoe-server-on-ubuntu/)
* [Ubuntu上架设PPPoE Server](http://www.cnblogs.com/kungfupanda/p/3268453.html)

## 客户端
```shell
sudo apt install pppoeconf
sudo pppoeconf
```
根据向导输入用户名密码，需要主机与PPPoE服务器联通。

```shell
# 建立PPPoE连接
pon dsl-provider
# 或
pon provider
# 停止PPPoE连接
poff dsl-provider
# 查看日志
plog
```


### 配置账号密码
vi /etc/ppp/chap-secrets
```shell
# Secrets for authentication using CHAP
# client                server          secret                  IP addresses
"user"              *               "password"           *
```

### 修改pppoeconf配置文件

vi /etc/ppp/peers/dsl-provider
或
vi /etc/ppp/peers/provider
```shell
# Minimalistic default options file for DSL/PPPoE connections
noipdefault
defaultroute
replacedefaultroute // 替换原始路由，重要！
hide-password
noauth
persist
plugin rp-pppoe.so eth0 // 此处是网卡接口
usepeerdns
user "user" // 用户名需要保持一致
```

### pppoe-discovery pppoe服务发现
```shell
pppoe-discovery -I eth0
```

## 参考：
* [Linux中PPPOE技术分析](http://blog.csdn.net/lanmolei814/article/details/45770081)
* [Linux 实现多条ADSL负载均衡](http://itfish.net/article/54973.html)
* [Setting Up a PPPoE Connection in Ubuntu via command line](http://ubuntuguide.net/setting-up-pppoe-ubuntu-command)
* [ADSL（PPPOE）接入指南](http://wiki.ubuntu.org.cn/ADSL%EF%BC%88PPPOE%EF%BC%89%E6%8E%A5%E5%85%A5%E6%8C%87%E5%8D%97)
* [ADSLPPPoE](https://help.ubuntu.com/community/ADSLPPPoE)
