## 基础

- host: 服务端IP/域名
- pub_wg_port: 服务端对外wg端口号
- pub_mgmt_port: 服务端web管理界面端口
- password: 服务端web管理界面管理密码
- wg网段: 10.17.0.0/24

### wg服务端安装脚本
```
docker run -d \
  --name=wg-easy \
  -e WG_HOST=<host> \
  -e WG_PORT=<pub_wg_port> \
  -e WG_DEFAULT_DNS=8.8.8.8 \
  -e PASSWORD=<password> \
  -e WG_DEFAULT_ADDRESS=10.17.0.x \
  -e WG_ALLOWED_IPS=10.17.0.0/24 \
  -e WG_PERSISTENT_KEEPALIVE=15 \
  -v ~/.wg-easy:/etc/wireguard \
  -p <pub_wg_port>:51820/udp \
  -p <pub_mgmt_port>:51821/tcp \
  --cap-add=NET_ADMIN \
  --cap-add=SYS_MODULE \
  --sysctl="net.ipv4.conf.all.src_valid_mark=1" \
  --sysctl="net.ipv4.ip_forward=1" \
  --restart unless-stopped \
  weejewel/wg-easy
```
## wg客户端安装步骤
1. 安装配置
```
apt-get install wireguard resolvconf

echo "net.ipv4.ip_forward = 1
net.ipv6.conf.all.forwarding = 1" > /etc/sysctl.d/wg.conf && \
sysctl --system
```
2. 从服务端Web管理界面中添加新配置文件，下载配置文件，拷贝到`/etc/wireguard/wg17.conf`
3. 设置系统服务
```
systemctl enable wg-quick@wg17
systemctl start wg-quick@wg17
systemctl status wg-quick@wg17
```