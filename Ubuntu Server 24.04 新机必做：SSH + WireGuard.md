![image.png](https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/d02a2f1407d34b6781e71e8f4a73ba7f~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAgc2Npc2FnYQ==:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiMTAyMTg4OTc3NDM2NTA4OCJ9&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1770286118&x-orig-sign=sHWLDbcuChqmUXM3Kbv4uQjMqFM%3D)

## 1) 下载 Ubuntu Server 24.04 镜像（ISO）

### 选哪个下载入口？

*   **推荐入口 A：Ubuntu 官方下载页**（更直观）([Ubuntu](https://ubuntu.com/download/server "
    Get Ubuntu Server | Download
    \| Ubuntu"))
*   **推荐入口 B：releases.ubuntu.com（Noble / 24.04.x 列表）** （可以看到 ISO 文件名、SHA256SUMS 等）([Ubuntu 发行版](https://releases.ubuntu.com/noble "Ubuntu 24.04.3 (Noble Numbat)"))

### 选哪个 ISO？

一般 PC/服务器（Intel/AMD 64 位）请选择：

*   `ubuntu-24.04.x-live-server-amd64.iso`（Server install image）([Ubuntu 发行版](https://releases.ubuntu.com/noble "Ubuntu 24.04.3 (Noble Numbat)"))

> 说明：Server install image 默认不带图形界面（GUI），面向服务器用途。([Ubuntu 发行版](https://releases.ubuntu.com/noble "Ubuntu 24.04.3 (Noble Numbat)"))

### （可选）校验 SHA256

在 `releases.ubuntu.com/noble/` 页面能看到 `SHA256SUMS` 文件。下载后用 Windows PowerShell 校验：

    Get-FileHash .\ubuntu-24.04.x-live-server-amd64.iso -Algorithm SHA256

把输出的 Hash 与 `SHA256SUMS` 对应条目比对即可。([Ubuntu 发行版](https://releases.ubuntu.com/noble "Ubuntu 24.04.3 (Noble Numbat)"))

***

## 2) 用 Rufus 把 ISO 刻录到 U 盘（Windows）

Ubuntu 官方有一份专门讲 **Windows + Rufus** 的制作启动盘教程，流程大体如下。([Ubuntu](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows "
Create a bootable USB stick with Rufus on Windows
\| Ubuntu"))

### 准备

*   U 盘：建议 ≥ 8GB（官方教程写 ≥ 4GB 也可，但现在更推荐 8GB 起）([Ubuntu](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows "
    Create a bootable USB stick with Rufus on Windows
    \| Ubuntu"))
*   Rufus：从 rufus.ie 获取（Ubuntu 教程里也直接引用）([Ubuntu](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows "
    Create a bootable USB stick with Rufus on Windows
    \| Ubuntu"))

### Rufus 操作步骤（要点版）

1.  插入 U 盘，运行 Rufus（有时无需安装，直接运行）。

2.  **Device**：确认选中 U 盘（别选错盘）。([Ubuntu](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows "
    Create a bootable USB stick with Rufus on Windows
    \| Ubuntu"))

3.  **Boot selection**：选择下载的 `ubuntu-24.04.x-live-server-amd64.iso`。([Ubuntu](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows "
    Create a bootable USB stick with Rufus on Windows
    \| Ubuntu"))

4.  分区方案怎么选（经验法则）

    *   机器是 **UEFI**：通常选 **GPT**
    *   机器是老 BIOS/Legacy：通常选 **MBR**\
        （不确定就先用 UEFI+GPT，实在无法启动再换）

5.  点 **Start** 开始写入，确认会清空 U 盘数据。([Ubuntu](https://ubuntu.com/tutorials/create-a-usb-stick-on-windows "
    Create a bootable USB stick with Rufus on Windows
    \| Ubuntu"))

> 备注：Rufus 可能会问 “ISO mode / DD mode”。Rufus 官方说明里提到它默认更推荐 ISO mode（同时也允许选 DD）。一般装 Ubuntu 用默认推荐即可。([GitHub](https://github.com/pbatard/rufus/wiki/FAQ "FAQ · pbatard/rufus Wiki · GitHub"))

***

## 3) 安装 Ubuntu Server 24.04 的详细步骤（Subiquity 安装器）

Ubuntu Server 24.04 使用 **Subiquity** 文本界面安装器。官方有 “基础安装” 与 “逐屏说明” 两份文档，下面按实际安装顺序把每屏该怎么选讲清楚。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/basic-server-installation.html "Basic server installation - Ubuntu installation documentation"))

### 3.1 从 U 盘启动

1.  使用网线连接服务器与路由器/交换机
2.  把 U 盘插到目标机器，重启。
3.  进入 BIOS/Boot Menu（常见按键：`Esc / F2 / F10 / F12 / Enter`，不同品牌不同）。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/basic-server-installation.html "Basic server installation - Ubuntu installation documentation"))
4.  选择 U 盘启动，进入安装器语言选择界面。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/basic-server-installation.html "Basic server installation - Ubuntu installation documentation"))

### 3.2 安装器逐屏设置（建议与注意点）

按屏幕顺序：

#### (1) Language selection（语言）

选择安装器语言（同时会作为系统默认语言之一）。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

#### (2) Refresh（安装器更新）

如果提示安装器有更新，建议更新：可获得 bugfix/改进，更新后会回到相同进度继续。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

#### (3) Keyboard（键盘）

选键盘布局；物理机用国标/日标键盘时别选错。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

#### (4) Network（网络）

*   默认会尝试对有线网卡走 DHCP。
*   也可以在这里配置静态 IP、VLAN、Bond 等（安装器仅覆盖 Netplan 的一部分能力）。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

> 建议：如果希望装完立刻能 SSH，安装时最好保证网络能通（至少有默认路由）。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

#### (5) Proxy / Mirror（代理 / 软件源镜像）

没特殊网络要求就跳过/用默认。安装器会尝试基于地理位置选合适镜像源。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

#### (6) Storage（磁盘与分区）

两种常见选择：

*   **新机器/整盘装系统**：勾选 “Use an entire disk”，选目标盘 → Done → Confirm（最省心）。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/basic-server-installation.html "Basic server installation - Ubuntu installation documentation"))
*   **需要自定义分区/多盘/保留数据**：进自定义存储配置（这块复杂，生产环境务必先备份）。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

> ⚠️ 磁盘分区有数据丢失风险，官方文档明确建议安装前备份。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/basic-server-installation.html "Basic server installation - Ubuntu installation documentation"))

#### (7) Identity（用户/主机名）

设置：

*   hostname
*   用户名
*   密码

默认用户会有 sudo 权限。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

#### (8) SSH（强烈建议开启）

这里是关键屏：

*   勾选安装 OpenSSH（安装器明确说明默认系统没有开放端口，常见运维方式是 SSH，所以提供此选项）。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))
*   可从 GitHub / Launchpad 导入 SSH 公钥；导入后通常会默认禁用密码登录（之后可再改）。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

#### (9) Snaps（可选）

有网络时会推荐一些 server snaps。无刚需就不选，后面随时可以装。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

#### (10) Installation logs → Reboot

等待安装完成，选择 Reboot，拔掉 U 盘后从硬盘启动，使用刚才创建的用户名/密码登录。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

***

## 4) 远程访问（SSH）：装完立刻可连 + 装后补装/排障

### 4.1 如果在安装时已勾选 OpenSSH

装完后通常即可用 SSH 连接（前提：网络可达 + 22 端口未被防火墙拦）。安装器在 SSH 屏就支持安装 OpenSSH。([Ubuntu installation](https://canonical-subiquity.readthedocs-hosted.com/en/latest/tutorial/screen-by-screen.html "Screen-by-screen installer walk-through - Ubuntu installation documentation"))

在服务器本机执行，确认 IP：

    ip a
    hostname -I

### 4.2 如果安装时没装 OpenSSH：装后补装

Ubuntu Server 官方文档给出的安装命令：([documentation.ubuntu.com](https://documentation.ubuntu.com/server/how-to/security/openssh-server/ "OpenSSH server - Ubuntu Server documentation"))

    sudo apt update
    sudo apt install -y openssh-server

### 4.3 Windows 端连接方式

Windows 10/11 通常自带 OpenSSH Client，可直接 PowerShell：

    ssh <用户名>@<服务器IP>

首次连接会提示确认主机指纹（yes），然后输入密码或使用密钥。

### 4.4 防火墙（UFW）导致连不上：开放 22 端口

如果启用了 UFW，需要允许 22 端口。Ubuntu Server 防火墙文档示例：([documentation.ubuntu.com](https://documentation.ubuntu.com/server/how-to/security/firewalls/ "Firewall - Ubuntu Server documentation"))

    sudo ufw allow 22
    sudo ufw status

***

## 5) WireGuard：安装 + 配置文件编辑/导入 + 启动与状态（合并版）

### 5.1 安装

安装 WireGuard 与 resolvconf

    sudo apt-get update
    sudo apt-get install -y wireguard resolvconf

开启 IPv4/IPv6 转发（指定的命令）

    echo "net.ipv4.ip_forward = 1
    net.ipv6.conf.all.forwarding = 1" > /etc/sysctl.d/wg.conf && \
    sysctl --system

***

### 5.2 导入/编辑配置文件（wg0.conf）

*   WireGuard 常用约定：接口名 `wg0` 对应配置文件 `/etc/wireguard/wg0.conf`。([Ubuntu 文档](https://documentation.ubuntu.com/server/how-to/wireguard-vpn/common-tasks/ "Common tasks in WireGuard VPN"))

**方式 A：直接用终端编辑器（支持粘贴）**

    sudo mkdir -p /etc/wireguard
    sudo nano /etc/wireguard/wg0.conf

*   在大多数终端里，“粘贴”通常是 **Ctrl+Shift+V**，也可以右键粘贴。nano 本身对粘贴是友好的：

<!---->

    [Interface]
    PrivateKey = <YOUR_PRIVATE_KEY>
    Address = 10.0.0.2/32
    DNS = 8.8.8.8

    [Peer]
    PublicKey = <SERVER_PUBLIC_KEY>
    Endpoint = <SERVER_IP_OR_DOMAIN>:51820
    AllowedIPs = 10.0.0.0/24
    PersistentKeepalive = 25

> `DNS = ...` 这一行由 `wg-quick` 通过 `resolvconf(8)` 设置。([Man7.org](https://man7.org/linux/man-pages/man8/wg-quick.8.html "wg-quick(8) - Linux manual page"))

**方式 B：无需交互编辑，直接整段粘贴写入**

    sudo tee /etc/wireguard/wg0.conf >/dev/null <<'EOF'
    [Interface]
    PrivateKey = <YOUR_PRIVATE_KEY>
    Address = 10.0.0.2/32
    DNS = 8.8.8.8

    [Peer]
    PublicKey = <SERVER_PUBLIC_KEY>
    Endpoint = <SERVER_IP_OR_DOMAIN>:51820
    AllowedIPs = 10.0.0.0/24
    PersistentKeepalive = 25
    EOF

***

### 5.3 启动 / 停止 / 开机自启 / 查看状态

`wg-quick` 可以直接 up/down，也可以用 systemd 服务 `wg-quick@wg0` 管理（Ubuntu Server 文档明确支持这种用法）。([Ubuntu 文档](https://documentation.ubuntu.com/server/how-to/wireguard-vpn/common-tasks/ "Common tasks in WireGuard VPN"))

**立即启动 / 停止：**

    sudo wg-quick up wg0
    sudo wg-quick down wg0

**设为开机自启 + 立即启动：**

    sudo systemctl enable wg-quick@wg0
    sudo systemctl start wg-quick@wg0

**查看服务状态：**

    systemctl status wg-quick@wg0

> 可能会看到 `active (exited)`：这是因为 `wg-quick` 完成接口配置后退出，systemd 仍认为服务成功执行（常见且通常正常）。([Ubuntu 文档](https://documentation.ubuntu.com/server/how-to/wireguard-vpn/common-tasks/?utm_source=chatgpt.com "Common tasks in WireGuard VPN"))

**查看 WireGuard 握手/流量等运行状态：**

    sudo wg
    sudo wg show
    ip a show wg0

**看失败日志（排障必用）：**

    journalctl -u wg-quick@wg0 -b --no-pager
