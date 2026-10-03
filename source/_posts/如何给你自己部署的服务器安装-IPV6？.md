---
title: 家里的 Mac 怎么对外提供服务：IPv6 DDNS、反向 SSH 隧道和 Tailscale 怎么选
date: 2025-05-10 14:42:21
tags: [IPV6, Tailscale, SSH隧道, 反向代理, 服务器运维]
categories: [服务器运维]
---

家里的 Mac（或 Mac mini）上跑着网站、接口、后台，想让外面的人访问，该怎么做？家宽通常没有公网 IPv4，办法有好几种。这篇把几种办法放在一起对比：**IPv6 DDNS 直连**（这篇最早写的）、**阿里云反向 SSH 隧道**（我现在对外服务用的）、**Tailscale 组网**（我自己远程管理用的），以及把前两者结合起来的**阿里云加入 Tailscale 再反向代理**。

## 先看结论：几种方法怎么选

| | 方案一：IPv6 DDNS 直连 | 方案二：反向 SSH 隧道 + 云服务器 | 方案三：Tailscale 组网 | 方案四：云服务器加入 Tailscale + 反向代理 |
| --- | --- | --- | --- | --- |
| 适合 | 自己用、临时测试 | **对外提供网站 / 接口 / 小程序后端** | **自己的设备之间互相访问** | **对外提供服务，而且服务比较多** |
| 访问者 | 必须有 IPv6 | 任何人，用域名 https 访问 | 只有登录了同一账号的设备 | 任何人，用域名 https 访问 |
| 是否暴露家里网络 | 是，端口直接开在公网 | 否，只有云服务器的 443 对外 | 否 | 否，只有云服务器的 443 对外 |
| 需要公网 IP / 改光猫 | 需要 IPv6，并关闭光猫防火墙 | 不需要 | 不需要 | 不需要 |
| 家里一侧要做什么 | 开 IPv6、DDNS | 每个服务一个隧道 + launchd 配置 | 装 Tailscale | 装 Tailscale，不用开隧道 |
| 云服务器一侧要做什么 | 无 | Apache 反向代理 + 证书 | 无 | 装 Tailscale + Apache 反向代理 + 证书 |
| 小程序能用吗 | 基本不行 | **可以** | 不行 | **可以** |

一句话：**对外的服务走方案二或方案四，自己人用走方案三，方案一只适合做实验。** 服务少的时候方案二最直接；服务多了，每个服务都要维护一个隧道和端口，可以换成方案四。

## 方案一：IPv6 DDNS 直连

这是我最早的做法：给 Mac 的 IPv6 地址绑一个域名，让别人直接连家里。缺点也很明显：

- 访问者必须有 IPv6，只有 IPv4 的用户根本连不上；
- 要在光猫里关掉 IPv6 防火墙，等于把这台 Mac 的端口直接暴露在公网上，安全风险比较大；
- 家宽的 IPv6 前缀会变，所以要用 DDNS 工具不断更新解析；
- 不少运营商对家宽的 80/443 端口有限制，做网站还要处理 https 和备案。

具体步骤如下。

### 前提条件

1. 运营商提供了IPv6，但没有公网IPv4地址
2. 拥有阿里云的域名（例如：hgyjllk.top）
3. 获取了光猫的管理权限并关闭了IPv6防火墙

#### 一、准备工作

1. **检查IPv6连接**
   - 通过`ifconfig`命令确认Mac上的IPv6地址
   - 确认我们的IPv6地址，例如：`0000:0a00:0000:f000:800:b0a0:0f0:0dcd`

2. **准备阿里云账户**
   - 确保有一个阿里云账户域名或者已注册的域名
   （我的 freenom 快回来吧）

#### 二、创建RAM用户和AccessKey

1. **创建RAM用户**
   - 登录阿里云控制台
   - 进入RAM访问控制
   - 创建用户，设置登录名称和显示名称
   - 选择"使用永久AccessKey访问"

2. **获取AccessKey**
   - 保存生成的AccessKey ID和AccessKey Secret

3. **设置RAM用户权限**
   - 为RAM用户添加系统权限策略"AliyunDNSFullAccess"
   - 这一步非常重要，否则会出现"Forbidden.RAM"错误

#### 三、配置DNS记录

1. **在阿里云添加AAAA记录**
   - 登录阿里云域名控制台
   - 找到域名并进入解析设置
   - 添加一条AAAA记录，主机记录为"max"，记录值为IPv6地址（随你定）
   - 设置TTL值为600秒（10分钟）

