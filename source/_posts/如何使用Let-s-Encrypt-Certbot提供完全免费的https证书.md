---
title: 如何使用Let's Encrypt + Certbot提供完全免费的https证书
date: 2025-07-22 16:06:18
tags: [SSL, HTTPS, Let's Encrypt, Apache, 服务器运维]
categories: [服务器运维]
---

# 如何使用Let's Encrypt + Certbot提供完全免费的https证书

## 背景

阿里云的SSL证书到期了，续费需要付费。经过研究发现Let's Encrypt + Certbot可以提供完全免费且自动续期的SSL证书，完美替代付费方案。

## 阿里云SSL的缺点

- **付费续期**：证书到期后需要支付续费费用
- **手动更新**：需要手动下载证书并更新服务器配置
- **管理复杂**：多个域名需要分别购买和管理
- **有效期短**：免费证书有效期只有 90 天（3 个月），到期要手动下载并更新；付费证书也要定期续费

## Let's Encrypt + Certbot的优点

- **完全免费**：证书永久免费，无任何费用
- **自动续期**：证书有效期 90 天，Certbot 会在到期前自动续期，无需人工干预
- **自动配置**：一条命令自动配置Apache/Nginx
- **批量管理**：可以同时管理多个域名
- **权威认证**：被所有主流浏览器信任

## 实施步骤

### 1. 检查当前SSL配置状态

```bash
# 查看Apache SSL配置
sudo ls -la /etc/apache2/sites-enabled/ | grep ssl

# 检查现有Let's Encrypt证书
sudo certbot certificates
```

### 2. 安装Certbot

Certbot 官方现在推荐用 snap 安装，这样能一直拿到官方维护的最新版，并且自动更新：

```bash
# Ubuntu/Debian 系统
sudo apt update
sudo apt install snapd
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

如果系统不方便用 snap，也可以用系统源安装，但版本会比较旧：

```bash
# Ubuntu/Debian 系统（系统源，版本较旧）
sudo apt install certbot python3-certbot-apache
```

> 原来这里还写了 CentOS 的 `yum` 命令。CentOS 已经停止维护，如果用的是 Rocky Linux、AlmaLinux 等 RHEL 系发行版，请参考 [Certbot 官方安装说明](https://certbot.eff.org/instructions)。

### 3. 从阿里云SSL切换到Let's Encrypt

**识别问题**：通过`curl -I https://yourdomain.com`发现证书过期错误：
```
curl: (60) SSL certificate problem: certificate has expired
```

**检查配置文件**：发现Apache使用的是过期的阿里云证书路径：
```apache
SSLCertificateFile /etc/ssl/aliyun/domain_public.crt
SSLCertificateKeyFile /etc/ssl/aliyun/domain.key
SSLCertificateChainFile /etc/ssl/aliyun/domain_chain.crt
```

### 4. 批量申请Let's Encrypt证书

```bash
# 为单个域名申请证书并自动配置Apache
sudo certbot --apache -d yourdomain.com

# 为多个域名同时申请
sudo certbot --apache -d blog.domain1.com -d api.domain2.com -d www.domain3.com

# 强制重新申请（如果已有证书但配置有问题）
sudo certbot --apache -d yourdomain.com --force-renewal
```

**选择重定向选项**：
- 选择 `2: Redirect` 强制HTTPS访问
- 自动添加HTTP到HTTPS的重定向规则

### 5. 配置自动续期

```bash
# 检查自动续期状态
sudo systemctl status certbot.timer
sudo systemctl list-timers | grep certbot

# 启用系统定时器（推荐方式）
sudo systemctl enable certbot.timer
sudo systemctl start certbot.timer

# 测试自动续期功能
sudo certbot renew --dry-run

# 备选：使用cron任务
sudo crontab -e
# 添加：0 12 * * * /usr/bin/certbot renew --quiet
```

## 配置验证

### 检查证书状态
```bash
# 查看所有证书信息
sudo certbot certificates

# 测试HTTPS连接
curl -I https://yourdomain.com

# 检查HTTP重定向
curl -I http://yourdomain.com
```

### 预期结果
```
HTTP/1.1 200 OK
Server: Apache/2.4.41
Strict-Transport-Security: max-age=31536000
```

这样就能拥有一个非常不错的完全免费且自动续期的SSL证书

## 补充：证书有效期会越来越短（2026 年更新）

行业规则正在缩短 SSL 证书的有效期，Let's Encrypt 已经公布了时间表：

- 2027 年 2 月 10 日起，默认证书有效期从 90 天缩短为 **64 天**；
- 2028 年 2 月 16 日起，进一步缩短为 **45 天**。

这只影响新签发的证书，Certbot 自动续期会照常工作。所以更要确认自动续期是正常的（上面第 5 步的 `certbot renew --dry-run`），**不要依赖手动更新证书**。