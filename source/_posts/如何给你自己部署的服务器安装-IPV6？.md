---
title: 如何给你自己部署的服务器安装 IPV6？
date: 2025-05-10 14:42:21
tags: [IPV6, 服务器运维]
categories: [服务器运维]
---
# 在Mac上配置IPv6 DDNS实现公网访问

## 前提条件

1. 运营商提供了IPv6，但没有公网IPv4地址
2. 拥有阿里云的域名（例如：hgyjllk.top）
3. 获取了光猫的管理权限并关闭了IPv6防火墙

## 步骤总结

### 一、准备工作

1. **检查IPv6连接**
   - 通过`ifconfig`命令确认Mac上的IPv6地址
   - 确认我们的IPv6地址，例如：`0000:0a00:0000:f000:800:b0a0:0f0:0dcd`

2. **准备阿里云账户**
   - 确保有一个阿里云账户域名或者已注册的域名
   （我的 freenom 快回来吧）

### 二、创建RAM用户和AccessKey

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

### 三、配置DNS记录

1. **在阿里云添加AAAA记录**
   - 登录阿里云域名控制台
   - 找到域名并进入解析设置
   - 添加一条AAAA记录，主机记录为"max"，记录值为IPv6地址（随你定）
   - 设置TTL值为600秒（10分钟）

### 四、安装和配置DDNS-Go

1. **下载DDNS-Go**
   - 从GitHub下载适合Mac的ddns-go版本：https://github.com/jeessy2/ddns-go/releases
   - 下载后解压并设置权限：`chmod +x ./ddns-go`

2. **运行DDNS-Go**
   - 执行`./ddns-go`命令启动程序
   - 在浏览器打开http://localhost:9876进行配置

3. **配置DDNS-Go**
   - 选择"阿里云"服务商
   - 填入AccessKey ID和AccessKey Secret
   - 在IPv6部分填入域名

4. **设置为系统服务（可选）**
   - 执行`sudo ./ddns-go -s install`设置为系统服务
   - 可以添加参数：`-l`指定监听地址，`-f`指定同步间隔（秒）

### 五、测试和验证

1. **查看日志确认运行状态**
   - 检查DDNS-Go的运行日志
   - 看到类似"你的IP XXXX 没有变化的信息表示正常运行

2. **验证外部访问**
   - 从外部网络（如手机的IPv6网络）尝试访问5900(mac上是 5900远程端口)
   - 确认可以成功连接到Mac上开启的服务

## 排错经验

1. **RAM权限问题**
   - 如果出现"Forbidden.RAM"错误，需要检查RAM用户权限
   - 确保添加了"AliyunDNSFullAccess"系统权限策略

2. **IPv6地址获取问题**
   - DDNS客户端可能通过多种方式获取IPv6地址：网络API或本地网卡
   - 可以在DDNS-Go的日志中确认使用了哪种方式获取IPv6地址

3. **域名解析生效时间**
   - DNS记录更新后可能需要一段时间才能在全球范围内生效
   - TTL值设置较小可以加快更新速度