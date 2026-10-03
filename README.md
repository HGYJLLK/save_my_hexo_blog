# save_my_hexo_blog

[hgyjllk代码手记](https://blog.hgyjllk.top) 的 Hexo 源码仓库（主题：[Redefine](https://github.com/EvanNotFound/hexo-theme-redefine)）。

## 目录结构

| 路径 | 说明 |
| --- | --- |
| `source/_posts/` | 博客文章（Markdown） |
| `source/images/` | 站点图片 |
| `_config.yml` | Hexo 站点配置 |
| `_config.redefine.yml` | Redefine 主题配置（站点标题、导航栏、评论等） |
| `scaffolds/` | 新文章模板 |
| `public/` | 生成后的静态文件，**会提交进仓库**，上线用的就是它 |

## 写一篇新文章

```bash
npm install                # 第一次需要
npx hexo new "文章标题"     # 生成 source/_posts/文章标题.md
```

文章开头的 front-matter：

```markdown
---
title: 文章标题
date: 2026-10-02 14:00:00
tags: [标签1, 标签2]
categories: [分类]
---
```

## 本地预览

```bash
npx hexo server            # http://localhost:4000
```

改文章会自动刷新；改了 `_config.yml` 或 `_config.redefine.yml` 要**重启**预览服务。

## 生成并上线

```bash
npx hexo clean && npx hexo generate
git add -A
git commit -m "docs: Add new blog post"
git push
```

上线前检查 `public/index.html` 不是 0 字节。

## 注意事项

- 导航栏里 `Home`、`Archives` 两个键名保持英文，主题会自动显示成「首页」「归档」；写成中文会出现重复的「首页」。其余导航项（如「代码仓库」「更多网站」）直接用中文。
- 站点标题使用中文「hgyjllk代码手记」，和备案名称保持一致。
- `themes/redefine` 如果是个空目录，会盖住 npm 安装的主题，导致生成的页面全是空白。遇到时删掉这个空目录再生成。
- `_config.yml` 里的 `deploy.type` 为空，`hexo deploy` 不会生效，上线靠提交 `public/`。
- 线上页面里有一段统计脚本（`user.alcmaple.cn`），不在本仓库的配置里，是服务器侧加的。覆盖部署时留意不要丢。
