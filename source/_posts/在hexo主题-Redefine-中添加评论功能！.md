---
title: 在hexo主题<Redefine>中添加评论功能！
date: 2025-09-02 10:52:18
tags: [Hexo, blog, 评论功能, 主题, Redefine]
categories: [博客开发]
---

## 概述

偶然发现我hexo的主题是支持评论功能的，于是我给他加上这个功能。能够促进作者与读者之间的互动交流。我决定使用Giscus评论系统，它是基于GitHub Discussions的评论系统，适合我这种轻量小博客使用。

## Giscus评论系统的优势

**完全免费**：它基于GitHub的免费服务，无需付费订阅或购买服务器！
**主题兼容**：支持Redefine明暗主题自动切换
**功能完整**：支持回复、点赞、表情反应等交互功能

## 配置前准备

### 1. GitHub的仓库要求

要确保博客源码仓库满足以下条件：
- 仓库必须是公开的（Public）
- 已启用GitHub Discussions功能

启用Discussions的步骤：
1. 进入GitHub仓库主页
2. 点击Settings标签
3. 在Features部分找到Discussions选项
4. 勾选启用Discussions

### 2. 安装Giscus GitHub App

访问 https://github.com/apps/giscus 安装Giscus应用到你的仓库。这一步骤是必需的，因为Giscus需要相应权限来读取和创建Discussions。

## 获取Giscus配置参数

访问Giscus配置页面（https://giscus.app/zh-CN），按照以下步骤获取配置参数：

### 1. 基本配置
- **仓库**：输入你的GitHub仓库地址，格式为 `用户名/仓库名`
- **页面与讨论映射**：推荐选择"Discussion的标题包含页面的pathname"
- **Discussion分类**：建议选择"Announcements"或创建专门的评论分类

### 2. 功能配置
- **启用回复功能**：允许用户回复评论
- **启用emoji反应**：允许用户对评论进行表情反应
- **主题选择**：推荐使用"preferred_color_scheme"以自动适配系统主题

配置完成后，页面会自动生成所需的参数，包括：
- `data-repo`
- `data-repo-id`
- `data-category`
- `data-category-id`

## Redefine主题配置

编辑你的Redefine主题配置文件`_config.redefine.yml`，在评论部分添加以下配置：

```yaml
# 评论系统配置
comment:
  enable: true
  system: giscus
  config:
    giscus:
      repo: your-username/your-repository-name
      repo_id: R_kgDOxxxxxxxx
      category: Announcements
      category_id: DIC_kwDOxxxxxxxx
      mapping: pathname
      strict: 0
      reactions_enabled: 1
      emit_metadata: 0
      input_position: bottom
      theme: preferred_color_scheme
      lang: zh-CN
      loading: lazy
```

### 配置参数说明

**repo**: GitHub仓库地址，格式为用户名/仓库名
**repo_id**: 仓库的唯一标识符，从Giscus配置页面获取
**category**: Discussions分类名称
**category_id**: 分类的唯一标识符
**mapping**: 页面与Discussion的映射方式
  - `pathname`: 基于页面路径（推荐）
  - `url`: 基于完整URL
  - `title`: 基于页面标题
**strict**: 是否启用严格标题匹配（0=关闭，1=开启）
**reactions_enabled**: 是否启用emoji反应功能
**input_position**: 评论输入框位置（top/bottom）
**theme**: 主题样式
  - `light`: 浅色主题
  - `dark`: 深色主题
  - `preferred_color_scheme`: 跟随系统主题
**lang**: 界面语言
**loading**: 加载方式（lazy/eager）

## 部署与测试

完成配置后，执行以下命令重新生成并部署博客：

```bash
hexo clean
hexo generate
hexo deploy
```

部署完成后，访问任意博客文章页面，应该能看到Giscus评论组件。使用GitHub账号登录即可测试评论功能。

## 参考文档

- [Giscus官方文档](https://giscus.app/)
- [GitHub Discussions文档](https://docs.github.com/en/discussions)
- [Redefine主题文档](https://redefine-docs.ohevan.com/)
