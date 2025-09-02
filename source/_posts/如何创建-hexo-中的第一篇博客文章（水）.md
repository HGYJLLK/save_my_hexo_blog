---
title: 如何创建 hexo 中的第一篇博客文章（水）
date: 2025-04-18 17:37:15
tags: [ hexo, Apache , blog]
categories: [博客开发]
---
# 立马部署！

> 总结了使用Hexo创建博客、发布第一篇文章以及配置Redefine主题的完整流程。

## 一、初始化Hexo博客

### 1. 安装Node.js和Hexo命令行工具
```bash
# 安装Node.js
brew install node
node -v

# 安装Hexo命令行工具
npm install -g hexo-cli

# 初始化博客项目
hexo init my-blog
cd my-blog

# 安装依赖
npm install
```

### 2. 你将会得到一个类似如下的目录结构：
```
my-blog/
├── _config.yml        # 站点配置文件
├── package.json       # 应用程序信息
├── scaffolds/         # 模板文件夹
├── source/            # 存放用户资源的地方
│   ├── _drafts/       # 草稿箱
│   └── _posts/        # 文章
├── themes/            # 主题文件夹
└── node_modules/      # 依赖项
```

## 二、发布第一篇文章进行测试（不试试怎么知道）

### 1. 创建新文章
```bash
hexo new "我的第一篇博客文章"
```

### 2. 编辑文章
编辑 `source/_posts/我的第一篇博客文章.md` 文件：
```bash
vi source/_posts/我的第一篇博客文章.md
```

```markdown
---
title: 我的第一篇博客文章
date: 2024-04-18 14:30:00
tags: [博客, Hexo]
categories: 技术
---

## 大家好！

这是我用Hexo创建的第一篇博客文章。

```
然后 按ESC & :wq 保存并退出。

### 3. 生成静态文件并预览
```bash
# 生成静态文件
hexo generate
# 或简写
hexo g

# 启动本地服务器预览
hexo server
# 或简写
hexo s
```

访问 `http://localhost:4000` 预览博客。

## 三、安装一个好看的主题（be like redefine）！