#### 四、安装和配置DDNS-Go

1. **下载DDNS-Go**
   - 从GitHub下载适合Mac的ddns-go版本：https://github.com/jeessy2/ddns-go/releases
   - 下载后解压并设置权限：`chmod +x ./ddns-go`

2. **运行DDNS-Go**
   - 执行`./ddns-go`命令启动程序
   - 在浏览器打开 `http://localhost:9876` 进行配置

3. **配置DDNS-Go**
   - 选择"阿里云"服务商
   - 填入AccessKey ID和AccessKey Secret
   - 在IPv6部分填入域名

4. **设置为系统服务（可选）**
   - 执行`sudo ./ddns-go -s install`设置为系统服务
   - 可以添加参数：`-l`指定监听地址，`-f`指定同步间隔（秒）

#### 五、测试和验证

1. **查看日志确认运行状态**
   - 检查DDNS-Go的运行日志
   - 看到类似"你的IP XXXX 没有变化的信息表示正常运行

2. **验证外部访问**
   - 从外部网络（如手机的IPv6网络）尝试访问5900(mac上是 5900远程端口)
   - 确认可以成功连接到Mac上开启的服务

### 排错经验

1. **RAM权限问题**
   - 如果出现"Forbidden.RAM"错误，需要检查RAM用户权限
   - 确保添加了"AliyunDNSFullAccess"系统权限策略

2. **IPv6地址获取问题**
   - DDNS客户端可能通过多种方式获取IPv6地址：网络API或本地网卡
   - 可以在DDNS-Go的日志中确认使用了哪种方式获取IPv6地址

3. **域名解析生效时间**
   - DNS记录更新后可能需要一段时间才能在全球范围内生效
   - TTL值设置较小可以加快更新速度

## 方案二：阿里云反向 SSH 隧道（我现在对外服务用的）

思路：**让家里的 Mac 主动连出去**，在阿里云服务器上开一个只监听本机的端口，阿里云的 Apache 把用户请求转发到这个端口，请求就会顺着隧道回到家里的 Mac。用户访问的是阿里云的域名，看不到家里的网络。

```
用户 ──https──▶ 阿里云（域名 + 证书 + Apache 反向代理）
                      │ 127.0.0.1:18092
                      ▼
                 反向 SSH 隧道（家里 Mac 主动连上来的）
                      │
                      ▼
                 家里 Mac 上的服务（比如 :3000）
```

好处：家里不用公网 IP，也不用改光猫；证书、域名备案都在阿里云上做，小程序要求的 https 备案域名也满足；家里只有一条出站的 SSH 连接。

### 1. 在家里的 Mac 上建立隧道

先在 Mac 上生成一把专用的 SSH 密钥，把公钥放到阿里云服务器上（建议专门建一个低权限用户，不要直接用 root）：

```bash
ssh-keygen -t ed25519 -f ~/.ssh/tunnel_key -N ""
ssh-copy-id -i ~/.ssh/tunnel_key.pub 用户名@阿里云IP
```

然后建立反向隧道，把阿里云本机的 18092 端口转到 Mac 的 3000 端口：

```bash
ssh -i ~/.ssh/tunnel_key \
  -o BatchMode=yes \
  -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=30 \
  -o ServerAliveCountMax=3 \
  -N -R 127.0.0.1:18092:127.0.0.1:3000 \
  用户名@阿里云IP
```

几个参数的作用：

- `-N`：只转发端口，不执行命令；
- `-R 127.0.0.1:18092:127.0.0.1:3000`：在阿里云上监听 `127.0.0.1:18092`，转发到 Mac 的 `127.0.0.1:3000`。绑定在 `127.0.0.1` 意味着只有阿里云本机能访问，外网访问不到这个端口；
- `ExitOnForwardFailure=yes`：端口被占用等原因导致转发失败时直接退出，而不是假装连上了；
- `ServerAliveInterval=30` 和 `ServerAliveCountMax=3`：每 30 秒发一次心跳，连续 3 次没响应就断开，这样网络波动后进程会退出，交给下面的 launchd 重新拉起。

### 2. 用 launchd 让隧道开机自启、断线重连

