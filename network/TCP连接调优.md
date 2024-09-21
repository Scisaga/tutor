[RFC973: TCP协议](https://datatracker.ietf.org/doc/html/rfc793)

## FINWAIT-2状态
在TCP四次挥手中， A端主动关闭发FIN，然后收到B端的ACK，B端进入CLOSE_WAIT状态。收到B端的ACK后，A端进入FINWAIT-2状态，等待B端发FIN包。B端的CLOSE_WAIT的理论最大时间是无穷大，所以，A端理论上可能持续无穷大的时间才有机会获取到B端的FIN，因此，TCP四次挥手FINWAIT-2状态的理论最大时间是无穷大。

实际场景
- 客户端始发方向关闭连接，而服务端依然在传输大量数据这种情形，似乎几乎是没有的。相反，几乎很多的C/S模式的TCP连接都是单向的，比如文件下载，至始至终，数据几乎都是从服务器往客户端发送，在最初的客户端请求文件结束后，事实上就相当于客户端始发方向的连接已经被关闭了。

所以
* 标准的要求：收到对端的FIN之前，本端会一直保持FINWAIT-2状态
* 现实的要求：收到对端的FIN之前，本端会保持FINWAIT-2状态一段足够的时间，超过此时间，连接即释放 
   * 连接在FINWAIT-2超时后并不会进入TIMEWAIT状态，也不会发送reset，而是直接默默消失。
   * net.ipv4.tcp_fin_timeout 控制超时时间

参考：
* [一个TCP FIN_WAIT2状态细节引发的感慨](https://blog.csdn.net/dog250/article/details/81256550)
* [关于FIN_WAIT2](https://huoding.com/2016/09/05/542)
* [Why are connections in FIN_WAIT2 state not closed by the Linux kernel?](https://serverfault.com/questions/738300/why-are-connections-in-fin-wait2-state-not-closed-by-the-linux-kernel)

## 大量TIME_WAIT
```shell
# vi /etc/sysctl.conf
# Controls the use of TCP syncookies
net.ipv4.tcp_syncookies = 1

# The TIME-WAIT sockets for new connections can be reused
net.ipv4.tcp_tw_reuse = 1

# Enable fast recycling of TIME-WAIT sockets status
net.ipv4.tcp_tw_recycle = 1

# Decrease the time default value for tcp_fin_timeout connection
net.ipv4.tcp_fin_timeout = 15
```

使参数生效
sysctl -p

参考：
* [大量TIME_WAIT解决办法](http://coolnull.com/3605.html)
* [理解TIME_WAIT，彻底弄清解决TCP: time wait bucket table overflow](https://blog.51cto.com/benpaozhe/1767612)
* [IP TIME_WAIT状态原理](https://elf8848.iteye.com/blog/1739571)
* [TIME_WAIT and “port reuse”](http://blog.davidvassallo.me/2010/07/13/time_wait-and-port-reuse/)

### 强制关闭处于TIME_WAIT状态的连接
```shell
/etc/init.d/networking restart
```
参考：
* https://serverfault.com/questions/329845/how-to-forcibly-close-a-socket-in-time-wait
