## route
查看路由设置
```shell
route -n
```
添加到特定主机（10.0.0.50）的路由，使用10.0.0.1网关
```shell
sudo route add -host 10.0.0.50 gw 10.0.0.1
```
说明，当主机在连接本地网络的基础上成功进行了pppoe拨号后，会同时存在两个网关，会导致无法正常将数据包转发到本地网络的其他主机，此时需要手动添加到特定主机的路由

删除路由
```shell
sudo route del -host 10.0.0.50 gw 10.0.0.1
```

## 设定多个Gateway
增加routing table：
```shell
vi /etc/iproute2/rt_tables
```
最后一行添加：
```shell
1 rt2
```
其中1代表优先级第一位。

测试命令：
```shell
# 192.168.8.0/24 为第二个网口对应的网段，192.168.8.56 为第二个网口对应的ip地址
ip route add 192.168.8.0/24 dev enp2s0 src 192.168.8.56 table rt2
# enp2s0 为第二个网口名称，192.168.8.1 第二个网口对应网关
ip route add default via 192.168.8.1 dev enp2s0 table rt2
ip rule add from 192.168.8.56/32 table rt2
ip rule add to 192.168.8.56/32 table rt2
```

netplan:
```shell
network:
    ethernets:
        enp0s3:
            dhcp4: false
            addresses: [192.168.1.202/24]
            gateway4: 192.168.1.1
            nameservers:
              addresses: [8.8.8.8,8.8.4.4,192.168.1.1]
            routes:
            - to: 172.16.0.0/24
              via: 192.168.1.100
    version: 2
```

~~ interface配置： ~~
```shell
iface enp2s0 inet static
address 192.168.8.56
netmask 255.255.255.0
post-up ip route add 192.168.8.0/24 dev enp2s0 src 192.168.8.56 table rt2
post-up ip route add default via 192.168.8.1 dev enp2s0 table rt2
post-up ip rule add from 192.168.8.56/32 table rt2
post-up ip rule add to 192.168.8.56/32 table rt2
```

参考：
* [Routing for multiple uplinks/providers](http://tldp.org/HOWTO/Adv-Routing-HOWTO/lartc.rpdb.multiple-links.html)
* [Two Default Gateways on One System](https://www.thomas-krenn.com/en/wiki/Two_Default_Gateways_on_One_System)
* [how to configure 2 network interfaces with different gateways](https://askubuntu.com/questions/868942/how-to-configure-2-network-interfaces-with-different-gateways)
* [Two interfaces, two addresses, two gateways?](https://unix.stackexchange.com/questions/22770/two-interfaces-two-addresses-two-gateways)
* [How to add static route with netplan on Ubuntu 20.04 Focal Fossa Linux](https://linuxconfig.org/how-to-add-static-route-with-netplan-on-ubuntu-20-04-focal-fossa-linux)
* [netplan configuration - add static route](https://askubuntu.com/questions/1434760/netplan-configuration-add-static-route)

## iptables 

配置IPV4转发
```shell
echo "net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1" > /etc/sysctl.d/wg.conf && \
sysctl --system
```
查看现有配置信息
```shell
iptables -L
```
清空设置
```shell
sudo iptables -t nat -F POSTROUTING && \
sudo iptables -t nat -F PREROUTING && \
sudo iptables -t nat -F OUTPUT && \
sudo iptables -t nat -F && \
sudo iptables --delete-chain && \
sudo iptables -F
```

### 内网端口映射

```shell
# 设置端口映射（50:60221 --> 21:59998）
# 删除映射配置，将上述语句中的 -A 改为 -D
sudo iptables -t nat -A PREROUTING -d 10.0.0.50 -p tcp --dport 60221 -j DNAT --to-destination 10.0.0.21:59998 && \
sudo iptables -t nat -A POSTROUTING -d 10.0.0.21 -p tcp --dport 59998 -j SNAT --to 10.0.0.50

# 查看设置
sudo iptables -L -t nat

# 保存当前配置到文件
sudo iptables-save > iptables

# 此时可以在规则文件中直接修改，再导入配置文件，可以提升效率
sudo iptables-restore < iptables
```
### 参考
* [linux系统中查看己设置iptables规则](https://ivanzz1001.github.io/records/post/linuxops/2018/10/17/linux-iptables)