手动开隧道一断就没了。macOS 上用 launchd 守护，写一个 `~/Library/LaunchAgents/com.example.tunnel.plist`：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key><string>com.example.tunnel</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/bin/ssh</string>
    <string>-i</string><string>/Users/你的用户名/.ssh/tunnel_key</string>
    <string>-o</string><string>BatchMode=yes</string>
    <string>-o</string><string>ExitOnForwardFailure=yes</string>
    <string>-o</string><string>ServerAliveInterval=30</string>
    <string>-o</string><string>ServerAliveCountMax=3</string>
    <string>-N</string>
    <string>-R</string><string>127.0.0.1:18092:127.0.0.1:3000</string>
    <string>用户名@阿里云IP</string>
  </array>
  <key>RunAtLoad</key><true/>
  <key>KeepAlive</key><true/>
</dict>
</plist>
```

加载并启动：

```bash
launchctl bootstrap gui/$(id -u) ~/Library/LaunchAgents/com.example.tunnel.plist
launchctl list | grep com.example.tunnel   # 看到进程说明在跑
```

`KeepAlive` 会在进程退出后自动重启，所以断网恢复后隧道会自己重连。我现在家里的 Mac mini 上，每个对外的服务都有一个这样的 `.tunnel.plist`，各用一个端口，互不影响。

### 3. 在阿里云上配置 Apache 反向代理和证书

先启用代理模块：

```bash
sudo a2enmod proxy proxy_http ssl
sudo systemctl restart apache2
```

给应用域名建一个站点配置，把请求转给隧道端口：

```apache
<VirtualHost *:80>
    ServerName app.example.com
    ProxyPreserveHost On
    ProxyPass / http://127.0.0.1:18092/
    ProxyPassReverse / http://127.0.0.1:18092/
</VirtualHost>
```

启用站点后用 Certbot 自动签发 https 证书并开启跳转，详见[这篇](/2025/07/22/如何使用Let-s-Encrypt-Certbot提供完全免费的https证书/)：

```bash
sudo a2ensite app.example.com.conf && sudo systemctl reload apache2
sudo certbot --apache -d app.example.com
```

### 4. 验证

```bash
# 在阿里云上：隧道端口应该由 sshd 监听
ss -ltnp | grep 18092
# 在任意设备上：域名应该能打开家里的服务
curl -I https://app.example.com
```

### 常见问题

- **访问返回 502：** 先看阿里云上 `ss -ltnp | grep 18092` 有没有监听，没有说明隧道断了，在 Mac 上用 `launchctl list` 看进程；有监听但 502，说明家里 Mac 上的服务本身没起来；
- **改了 plist 不生效：** 先 `launchctl bootout gui/$(id -u)/com.example.tunnel`，再重新 `bootstrap`；
- **一个服务一个端口：** 多个服务就用 18093、18094 这样往后排，不要混用。

## 方案三：Tailscale（自己的设备互访用这个）

方案二是给**别人**访问用的。如果只是我自己想从外面访问家里的 Mac、NAS、Windows 电脑（SSH、远程桌面、传文件、打开内部的管理后台），用 **Tailscale** 最省事：它把你的所有设备放进同一个虚拟内网，每台设备有一个 `100.x.x.x` 的固定地址，不管在哪、在什么网络下，互相之间都能直连，不需要公网 IP，也不用改光猫或路由器。

### 怎么用

1. 在每台设备上安装 Tailscale，用同一个账号登录；
2. 在 Tailscale 管理页或客户端里能看到每台设备的 `100.x.x.x` 地址；
3. 之后直接用这个地址访问，比如：

```bash
ssh 用户名@100.x.x.x
```

给常用设备在 `~/.ssh/config` 里起个别名，以后只要 `ssh macmini`：

```
Host macmini
    HostName 100.x.x.x
    User 你的用户名
    IdentityFile ~/.ssh/某把密钥
```

跟方案一、二相比：设备不对公网开放，只有登录了你账号的设备才进得来，比方案一安全得多；也不需要自己搭服务器，但**别人访问不了**，所以它不适合做对外的网站。

### 一个坑：Clash 等代理会截走 Tailscale 的内网地址

如果 Mac 上开着 Clash 之类的系统代理，访问 `100.x.x.x` 的请求会被代理截走，表现为莫名其妙的 502 或连接失败，很容易误判成服务端的问题。解决办法是让代理绕过 Tailscale 的网段 `100.64.0.0/10`：

```bash
# 终端工具：在 shell 配置里把 Tailscale 网段加入 NO_PROXY
export NO_PROXY="localhost,127.0.0.1,::1,.local,100.64.0.0/10"
```

浏览器走的是 macOS 的系统代理，不认 `NO_PROXY`，要改系统的绕过列表：

```bash
networksetup -getproxybypassdomains Wi-Fi          # 先看当前的绕过列表
networksetup -setproxybypassdomains Wi-Fi "原有条目..." "100.*"   # 把原有条目和 100.* 一起写回，会整体覆盖
```

改完关掉旧的浏览器标签页重新打开，已经建立的连接不会自动重连。

## 方案四：阿里云加入 Tailscale，用 100.x 地址反向代理

方案二里，每多一个服务就要多开一条隧道、多占一个端口、多一个 launchd 文件。如果阿里云服务器也装上 Tailscale，和家里的 Mac 在同一个虚拟内网里，Apache 就可以**直接用 Mac 的 `100.x.x.x` 地址**做反向代理，不再需要隧道。

```
用户 ──https──▶ 阿里云（域名 + 证书 + Apache 反向代理）
                      │ 走 Tailscale 内网（加密）
                      ▼
                 家里 Mac 的 100.x.x.x:3000
