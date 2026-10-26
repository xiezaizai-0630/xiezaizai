<div align="center">

# boy

> Xiezaizai 的个人博客 · 事缓则圆

[![Astro](https://img.shields.io/badge/Astro-7-BC52EE?logo=astro&logoColor=white)](https://astro.build)
[![TypeScript](https://img.shields.io/badge/TypeScript-5-3178C6?logo=typescript&logoColor=white)](https://www.typescriptlang.org)
[![License](https://img.shields.io/badge/license-MIT-green)](./LICENSE)

[**🌐 在线访问**](https://xiezaiz.ccwu.cc/) · [**📝 关于我**](https://xiezaiz.ccwu.cc/about/)

</div>

---

## 简介

这是 [Xiezaizai](https://github.com/xiezaiz-0630) 的个人博客源码仓库，用来记录生活、学习和折腾的痕迹。

站点基于 Astro 构建，静态生成，部署在 Cloudflare Pages。

- **博客地址**：<https://xiezaiz.ccwu.cc/>
- **仓库地址**：<https://github.com/xiezaiz-0630/xiezaizai>

## 主要功能

- 文章归档、分类、标签、系列
- 基于 Pagefind 的全文搜索
- 动态、项目、相册、书签导航
- 友链与留言板，基于 giscus 的评论
- RSS / Atom 订阅
- 亮色 / 暗色 / 跟随系统三种主题模式
- 全屏壁纸与响应式布局

## 本地运行

环境要求：Node.js ≥ 22，pnpm ≥ 11。

```bash
pnpm install     # 安装依赖
pnpm dev         # 本地开发，默认 http://localhost:4321
pnpm build       # 构建到 ./dist/
pnpm preview     # 预览构建结果
```

其他常用命令：

| 命令 | 说明 |
| :--- | :--- |
| `pnpm check` | 检查 Astro 代码 |
| `pnpm type-check` | TypeScript 类型检查 |
| `pnpm format` | Biome 格式化 |
| `pnpm new-post <filename>` | 新建文章 |
| `pnpm new-d <content>` | 新建动态 |

## 内容与配置

```
src/
├── config/          # 站点配置（站点信息、导航、评论、打赏等）
├── content/
│   ├── posts/       # 文章
│   ├── dynamic/     # 动态
│   ├── projects/    # 项目
│   └── spec/        # 关于、友链、留言页内容
├── pages/           # 路由页面
├── components/      # 组件
├── layouts/         # 布局
└── styles/          # 样式
```

文章写在 `src/content/posts/` 下，frontmatter 示例：

```yaml
---
title: 文章标题
published: 2026-10-25
description: 文章简介
tags: [随笔]
category: 生活
draft: false
---
```

## 致谢与许可

本项目基于 [fuwari](https://github.com/saicaca/fuwari)（作者 [saicaca](https://github.com/saicaca)）与 [Firefly](https://github.com/CuteLeaf/Firefly)（作者 [CuteLeaf](https://github.com/CuteLeaf)）二次修改，遵循 [MIT License](./LICENSE)。

- Copyright (c) 2024 [saicaca](https://github.com/saicaca) - fuwari
- Copyright (c) 2025 [CuteLeaf](https://github.com/CuteLeaf) - Firefly

根据 MIT 协议保留上述版权声明，其余内容与配置为本站自有。
