# AI 工具导航与资讯网站需求文档

## 1. 项目概述

### 1.1 项目名称

- 暂定项目名称：**AInew**

### 1.2 项目目标

- 构建一个高质量的 AI 工具和资讯聚合平台，通过搜索引擎优化（SEO）获取可持续的自然流量。
- 提供简洁且有用的信息，帮助用户发现最新的 AI 工具、了解行业动态、探索热门大模型及 MCP（Managed Cloud Platforms）服务导航。

### 1.3 目标用户

- 对人工智能技术感兴趣的普通用户。
- 寻找效率工具的产品经理、设计师、开发者、内容创作者等专业人士。
- 希望了解 AI 行业动态的学生和研究者。

### 1.4 核心价值主张

- **全面聚合**：一站式发现数千款 AI 工具，涵盖写作、设计、视频、编程、营销等全场景。
- **信息及时**：通过自动化系统，每日更新最新发布的 AI 工具和行业资讯。
- **搜索友好**：网站结构和内容高度优化，确保在 Google、Bing 等搜索引擎中获得良好排名。
- **零成本访问**：完全免费使用，无广告干扰（初期）。

## 2. 功能需求

### 2.1 核心功能模块

| 模块 | 功能描述 |
| --- | --- |
| AI 工具导航 | - 展示所有收录的 AI 工具。<br>- 每个工具拥有独立详情页，包含：名称、Logo、简介、官网链接、价格（免费/付费）、支持平台、适用场景、标签、用户评分（可选）。<br>- 支持按 **分类 (Category)**、**标签 (Tag)**、**价格** 进行筛选。<br>- 支持关键词搜索。 |
| AI 资讯博客 | - 发布关于 AI 领域的新闻、趋势分析、工具评测文章。<br>- 每篇文章有独立页面，包含标题、摘要、作者、发布时间、正文、相关标签。<br>- 支持文章列表按时间倒序排列。 |
| 热门大模型 | - 列出当前最流行或最具影响力的大规模机器学习模型。<br>- 包括模型介绍、应用场景、性能指标等。 |
| MCP 服务导航 | - 提供各大云服务商的 AI 相关服务概览。<br>- 包含服务名称、提供商、简介、定价、使用案例等。 |
| 首页展示 | - 轮播图或精选区：展示热门或新上架的 AI 工具、大模型、MCP 服务。<br>- 最新工具列表：显示最近添加的 AI 工具。<br>- 最新资讯列表：显示最近发布的文章。<br>- 分类导航快捷入口。 |

### 2.2 内容管理需求

#### 预先抓取内容填充

- **RSS Feeds 和 API 数据源**：
  - TechCrunch AI: <https://techcrunch.com/tag/artificial-intelligence/feed/>
  - VentureBeat AI: <https://venturebeat.com/category/ai/feed/>
  - Product Hunt API: 获取 AI 类别新品。
  - Google News API: 根据关键词检索 AI 新闻。
  - ArXiv API: 获取最新的 AI 研究论文。

- **脚本实现**：
  - 编写 Python 脚本，使用 `feedparser` 库解析 RSS Feed，使用 `requests` 库调用 API 接口。
  - 解析数据后，生成符合 Hugo 格式的 Markdown 文件，存入对应目录（`content/ai-tools/`、`content/ai-news/`、`content/models/`、`content/mcp-services/`）。

#### 自动更新内容

- **定时任务**：
  - 使用 GitHub Actions 定期运行上述脚本，确保内容的实时性。

- **示例 GitHub Actions 工作流 `update.yml`**：

  ```yaml
  name: Update Content

  on:
    schedule:
      - cron: '0 0 * * *' # 每天凌晨执行
    workflow_dispatch:

  jobs:
    update:
      runs-on: ubuntu-latest
      steps:
        - uses: actions/checkout@v3
          with:
            token: ${{ secrets.GITHUB_TOKEN }}
        - name: Set up Python
          uses: actions/setup-python@v4
          with:
            python-version: '3.8'
        - name: Install dependencies
          run: |
            pip install feedparser requests
        - name: Run update script
          run: python scripts/update_content.py
        - name: Build Hugo site
          run: hugo --minify
        - name: Deploy to GitHub Pages
          uses: peaceiris/actions-hugo@v2
          with:
            hugo-version: 'latest'
            publish-folder: 'public'
            external-github-token: ${{ secrets.GITHUB_TOKEN }}
            commit-message: 'Automated update: new tools and posts'
  ```

