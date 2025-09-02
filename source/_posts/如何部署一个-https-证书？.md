---
title: 如何部署一个 https 证书？
date: 2025-05-10 14:01:57
tags: [SSL, HTTPS, Apache, 服务器运维]
categories: [服务器运维]
---
## 在阿里云上部署 ssh 证书的前提条件

- 一台运行于阿里云服务器的网站或者应用
- 域名已正确解析到服务器IP（已经有 http 连接）

## 阿里云SSL证书下载与准备

1. 登录阿里云SSL证书控制台
2. 找到已签发的证书，点击"下载"
3. 选择"Apache"服务器类型下载证书文件
4. 解压下载的压缩包，里面包含:
   - *.pem (证书文件)
   - *.key (私钥文件)

## 配置步骤

### 1. 检查并启用SSL模块

首先确认Apache的SSL模块已启用:

```bash
sudo a2enmod ssl
sudo systemctl restart apache2
```

### 2. 创建证书存储目录

创建专门存储SSL证书的目录:

```bash
sudo mkdir -p /etc/ssl/你的域名
```

例如:

```bash
sudo mkdir -p /etc/ssl/blog.hgyjllk.cn
```

### 3. 上传证书文件

将证书文件上传到服务器。你可以使用scp或其它工具:

```bash
scp 域名_public.crt root@服务器IP:/etc/ssl/你的域名/
scp 域名.key root@服务器IP:/etc/ssl/你的域名/
scp 域名_chain.crt root@服务器IP:/etc/ssl/你的域名/
```

或者直接在服务器上创建这些文件，复制粘贴内容:

```bash
sudo nano /etc/ssl/你的域名/域名_public.crt
# 粘贴证书内容
```

### 4. 创建SSL虚拟主机配置

对于主域名:

```bash
sudo nano /etc/apache2/sites-available/域名-ssl.conf
```

添加如下内容:

```apache
<VirtualHost *:443>
    ServerName 你的域名
    DocumentRoot /var/www/html  # 修改为你的网站根目录
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/你的域名/域名_public.crt
    SSLCertificateKeyFile /etc/ssl/你的域名/域名.key
    SSLCertificateChainFile /etc/ssl/你的域名/域名_chain.crt
    
    <Directory /var/www/html>  # 修改为你的网站根目录
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
    
    ErrorLog ${APACHE_LOG_DIR}/域名-error.log
    CustomLog ${APACHE_LOG_DIR}/域名-access.log combined
</VirtualHost>
```

对于子域名，创建类似的配置文件:

```bash
sudo nano /etc/apache2/sites-available/子域名-ssl.conf
```

内容类似，注意修改ServerName和DocumentRoot。

### 5. 启用SSL站点配置

```bash
sudo a2ensite 域名-ssl.conf
sudo a2ensite 子域名-ssl.conf  # 如果有子域名
sudo systemctl reload apache2
```

### 6. 测试配置文件语法

在重启Apache前检查配置文件语法是否正确:

```bash
sudo apache2ctl configtest
```

### 7. 重启Apache服务

```bash
sudo systemctl restart apache2
```

## 配置HTTP自动跳转HTTPS (可选)

编辑HTTP虚拟主机配置:

```bash
sudo nano /etc/apache2/sites-available/域名.conf
```

添加重定向规则:

```apache
<VirtualHost *:80>
    ServerName 你的域名
    Redirect permanent / https://你的域名/
</VirtualHost>
```

启用配置并重新加载Apache:

```bash
sudo a2ensite 域名.conf
sudo systemctl reload apache2
```

## 常见问题排查

### 1. 403 Forbidden错误

检查网站目录权限:

```bash
sudo chmod -R 755 /var/www/你的网站目录
sudo chown -R www-data:www-data /var/www/你的网站目录
```

### 2. SSL证书不生效

检查证书路径是否正确:

```bash
sudo ls -la /etc/ssl/你的域名/
```

确保证书文件有正确的读取权限:

```bash
sudo chmod 644 /etc/ssl/你的域名/*.crt
sudo chmod 600 /etc/ssl/你的域名/*.key
```

### 3. 配置文件未启用

检查启用的站点:

```bash
ls -la /etc/apache2/sites-enabled/
```

### 4. 查看Apache错误日志

```bash
sudo tail -f /var/log/apache2/error.log
```

## 多域名配置示例

如果你需要在同一服务器上配置多个域名的HTTPS:

1. 为每个域名获取并上传SSL证书
2. 为每个域名创建单独的SSL配置文件
3. 分别启用每个配置

## SSL证书续期维护

阿里云SSL证书通常有效期为三个月。到期前:

1. 在阿里云控制台续期证书
2. 下载新证书
3. 替换服务器上的证书文件
4. 重新加载Apache

```bash
sudo systemctl reload apache2
```

## 注意事项

- 更新网站内容或代码时，SSL配置不会受影响
- 重启服务器后，Apache应自动启动并加载SSL配置
- 使用`reload`而非`restart`可减少网站访问中断

## 命令解释

- `a2enmod`: Apache Enable Module，启用Apache模块
- `a2ensite`: Apache Enable Site，启用网站配置
- `a2dissite`: Apache Disable Site，禁用网站配置
- `systemctl reload`: 重新加载配置但不重启服务
- `systemctl restart`: 完全重启服务
