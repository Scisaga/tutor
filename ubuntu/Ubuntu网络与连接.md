## 基本命令
```shell
# 查看所有网络接口的属性
ip a
iptables -L            # 查看防火墙设置
route -n               # 查看路由表
netstat -lntp          # 查看所有监听端口
netstat -antp          # 查看所有已经建立的连接
netstat -s             # 查看网络统计信息
ethtool eth0           # 查看网卡连接速度
```

### 设置静态IP
ubuntu <= 16.04：修改/etc/network/interfaces
```shell
auto enp2s0
iface enp2s0 inet static
address 10.0.0.18
netmask 255.255.255.0
gateway 10.0.0.1
```
ubuntu >= 18.04：配置 netplan

### 查看当前 DNS 服务器
Ubuntu >= 15
```shell
nmcli device show eth0 | grep IP4.DNS
```

Ubuntu <= 14
```shell
nmcli dev list iface eth0 | grep IP4
```

### 测试 DNS 解析
```shell
nslookup [-option ...] domain-name [dns-server]
nslookup isc.org 8.8.8.8
```

### 配置 MAC 地址
列出本地网络所有设备的MAC地址
```shell
sudo nmap -sn 192.168.1.0/24 // 对D段进行遍历
arp -n
sudo arp-scan -l
sudo arp-scan --interface=eth0 --localnet
```
修改MAC地址
```shell
sudo ifconfig eth0 down
sudo ifconfig eth0 hw ether xx:xx:xx:xx:xx:xx
sudo ifconfig eth0 up
```

### 实时网速统计
```shell
sudo apt install nethogs
sudo nethogs

```

参考：
* [从零开始学习iftop流量监控（找出服务器耗费流量最多的ip和端口）](https://www.cnblogs.com/chenqionghe/p/10680075.html)

## 远端测试
### 测试远程端口是否开放
```shell
nmap <IP> -p <PORT>
```

### 网速测试
```shell
sudo apt-get install iperf
# 服务端
iperf -s
# 客户端
iperf -c <服务端IP>
```

使用netcat
```shell
# 服务端
nc -vvlnp 12345 >/dev/null
# 客户端
dd if=/dev/zero bs=1M count=1K | nc -vvn <服务端IP> 12345
```

参考：
* http://askubuntu.com/questions/7976/how-do-you-test-the-network-speed-betwen-two-boxes

## SSH端口转发

端口转发 port forwarding 只需要与服务器对应的端口建立 TCP 连接即可，主要分：
* 本地端口转发
* 远程端口转发
* 动态端口转发

```shell
# 将本地4000端口转发的到example.com网段127.0.0.1主机的3306
ssh -L 4000:127.0.0.1:3306 user@example.com

# 将example.com的7000端口绑定到本地网段127.0.0.1的8000端口（端口转发）
ssh -R 7000:127.0.0.1:8000 user@example.com

# 在本地主机创4000端口创建了socks代理，发送到该端口的所有请求都会通过example.com转发
ssh -D 4000 user@example.com
```

参考：
* [https://lvii.github.io/system/2013-10-08-ssh-remote-port-forwarding/ SSH 远程端口转发]
* [https://www.booleanworld.com/guide-ssh-port-forwarding-tunnelling/ A Guide to SSH Port Forwarding/Tunnelling]
* [https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/ 实战 SSH 端口转发]
* [https://hackertarget.com/ssh-examples-tunnels/ SSH Examples, Tips & Tunnels]

### SSH 内网端口暴露
外网机 
```shell
# vi /etc/ssh/sshd_config
GatewayPorts clientspecified
# 重启ssh服务
sudo service ssh restart
```

内网机
```shell
ssh -N -f -R [bind_address:]port:host:host_port
# 将外网机的60000端口映射到内网机的22端口上
ssh -N -f -R 0.0.0.0:60000:localhost:22 root@remote-host
```

### 指定 network interface
使用-b参数
```shell
# 其中192.168.0.100是特定interface对应的本地ip地址
ssh -b 192.168.0.100 user@10.75.0.5
```

### autossh 增强稳定性
在/etc/rc.local中添加
```shell
su - <user> -c "autossh -N -f -M 10984 -o \"PubkeyAuthentication=yes\" -o \"PasswordAuthentication=no\" -i <user_private_key_path> -R 0.0.0.0:60000:localhost:22 root@<remote-host> -p <remote_ssh_port>"
```
注：使用autossh前使用ssh执行对应指令进行测试

参考：
* [ssh -D -L -R 差异](http://www.cnblogs.com/-chaos/p/3378564.html)
* [使用 ssh -R 建立反向/远程TCP端口转发代理](https://yq.aliyun.com/articles/8469)
* [Bypassing corporate firewall with reverse ssh port forwarding](https://toic.org/blog/2009/reverse-ssh-port-forwarding/)
* [Persistent reverse (NAT bypassing) SSH tunnel access with autossh](https://raymii.org/s/tutorials/Autossh_persistent_tunnels.html)
* http://myhomelab.blogspot.com/2012/12/ssh-tunneling.html
* http://bbrinck.com/post/2318562750/reverse-ssh-tunneling-easier-than-port
* http://serverfault.com/questions/370270/autossh-not-working-for-two-or-more-tunnels-or-is-there-an-alternative
* Autossh 用户手册：http://www.manpagez.com/man/1/autossh/
* [autossh for Persistent Reverse ssh Tunnels](https://akntechblog.wordpress.com/2010/09/11/autossh-for-persistent-reverse-ssh-tunnels/)
* https://github.com/jonhiggs/autossh