### 1. 下载主题
打开这个网址：[Hexo主题库](https://hexo.io/themes/)
寻找一个你喜欢的主题！我的部署如下：
```bash
# 把喜欢的主题Git克隆主题到themes目录吧！
git clone https://github.com/EvanNotFound/hexo-theme-redefine.git themes/redefine
```

### 2. 配置站点使用Redefine主题
修改Hexo根目录下的 `_config.yml`：
```bash
vi _config.yml
```
command+f 搜索 `theme` 找到 `theme: ` 这一行，将 `theme: default` 修改为 `theme: redefine`。

```yaml
theme: redefine
```

### 3. 创建主题配置文件
在Hexo根目录创建 `_config.redefine.yml` 文件，复制主题默认配置并进行自定义：
```bash
touch _config.redefine.yml
vi _config.redefine.yml
```
> 以下是我的自行汉化～建议英语好的还是用原作者的吧 XP
```json
# 主题配置文件 - Redefine 主题 v2
# 作者: EvanNotFound
# GitHub: https://github.com/EvanNotFound/hexo-theme-redefine

# ========== 基本信息 ==========
info:
  # 网站标题
  title: 我的博客
  # 网站副标题
  subtitle: 个人博客副标题
  # 作者名称
  author: 你的名字
  # 网站URL
  url: https://blog.hgyjllk.top

# ========== 图片设置 ==========
defaults:
  # 网站图标
  favicon: /images/favicon.png
  # 网站logo（可留空）
  logo: 
  # 头像
  avatar: /images/avatar.jpg

# ========== 颜色设置 ==========
colors:
  # 主题色
  primary: "#005080"
  # 次要颜色（预留）
  secondary:
  # 默认主题模式
  default_mode: light  # light亮色模式, dark暗色模式

# ========== 全局设置 ==========
global:
  # 字体设置
  fonts:
    # 中文字体
    chinese: 
      enable: false  # 是否启用自定义中文字体
      family:        # 字体名称
      url:           # 字体CSS链接
    # 英文字体
    english: 
      enable: false  # 是否启用自定义英文字体
      family:        # 字体名称
      url:           # 字体CSS链接
    # 标题字体（导航栏、侧边栏）
    title:
      enable: false  # 是否启用自定义标题字体
      family:        # 字体名称
      url:           # 字体CSS链接
  # 内容最大宽度
  content_max_width: 1000px
  # 侧边栏宽度
  sidebar_width: 210px
  # 鼠标悬停效果
  hover:
    shadow: true     # 阴影效果
    scale: false     # 缩放效果
  # 滚动进度
  scroll_progress:
    bar: false       # 进度条
    percentage: true # 百分比
  # 网站计数器
  website_counter:
    url: https://cn.vercount.one/js  # 计数器API
    enable: true     # 是否启用网站计数器
    site_pv: true    # 网站浏览量
    site_uv: true    # 网站访客数
    post_pv: true    # 文章浏览量
  # 是否启用单页面体验（类似PJAX）
  single_page: true
  # 是否启用加载动画
  preloader:
    enable: false
    custom_message:  # 自定义消息，留空则显示网站标题
  # 是否启用Open Graph
  open_graph:
    enable: true
    image: /images/og-image.webp   # 默认分享图片
    description: 我的个人博客      # 描述
  # Google分析
  google_analytics:
    enable: false    # 是否启用Google分析
    id:              # Google分析ID

# ========== FontAwesome图标 ==========
fontawesome:
  # 细线图标
  thin: false
  # 轻量图标
  light: false
  # 双色图标
  duotone: false
  # 锐利实心图标
  sharp_solid: false

# ========== 首页顶部大图 ==========
home_banner:
  # 是否启用首页顶部大图
  enable: true
  # 大图样式
  style: fixed       # static静态或fixed固定
  # 大图背景图片
  image: 
    light: /images/banner-light.webp  # 亮色模式图片
    dark: /images/banner-dark.webp    # 暗色模式图片
  # 大图标题
  title: 我的博客
  # 大图副标题
  subtitle:
    text: ["副标题文本"]  # 副标题文本数组
    hitokoto:        # 一言配置
      enable: false  # 是否启用一言
      show_author: false  # 是否显示作者
      api: https://v1.hitokoto.cn  # API地址
    typing_speed: 100    # 打字速度(毫秒)
    backing_speed: 80    # 退格速度(毫秒)
    starting_delay: 500  # 开始延迟(毫秒)
    backing_delay: 1500  # 退格延迟(毫秒)
    loop: true       # 是否循环
    smart_backspace: true  # 智能退格
  # 文字颜色
  text_color: 
    light: "#fff"    # 亮色模式文字颜色
    dark: "#d1d1b6"  # 暗色模式文字颜色
  # 文字样式
  text_style: 
    title_size: 2.8rem      # 标题字体大小
    subtitle_size: 1.5rem   # 副标题字体大小
    line_height: 1.2        # 行高
  # 自定义字体
  custom_font: 
    enable: false   # 是否启用自定义字体
    family:         # 字体名称
    url:            # 字体链接
  # 社交链接
  social_links:
    enable: false   # 是否启用
    style: default  # 样式: default默认, reverse反转, center居中
    # 社交链接
    links:
      github:       # GitHub链接 
      instagram:    # Instagram链接
      zhihu:        # 知乎链接
      twitter:      # Twitter链接
      email:        # 邮箱地址
    # 带二维码的社交链接
    qrs:
      weixin:       # 微信二维码图片链接

# ========== 导航栏 ==========
navbar:
  # 自动隐藏导航栏
  auto_hide: false
  # 导航栏背景颜色
  color:
    left: "#f78736"   # 左侧颜色
    right: "#367df7"  # 右侧颜色
    transparency: 35  # 透明度(10-99)
  # 导航栏宽度
  width:
    home: 1200px      # 首页宽度
    pages: 1000px     # 其他页面宽度
  # 导航链接
  links:
    首页: 
      path: / 
      icon: fa-regular fa-house  # 图标(可为空)
    归档: 
      path: /archives 
      icon: fa-regular fa-archive
    分类: 
      path: /categories 
      icon: fa-regular fa-folder
    标签: 
      path: /tags 
      icon: fa-regular fa-tags
    关于: 
      path: /about
      icon: fa-regular fa-user
  # 搜索功能(需要安装hexo-generator-searchdb插件)
  search:
    enable: false     # 是否启用
    preload: true     # 预加载搜索数据

# ========== 首页文章设置 ==========
home:
  # 侧边栏设置
  sidebar:
    enable: true      # 是否启用侧边栏
    position: left    # 侧边栏位置: left左侧, right右侧
    first_item: menu  # 侧边栏第一项: menu菜单, info信息
    announcement:     # 公告文本
    show_on_mobile: true  # 在移动端显示侧边栏导航
    links:
      归档: 
        path: /archives 
        icon: fa-regular fa-archive
      分类: 
        path: /categories 
        icon: fa-regular fa-folder
      标签: 
        path: /tags 
        icon: fa-regular fa-tags
  # 文章日期格式
  article_date_format: auto  # auto自动, relative相对时间, YYYY-MM-DD等
  # 文章摘要长度
  excerpt_length: 200  # 最大摘要长度
  # 文章分类可见性
  categories:
    enable: true   # 是否启用
    limit: 3       # 最多显示分类数量
  # 文章标签可见性
  tags:
    enable: true   # 是否启用
    limit: 3       # 最多显示标签数量

# ========== 文章页设置 ==========
articles:
  # 文章样式
  style:
    font_size: 16px           # 字体大小
    line_height: 1.5          # 行高
    image_border_radius: 14px # 图片圆角
    image_alignment: center   # 图片对齐方式: left左对齐, center居中
    image_caption: false      # 是否显示图片说明
    link_icon: true           # 是否显示链接图标
    delete_mask: false        # 为删除线添加遮罩效果
    title_alignment: left     # 标题对齐: left左对齐, center居中
  # 字数统计(需要安装hexo-wordcount插件)
  word_count:
    enable: true  # 是否启用
    count: true   # 是否显示字数统计
    min2read: true # 是否显示阅读时间
  # 作者标签
  author_label: 
    enable: true  # 是否启用
    auto: false   # 是否自动添加等级标签
    list: []      # 自定义标签列表
  # 代码块设置
  code_block:
    copy: true    # 是否启用复制按钮
    style: mac    # 样式: mac或simple
    highlight_theme:  # 高亮主题
      light: github   # 亮色模式主题
      dark: vs2015    # 暗色模式主题
  # 目录设置
  toc:
    enable: true      # 是否启用目录
    max_depth: 3      # 目录深度
    number: false     # 是否自动添加编号
    expand: true      # 是否展开目录
    init_open: true   # 默认打开目录
  # 版权声明
  copyright:
    enable: true      # 是否启用
    default: cc_by_nc_sa  # 默认许可证
  # 是否启用图片懒加载
  lazyload: true
  # 自动在中英文之间添加空格
  pangu_js: false

# ========== 评论系统 ==========
comment:
  # 是否启用评论
  enable: true
  # 评论系统
  system: waline  # waline, gitalk, twikoo, giscus
  # 系统配置
  config:
    # Waline评论系统
    waline:
      serverUrl: https://example.example.com  # 服务器地址
      lang: zh-CN  # 语言
      # 其他配置请参考waline文档

# ========== 页脚 ==========
footer:
  # 显示网站运行时间
  runtime: true
  # 页脚图标
  icon: '<i class="fa-solid fa-heart fa-beat" style="--fa-animation-duration: 0.5s; color: #f54545"></i>'
  # 网站开始时间
  start: 2024/4/18 00:00:00
  # 站点统计
  statistics: true  # 显示站点统计（文章总数、字数统计）
  # ICP备案号
  icp:
    enable: false   # 是否启用
    number:         # ICP备案号
    url:            # ICP备案链接

# ========== 注入自定义代码 ==========
inject:
  # 是否启用注入
  enable: false
  # 注入自定义HTML头部代码
  head: 
    -
    -
  # 注入自定义HTML底部代码
  footer:
    -
    -

# ========== 插件 ==========
plugins:
  # RSS订阅(需要安装hexo-generator-feed插件)
  feed:
    enable: false
  # 音乐播放器
  aplayer:
    enable: false
    type: fixed  # fixed固定或mini迷你
    audios:
      - name:    # 音频名称
        artist:  # 艺术家
        url:     # 音频链接
        cover:   # 封面图片
  # Mermaid图表(需要安装hexo-filter-mermaid-diagrams插件)
  mermaid:
    enable: false
    version: "11.4.1"

# ========== 页面模板 ==========
page_templates:
  # 友情链接页列数
  friends_column: 2
  # 标签页样式
  tags_style: blur  # blur模糊或cloud云标签

# ========== CDN设置 ==========
cdn:
  # 是否启用CDN
  enable: false
  # CDN提供商
  provider: npmmirror  # npmmirror, zstatic, cdnjs, jsdelivr, unpkg, custom

# ========== 开发者模式 ==========
developer:
  # 是否启用开发者模式(仅供主题开发者使用)
  enable: false
```
 然后自己把需要个性化的地方都更改一下！查看一下效果！
 ```bash
 hexo clean && hexo g && hexo s
 ```

## 四、生成并部署博客

### 1. 使用SCP部署
```bash
# 生成静态文件
hexo clean && hexo generate

# 使用SCP上传到服务器
scp -r public/* root@你的公网 IP:/var/www/blog.hgyjllk.top/public_html/
```
然后完成部署啦！