### 2.3 SEO 专项需求

| 需求 | 技术实现方案 |
| --- | --- |
| 静态页面生成 | 使用 Hugo SSG 生成纯 HTML 页面，确保爬虫可高效抓取所有内容。 |
| 语义化 URL | 生成对 SEO 友好的 URL，例如：`/ai-tools/chatgpt`、`/ai-news/openai-launches-new-model`、`/models/gpt-3`、`/mcp-services/aws-sagemaker`。 |
| Meta 标签优化 | 为每一页自动生成唯一的 `<title>` 和 `<meta name="description">`。 |
| 结构化数据 (Schema) | 在页面中嵌入 JSON-LD 格式的 Schema Markup：<br>- 工具页：`SoftwareApplication`<br>- 文章页：`Article`<br>- 大模型页：`Dataset` 或 `TechArticle`<br>- MCP 服务页：`Service`。 |
| 站点地图 (Sitemap) | 自动生成 `sitemap.xml` 并提交至 Google Search Console。 |
| Robots 协议 | 生成 `robots.txt` 文件，指导爬虫行为。 |
| 性能优化 | 图片懒加载、压缩资源、利用 CDN（GitHub Pages）实现快速加载。 |

## 3. 技术架构

### 3.1 技术栈

| 层级 | 技术选择 | 理由 |
| --- | --- | --- |
| 前端框架 | [Hugo](https://gohugo.io/) (静态网站生成器) | 极致性能、简单易学、强大的模板系统、天生 SEO 友好。 |
| 托管平台 | [GitHub Pages](https://pages.github.com/) | 免费、稳定、全球 CDN 加速、与 GitHub 生态无缝集成。 |
| 内容存储 | Markdown 文件 + Git 仓库 | 内容即代码，版本控制清晰，易于管理和协作。 |
| 自动化引擎 | [GitHub Actions](https://github.com/features/actions) | 免费定时任务，可运行脚本实现内容抓取、网站构建和部署。 |
| 内容抓取 | Python 脚本（使用 feedparser、requests、BeautifulSoup 等库） | 灵活强大，易于处理各种数据源。 |
| 图片存储 | 专业免费图床（如 路过图床、SM.MS、z.run） | 保持 GitHub 仓库轻量，利用专业 CDN 提升图片加载速度。 |
| 主题 | 从 [Hugo Themes](https://themes.gohugo.io/) 中选择或定制 | 快速搭建美观界面，重点关注 SEO 和响应式设计。 |

### 3.2 系统架构图

```
+----------------+        +---------------------+
| 外部数据源      |        |                     |
| (RSS, API, etc.)|        |                     |
+-------+--------+        |                     |
        |                 |    抓取             |
        |                 v                     |
        |         +---------------+             |
        |         |               |             |
        +-------->| GitHub 仓库    |-------------+
                  |               |
                  | - Hugo 项目   |
                  |   - content/  |
                  |   - scripts/  |     构建
                  |   - config.toml
                  +-------+-------+
                          |
                          v
                 +--------+--------+
                 | 自动化脚本       |
                 | (Python)        |
                 +--------+--------+
                          |
                          v
                 +--------+--------+
                 | GitHub Actions  |
                 | (定时触发 & 部署)|
                 +--------+--------+
                          |
                          v
                 +--------+--------+
                 | GitHub Pages    |
                 | (CDN)           |
                 +----------------+
```

## 4. 非功能性需求

| 需求 | 描述 |
| --- | --- |
| 性能 | 页面加载速度应极快（Lighthouse 性能评分 > 90），所有静态资源通过 CDN 分发。 |
| 可用性 | 网站 7×24 小时可用，GitHub Pages SLA 高。 |
| 可维护性 | 代码结构清晰，文档齐全，自动化程度高，降低长期维护成本。 |
| 可扩展性 | 架构支持未来添加新功能，如用户评论、收藏夹、API 接口等。 |
| 安全性 | 静态网站本身攻击面小。依赖的第三方库需定期更新。 |

## 5. 开发与部署流程

### 5.1 预先抓取内容填充

1. **编写抓取脚本**：
   - 创建 Python 脚本 `scripts/fetch_initial_content.py`，用于从选定的数据源抓取初始内容并生成 Markdown 文件。

   ```python
   import feedparser
   import os
   from datetime import datetime
   import requests

   def fetch_rss_feed(url):
       return feedparser.parse(url)

   def fetch_api_data(url, headers=None):
       response = requests.get(url, headers=headers)
       response.raise_for_status()
       return response.json()

   def upload_image_to_imgchr(image_url):
       # 下载远程图片
       image_response = requests.get(image_url, stream=True)
       if image_response.status_code != 200:
           return None

       # 准备上传
       files = {'image': image_response.raw}
       data = {
           'format': 'json',
           'type': 'file'
       }
       upload_response = requests.post('https://imgchr.com/json', files=files, data=data)
       if upload_response.status_code == 200:
           result = upload_response.json()
           if result['code'] == 200:
               return result['data']['url']
       return None

   def generate_markdown(feed_entry, category):
       title = feed_entry.title
       link = feed_entry.link
       published = feed_entry.published if hasattr(feed_entry, 'published') else datetime.now().isoformat()
       summary = feed_entry.summary if hasattr(feed_entry, 'summary') else ''
       image_url = feed_entry.get('media_content', [{'url': ''}])[0]['url'] or feed_entry.get('thumbnail', {}).get('url', '')

       # 上传图片到路过图床
       hosted_image_url = upload_image_to_imgchr(image_url) if image_url else ''

       filename = f"content/{category}/{title.replace(' ', '-').lower()}.md"

       with open(filename, 'w', encoding='utf-8') as file:
           file.write(f"""---
   title: \"{title}\"
   link: \"{link}\"
   date: \"{published}\"
   featured_image: \"{hosted_image_url}\"

   {summary}
   """)

   rss_feeds = [
       'https://techcrunch.com/tag/artificial-intelligence/feed/',
       'https://venturebeat.com/category/ai/feed/'
   ]

   for rss_feed in rss_feeds:
       feed = fetch_rss_feed(rss_feed)
       for entry in feed.entries:
           generate_markdown(entry, 'ai-news')
   ```

2. **运行脚本**：执行 `python scripts/fetch_initial_content.py`，将生成的 Markdown 文件添加到 Git 仓库中。

### 5.2 初始化仓库

1. 创建 GitHub 仓库：在 GitHub 上创建新的仓库，并将本地项目推送至 GitHub。

### 5.3 配置自动化

1. 配置 GitHub Actions：添加 `.github/workflows/update.yml` 文件，配置 GitHub Actions 工作流，定期运行抓取脚本并自动部署。

### 5.4 上线

1. 启用 GitHub Pages：在 GitHub 仓库 **Settings > Pages** 中启用 GitHub Pages，指向 `gh-pages` 分支或 `docs/` 文件夹（依据 Hugo 部署动作配置）。
2. 访问 `https://<username>.github.io/aibase-clone/` 查看网站。

### 5.5 持续运营

1. **监控 GitHub Actions 运行日志**：使用 GitHub Actions 日志检查每次抓取和部署是否成功。
2. **使用 Google Search Console 监控索引状态和搜索表现**：提交 `sitemap.xml` 至 Google Search Console，跟踪网站表现。
3. **根据数据反馈优化内容和 SEO 策略**：分析流量数据，调整内容策略，提升用户体验。

## 6. 成功指标（KPIs）

- **SEO 表现**：
  - 谷歌索引页面数 > 1000 个页面。
  - 目标关键词进入 Google 前 10 页。
  - 自然搜索流量（基于 Google Analytics）。
- **内容规模**：
  - 收录 AI 工具数量 > 500。
  - 发布资讯文章数量 > 100。
  - 热门大模型数量 > 50。
  - MCP 服务数量 > 50。
- **用户参与**：
  - 平均页面停留时间。
  - 跳出率。
