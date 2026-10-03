---
title: 微信小程序接入阿里云 OSS 对象存储：Bucket、权限、CORS 和域名白名单
date: 2026-10-03 21:00:00
tags: [微信小程序, 阿里云, OSS, 对象存储]
categories: [服务器运维]
---

做小程序（跑腿类）上线准备时，头像、订单图片、富文本图片都放在阿里云 OSS 里。这篇把整个接入流程整理一遍，包括创建 Bucket 时一个容易踩的权限坑。

## 整体结构

- **Bucket**：存放图片的桶，配置里对应 `ossRegion`（地域）和 `ossBucket`（桶名）。
- **小程序上传**：走自己的服务器，再由服务器传到 OSS。
- **管理后台上传**：浏览器直传 OSS，需要后端用 STS 换取临时密钥。
- 这些配置我们放在数据库里，由后台的「上传设置」页修改，没有写进代码。

## 一、创建 Bucket

在 [OSS 控制台](https://oss.console.aliyun.com/) 创建一个 Bucket，地域选离用户近的（我们用的是广州 `oss-cn-guangzhou`）。

小程序里的 `<image>` 标签直接用图片的裸链接，没有做签名 URL，所以这个 Bucket 必须是**公共读**。阿里云有两层限制，两层都要改对：

1. **创建时**：读写权限只能选「私有」，这是阿里云强制的，正常现象，直接创建。
2. **创建后**，进入 Bucket →「权限管理」：
   1. 先到「阻止公共访问」，把开关**关闭**；
   2. 再到「读写权限」，改成**公共读**。

只改第二步没用，「阻止公共访问」默认开着，会照样拦截访问，图片会返回 403。

**一个项目一个 Bucket。** 不同项目的图片不要混在同一个 Bucket 里：以后单独统计流量、控制权限、清理数据都会更方便，也避免给一个项目授权时，顺带放大到别的项目的文件。新建时先看清桶名，别误点进别的项目的 Bucket。

证书这类敏感文件也不要放进公共读的图片桶，需要的话另建一个**私有**桶。

## 二、RAM 子账号和 STS 角色（管理后台直传用）

浏览器直传 OSS 不能把长期密钥发给前端，要后端用 STS 换一个临时凭证。在 [RAM 控制台](https://ram.console.aliyun.com/) 里：

1. 创建一个 RAM 子账号，生成 AccessKey，填进后台的 `accessKeyId` / `accessKeySecret`（不要用主账号的 AccessKey）；
2. 创建一个 RAM 角色，可信实体选**当前阿里云账号**；
3. 给角色添加权限 `AliyunOSSFullAccess`，给子账号添加权限 `AliyunSTSAssumeRoleAccess`；
4. 把角色的 **ARN** 填进后台的 ARN 输入框。

我们踩过这个坑：ARN 一直是空的，后台上传就一直失败，排查了一阵才发现。

`AliyunOSSFullAccess` 是对所有 Bucket 的完全权限，范围比较大。想收紧的话，可以自定义一个只允许操作指定 Bucket 的权限策略，再挂给角色。

## 三、跨域设置（CORS）

浏览器直传是跨域请求，要在 OSS 里放行。进入 Bucket →「权限管理」→「跨域设置（CORS）」，添加一条规则：

| 项目 | 填写 |
| --- | --- |
| 来源 | 管理后台的域名，比如 `https://admin.example.com` |
| 允许 Methods | GET、POST、PUT、DELETE、HEAD |
| 允许 Headers | `*` |
| 暴露 Headers | `ETag` |

## 四、小程序的域名白名单

在[微信公众平台](https://mp.weixin.qq.com/)的「开发 → 开发管理 → 开发设置 → 服务器域名」里：

- `request` 合法域名：填自己的 API 域名，必须是 **https**，而且要已经 ICP 备案；
- `uploadFile` 合法域名：填 OSS 的域名；
- `downloadFile` 合法域名：填 OSS 的域名。

没配的话，小程序里上传、下载图片都会被拦截。

## 五、配置落地

把下面几项填进后台的「地图及上传设置」：

- `ossRegion`：Bucket 所在地域，比如 `oss-cn-guangzhou`；
- `ossBucket`：桶名；
- `accessKeyId` / `accessKeySecret`：RAM 子账号的 AccessKey；
- `arn`：上面角色的 ARN。

## 排查清单

- 图片返回 403：先看「阻止公共访问」是不是还开着，再看读写权限是不是公共读；
- 后台上传失败：看 ARN 是否填了，子账号是否挂了 `AliyunSTSAssumeRoleAccess`；
- 浏览器控制台报跨域：检查 CORS 的来源是否和后台域名完全一致，包括 `https://`；
- 小程序里图片不显示或上传失败：检查域名白名单是不是漏了 OSS 域名。

AccessKey 只放在后台配置或服务器环境里，不要写进代码、文档或聊天记录；一旦泄露，马上到 RAM 里轮换。