```

### 1. 在阿里云服务器上安装并加入 Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
tailscale status      # 能看到家里的 Mac
tailscale ip -4       # 本机的 Tailscale 地址
```

`tailscale up` 会给出一个登录链接，在浏览器里用同一个账号登录授权即可。服务器是长期在线的设备，可以在 Tailscale 管理后台的 Machines 页面里给它关闭密钥过期（Disable key expiry），避免密钥过期后突然掉线。官方也提醒这会降低一点安全性，所以只对可信的服务器这样做，设备丢失或更换时要立刻撤销密钥。

### 2. 家里的 Mac 上让服务监听 Tailscale 能访问到的地址

Mac 上同样装好 Tailscale，并查到它的地址：

```bash
tailscale ip -4       # 例如 100.x.x.x
```

服务不能只监听 `127.0.0.1`，要监听 `0.0.0.0` 或 Tailscale 的地址，阿里云才能通过 `100.x.x.x:端口` 访问到。注意只让它在 Tailscale 内网里可见，不要把这个端口映射到公网。

### 3. Apache 直接代理到 Mac 的 Tailscale 地址

```apache
<VirtualHost *:80>
    ServerName app.example.com
    ProxyPreserveHost On
    ProxyPass / http://100.x.x.x:3000/
    ProxyPassReverse / http://100.x.x.x:3000/
</VirtualHost>
```

之后和方案二一样，用 `certbot --apache -d app.example.com` 签发证书。以后新增一个服务，只要在 Mac 上多起一个端口、在 Apache 里多加一个站点配置，不用再建隧道。

### 和方案二比

| | 方案二：反向 SSH 隧道 | 方案四：云服务器加入 Tailscale |
| --- | --- | --- |
| 新增一个服务 | 新建一条隧道 + 一个 launchd 文件 + 一个端口 | 只加 Apache 配置 |
| 云服务器上要装什么 | 不用装额外软件 | 要装 Tailscale |
| 断线重连 | 靠 `ssh` 的心跳 + launchd `KeepAlive` | 靠 Tailscale 自己处理 |
| 依赖 | 只依赖 SSH | 依赖 Tailscale 的服务（登录、打洞、中继） |
| 云服务器能反过来访问家里 | 只能访问隧道映射出来的端口 | 能访问家里设备上任何开放的端口 |

方案四更省事，但把云服务器和家里都拴在 Tailscale 上；方案二只依赖 SSH，更朴素，出问题也更好排查。服务不多时我继续用方案二。

### 为什么不用 Tailscale 自带的 Funnel

Tailscale 有一个叫 **Funnel** 的功能，可以直接把家里的服务暴露到公网，不需要云服务器。但官方文档里写明了几个限制，导致它不适合给小程序或自己的域名用：

- 只能使用你 tailnet 的 `xxx.ts.net` 域名，**不支持自己的域名**，而小程序要求的是已备案的自有域名；
- 只能用 443、8443、10000 这三个端口；
- 流量有不可配置的带宽限制。

所以对外服务还是需要一台有备案域名的云服务器做入口，Funnel 适合临时给朋友看一下 Demo。

## 我的组合用法

- **对外的网站、接口、小程序后端：** 方案二。家里的 Mac mini 通过反向隧道连到阿里云，阿里云出域名和 https。服务多起来的话，可以改用方案四，省掉每个服务一条隧道的维护；
- **我自己管理、传文件、远程操作：** 方案三。Mac、NAS、Windows 电脑都在 Tailscale 里，直接用 `100.x` 地址 SSH；
- **方案一：** 现在基本不用了，只在测试 IPv6 的时候会用。

