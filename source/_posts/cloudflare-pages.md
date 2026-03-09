---
title: 从树莓派内网穿透到 Cloudflare Pages
date: 2026-03-09 13:16:57
categories:
- 运维
tags:
- Cloudflare
- Pages
---

作为一个折腾党，最初的设想是将网站跑在自家的**树莓派**上，通过 **Cloudflare Tunnel** 实现内网穿透。

虽然这种方式极具“极客感”，但在实际使用中发现，家庭宽带的上行速度和不稳定的延迟，树莓派本身的性能，对于一个博客来说还是略显吃力。

经过一番探索，将主站迁移至 **Cloudflare Pages**。这篇文章将记录这一转变背后的逻辑、操作步骤以及安全配置心得。

<!-- more -->

## 一、 为什么选择 Cloudflare Pages？

在对比了 **GitHub Pages**、**本地树莓派托管**和 **Cloudflare Pages** 后，后者的优势显而易见：

1. **全球加速（CDN）**：内容分发至全球 300+ 边缘节点，国内访问不再经过我家那根细细的网线。
2. **自动构建（CI/CD）**：只需 `git push` 到 GitHub，Cloudflare 会自动完成代码拉取、构建和发布。
3. **无限可能（Functions）**：支持 Serverless 函数，静态页面也能跑后端逻辑。
4. **安全加固**：自带 Bot 防护、脚本监控等企业级安全开关。

---

## 二、 迁移指南：5 分钟完成部署

如果你已经有了 GitHub 仓库和自定义域名（如我的 `kaifeiji.cc`），迁移过程极其顺滑：

### 1. 关联仓库

在 Cloudflare 控制台进入 **Workers & Pages**，选择 **Connect to Git**。授权后选择你的项目仓库。

### 2. 配置构建（Build Settings）

* **框架预设**：如果你是纯 HTML，选 **None**；如果是 Hugo/Hexo，选择对应框架。
* **构建命令**：使用框架默认值即可。
* **输出目录**：通常为 `/` 或 `public`。

### 3. 绑定域名

在 Pages 项目的 **Custom domains** 选项卡中添加你的主域名。由于我的 DNS 解析已在 Cloudflare，系统会自动完成 CNAME 记录的替换，实现无缝切换。

### 4. 发布上线

在 Github 进行代码提交后，Cloudflare 会自动触发构建和部署流程。几分钟后，博客就会在 Cloudflare Pages 上更新。

---

## 三、 安全与性能的“全副武装”

托管在云端并不意味着可以掉以轻心。为 `kaifeiji.cc` 开启了以下增强配置：

### 1. 阻击 AI 爬虫

在 AI 大模型时代，为了保护原创内容不被白嫖，我开启了 **Block AI training bots**。同时通过 **Manage robots.txt** 自动声明禁爬协议，既有“硬拦截”也有“软告示”。

### 2. 机器人对抗模式 (Bot Fight Mode)

开启此模式可以有效过滤掉那些无意义的扫描器和恶意爬虫，节省资源的同时让统计数据更真实。

### 3. 页面护盾 (Page Shield)

实时监控网站加载的第三方脚本。对于个人站长来说，这能有效防止因插件漏洞导致的供应链攻击。

---

## 结语

如果你也在纠结，不妨试试 Cloudflare Pages，它可能是目前个人博客最省心的方案。