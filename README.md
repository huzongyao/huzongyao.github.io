# HuZongYao的博客

这是一个基于Hexo框架搭建的个人技术博客，记录了作者在技术学习和实践过程中的心得与笔记。

## 项目结构

```
├── .github/            # GitHub Actions 部署配置
├── scaffolds/          # Hexo 模板文件
├── source/             # 博客源文件
│   ├── _data/          # 数据文件
│   ├── _posts/         # 博客文章
│   ├── about/          # 关于页面
│   ├── pwa/            # PWA 相关文件
│   └── reading/        # 阅读页面
├── themes/             # 主题目录
│   └── raytaylorism/   # 使用的主题
├── _config.yml         # Hexo 主配置文件
└── package.json        # 项目依赖配置
```

## 技术栈

- **框架**: Hexo
- **主题**: raytaylorism
- **部署**: GitHub Pages + GitHub Actions
- **PWA**: 支持渐进式Web应用

## 功能特点

- ✅ 响应式设计，适配PC和移动设备
- ✅ 支持PWA，可离线访问
- ✅ 代码高亮显示
- ✅ 分类和标签管理
- ✅ 文章归档
- ✅ 搜索功能
- ✅ 社交链接

## 博客内容

博客涵盖了多个技术领域的学习笔记和实践经验：

- **前端开发**: WebAssembly、WebP优化、前端网络安全
- **移动开发**: Android组件化、NDK开发、ROM修改和刷机
- **后端开发**: Java多线程、SpringBoot、Python Web框架
- **人工智能**: TensorFlow、OpenCV
- **嵌入式开发**: ESP8266、ESP32、PlatformIO

## 快速开始

### 环境要求

- Node.js
- npm 或 yarn

### 安装依赖

```bash
npm install
```

### 本地运行

```bash
# 清理缓存
hexo clean

# 生成静态文件
hexo generate

# 启动本地服务器
hexo server
```

访问 http://localhost:4000 查看博客

### 部署

项目使用GitHub Actions自动部署，推送代码到主分支后会自动构建并部署到GitHub Pages。

## 主题配置

博客使用了raytaylorism主题，主要配置包括：

- 颜色方案：基于Material Design 3
- 字体：Roboto
- 布局：响应式设计
- 功能：支持代码高亮、社交分享、文章统计等

## 文章管理

### 创建新文章

```bash
hexo new "文章标题"
```

### 文章格式

文章使用Markdown格式，包含以下Front Matter：

```yaml
title: 文章标题
date: 2024-01-01 00:00:00
tags:
- 标签1
- 标签2
categories:
- 分类1
- 分类2
```

## 贡献

欢迎提交Issue和Pull Request来改进这个博客项目。

## 许可证

MIT License

## 联系方式

- GitHub: https://github.com/huzongyao
- 博客地址: https://huzongyao.github.io

---

**最后更新**: 2026-03-26