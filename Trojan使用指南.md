# 1 安装Trojan客户端
```bash
sudo apt-get -y install trojan
```

# 2 配置 `config.json` 文件
```bash
vi /etc/bin/trojan/config.json
```
按如下方式进行配置：
- `run_type`：表示运行模式，指定填为 "client"
- `local_port`：表示本地端口，例如 1080
- `remote_addr`：表示远程服务器地址，填服务器IP，例如 "219.22.49.201"
- `remote_port`：表示远程服务器端口，如 22201
- `password`：表示远程服务器的节点访问密码
- `ssl`：
	- `sni`：表示主机SNI名，如 "jpo123.ovod.me"，对应节点内容里的"服务器SNI"（如果配置文件中没有，则添加这个配置）
	- `verify`：将值修改为 false（如果配置文件中没有，则添加这个配置）
	- `verify_hostname`：值修改为 false（如果配置文件中没有，则添加这个配置）
	- `cert`：将值修改为 ""（改为空值）
	- `key`：将值修改为 ""（改为空值）

# 3 运行 Trojan 服务

使用以下命令配置 Trojan 服务

```bash
cat <<EOF > /etc/systemd/system/trojan.service
[Unit]
Description=trojan
After=network.target

[Service]
Type=simple
PIDFile=/etc/bin/trojan/trojan.pid
ExecStart=trojan -c /etc/bin/trojan/config.json -l /etc/bin/trojan/trojan.log
ExecReload=/bin/kill -HUP \$MAINPID
Restart=on-failure
RestartSec=1s

[Install]
WantedBy=multi-user.target
EOF
```
    
使用以下命令启动 Trojan 服务  
```bash
systemctl start trojan
```

通过如下命检查 Trojan 服务是否启动成功，如果在控制台看到有类似 `trojan` 的内容展示，即表示正在运行  Trojan 服务，如果未启动成功，可以通过 `cat /usr/src/trojan/trojan.log` 命令查看日志。
``` bash
ps aux | grep trojan | grep -v grep  
```

使用以下命令将 Trojan 服务设置为开机自启动
```bash
systemctl enable trojan
```

# 在 Chrome/Firefox 浏览器中使用 Trojan 代理

1. 在 Chrome 应用商店 或者 Firefox 拓展商店中下载 SwitchyOmega
2. 配置SwitchyOmega中的 `proxy` 代理使浏览器可以连接本地的 `1080` 端口，其中：
   - 协议：`SOCKS5`
   - 代理服务器：`127.0.0.1`
   - 端口：`1080`
   配置完毕后在浏览器地址栏输入 google.com，查看是否可以正常打开
