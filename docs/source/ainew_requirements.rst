AI 工具导航与资讯网站需求文档
============================

1. 项目概述
----------

1.1 项目名称
~~~~~~~~~~~~

- 暂定项目名称：**AInew**

1.2 项目目标
~~~~~~~~~~~~

- 构建一个高质量的 AI 工具和资讯聚合平台，通过搜索引擎优化（SEO）获取可持续的自然流量。
- 提供简洁且有用的信息，帮助用户发现最新的 AI 工具、了解行业动态、探索热门大模型及 MCP（Managed Cloud Platforms）服务导航。

1.3 目标用户
~~~~~~~~~~~~

- 对人工智能技术感兴趣的普通用户。
- 寻找效率工具的产品经理、设计师、开发者、内容创作者等专业人士。
- 希望了解 AI 行业动态的学生和研究者。

1.4 核心价值主张
~~~~~~~~~~~~~~~~~~

- **全面聚合**：一站式发现数千款 AI 工具，涵盖写作、设计、视频、编程、营销等全场景。
- **信息及时**：通过自动化系统，每日更新最新发布的 AI 工具和行业资讯。
- **搜索友好**：网站结构和内容高度优化，确保在 Google、Bing 等搜索引擎中获得良好排名。
- **零成本访问**：完全免费使用，无广告干扰（初期）。

2. 功能需求
----------

2.1 核心功能模块
~~~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 模块
     - 功能描述
   * - AI 工具导航
     -
       - 展示所有收录的 AI 工具。
       - 每个工具拥有独立详情页，包含：名称、Logo、简介、官网链接、价格（免费/付费）、支持平台、适用场景、标签、用户评分（可选）。
       - 支持按 **分类 (Category)**、**标签 (Tag)**、**价格** 进行筛选。
       - 支持关键词搜索。
   * - AI 资讯博客
     -
       - 发布关于 AI 领域的新闻、趋势分析、工具评测文章。
       - 每篇文章有独立页面，包含标题、摘要、作者、发布时间、正文、相关标签。
       - 支持文章列表按时间倒序排列。
   * - 热门大模型
     -
       - 列出当前最流行或最具影响力的大规模机器学习模型。
       - 包括模型介绍、应用场景、性能指标等。
   * - MCP 服务导航
     -
       - 提供各大云服务商的 AI 相关服务概览。
       - 包含服务名称、提供商、简介、定价、使用案例等。
   * - 首页展示
     -
       - 轮播图或精选区：展示热门或新上架的 AI 工具、大模型、MCP 服务。
       - 最新工具列表：显示最近添加的 AI 工具。
       - 最新资讯列表：显示最近发布的文章。
       - 分类导航快捷入口。

2.2 内容管理需求
~~~~~~~~~~~~~~~~~~

预先抓取内容填充
^^^^^^^^^^^^^^^^^^

- **RSS Feeds 和 API 数据源**：

  - TechCrunch AI: https://techcrunch.com/tag/artificial-intelligence/feed/
  - VentureBeat AI: https://venturebeat.com/category/ai/feed/
  - Product Hunt API: 获取 AI 类别新品。
  - Google News API: 根据关键词检索 AI 新闻。
  - ArXiv API: 获取最新的 AI 研究论文。

- **脚本实现**：

  - 编写 Python 脚本，使用 ``feedparser`` 库解析 RSS Feed，使用 ``requests`` 库调用 API 接口。
  - 解析数据后，生成符合 Hugo 格式的 Markdown 文件，存入对应目录（``content/ai-tools/`` 、 ``content/ai-news/`` 、 ``content/models/`` 、 ``content/mcp-services/`` ）。

自动更新内容
^^^^^^^^^^^^

- **定时任务**：

  - 使用 GitHub Actions 定期运行上述脚本，确保内容的实时性。

- **示例 GitHub Actions 工作流 ``update.yml``**：

  .. code-block:: yaml

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

2.3 SEO 专项需求
~~~~~~~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 需求
     - 技术实现方案
   * - 静态页面生成
     - 使用 Hugo SSG 生成纯 HTML 页面，确保爬虫可高效抓取所有内容。
   * - 语义化 URL
     - 生成对 SEO 友好的 URL，例如：``/ai-tools/chatgpt`` 、 ``/ai-news/openai-launches-new-model`` 、 ``/models/gpt-3`` 、 ``/mcp-services/aws-sagemaker``。
   * - Meta 标签优化
     - 为每一页自动生成唯一的 ``<title>`` 和 ``<meta name="description">``。
   * - 结构化数据 (Schema)
     - 在页面中嵌入 JSON-LD 格式的 Schema Markup：

       - 工具页：``SoftwareApplication``
       - 文章页：``Article``
       - 大模型页：``Dataset`` 或 ``TechArticle``
       - MCP 服务页：``Service``
   * - 站点地图 (Sitemap)
     - 自动生成 ``sitemap.xml`` 并提交至 Google Search Console。
   * - Robots 协议
     - 生成 ``robots.txt`` 文件，指导爬虫行为。
   * - 性能优化
     - 图片懒加载、压缩资源、利用 CDN（GitHub Pages）实现快速加载。

3. 技术架构
----------

3.1 技术栈
~~~~~~~~~~

.. list-table::
   :header-rows: 1
   :widths: 25 35 40

   * - 层级
     - 技术选择
     - 理由
   * - 前端框架
     - `Hugo <https://gohugo.io/>`_ (静态网站生成器)
     - 极致性能、简单易学、强大的模板系统、天生 SEO 友好。
   * - 托管平台
     - `GitHub Pages <https://pages.github.com/>`_
     - 免费、稳定、全球 CDN 加速、与 GitHub 生态无缝集成。
   * - 内容存储
     - Markdown 文件 + Git 仓库
     - 内容即代码，版本控制清晰，易于管理和协作。
   * - 自动化引擎
     - `GitHub Actions <https://github.com/features/actions>`_
     - 免费定时任务，可运行脚本实现内容抓取、网站构建和部署。
   * - 内容抓取
     - Python 脚本（使用 ``feedparser`` 、 ``requests`` 、 ``BeautifulSoup`` 等库）
     - 灵活强大，易于处理各种数据源。
   * - 图片存储
     - 专业免费图床（如 路过图床、SM.MS、z.run）
     - 保持 GitHub 仓库轻量，利用专业 CDN 提升图片加载速度。
   * - 主题
     - `Hugo Themes <https://themes.gohugo.io/>`_ 中选择或定制
     - 快速搭建美观界面，重点关注 SEO 和响应式设计。

3.2 系统架构图
~~~~~~~~~~~~~~

.. code-block:: text

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
                     |   - scripts/  |
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

4. 非功能性需求
--------------

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - 需求
     - 描述
   * - 性能
     - 页面加载速度应极快（Lighthouse 性能评分 > 90），所有静态资源通过 CDN 分发。
   * - 可用性
     - 网站 7×24 小时可用，GitHub Pages SLA 高。
   * - 可维护性
     - 代码结构清晰，文档齐全，自动化程度高，降低长期维护成本。
   * - 可扩展性
     - 架构支持未来添加新功能，如用户评论、收藏夹、API 接口等。
   * - 安全性
     - 静态网站本身攻击面小。依赖的第三方库需定期更新。

5. 开发与部署流程
----------------

5.1 预先抓取内容填充
~~~~~~~~~~~~~~~~~~~~

1. **编写抓取脚本**：创建 Python 脚本 ``scripts/fetch_initial_content.py``，用于从选定的数据源抓取初始内容并生成 Markdown 文件。

   .. code-block:: python

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
              file.write(f"""---\n"""
                         f"title: \"{title}\"\n"
                         f"link: \"{link}\"\n"
                         f"date: \"{published}\"\n"
                         f"featured_image: \"{hosted_image_url}\"\n\n"
                         f"{summary}\n")

      rss_feeds = [
          'https://techcrunch.com/tag/artificial-intelligence/feed/',
          'https://venturebeat.com/category/ai/feed/'
      ]

      for rss_feed in rss_feeds:
          feed = fetch_rss_feed(rss_feed)
          for entry in feed.entries:
              generate_markdown(entry, 'ai-news')

2. **运行脚本**：执行 ``python scripts/fetch_initial_content.py``，将生成的 Markdown 文件添加到 Git 仓库中。

5.2 初始化仓库
~~~~~~~~~~~~~~

1. 创建 GitHub 仓库：在 GitHub 上创建新的仓库，并将本地项目推送至 GitHub。

5.3 配置自动化
~~~~~~~~~~~~~~

1. 配置 GitHub Actions：添加 ``.github/workflows/update.yml`` 文件，配置 GitHub Actions 工作流，定期运行抓取脚本并自动部署。

5.4 上线
~~~~~~~~

1. 启用 GitHub Pages：在 GitHub 仓库 **Settings > Pages** 中启用 GitHub Pages，指向 ``gh-pages`` 分支或 ``docs/`` 文件夹（依据 Hugo 部署动作配置）。
2. 访问 ``https://<username>.github.io/aibase-clone/`` 查看网站。

5.5 持续运营
~~~~~~~~~~~~

1. **监控 GitHub Actions 运行日志**：使用 GitHub Actions 日志检查每次抓取和部署是否成功。
2. **使用 Google Search Console 监控索引状态和搜索表现**：提交 ``sitemap.xml`` 至 Google Search Console，跟踪网站表现。
3. **根据数据反馈优化内容和 SEO 策略**：分析流量数据，调整内容策略，提升用户体验。

6. 成功指标 (KPIs)
------------------

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

7. 网站项目结构与实现
------------------

7.1 文件与目录结构
~~~~~~~~~~~~~~~~~~

.. code-block:: text

   ainew/
     ├── archetypes/
     │   └── default.md
     ├── assets/
     ├── config/
     │   └── _default/
     │       ├── config.toml
     │       ├── menus.toml
     │       └── params.toml
     ├── content/
     │   ├── ai-news/
     │   ├── ai-tools/
     │   ├── mcp-services/
     │   └── models/
     ├── data/
     │   └── curated/
     ├── layouts/
     │   ├── _default/
     │   ├── ai-news/
     │   ├── ai-tools/
     │   ├── mcp-services/
     │   └── models/
     ├── static/
     │   ├── images/
     │   └── uploads/
     ├── themes/
     │   └── ainew/
     ├── scripts/
     │   └── update_content.py
     ├── .github/
     │   └── workflows/
     │       └── update.yml
     └── package.json

- ``archetypes/``：定义默认 Front Matter，保证新建内容字段齐全。
- ``assets/`` 与 ``static/``：存放全局样式、脚本和静态资源，配合 Hugo Pipes 进行压缩与指纹化。
- ``config/_default/``：拆分配置文件，方便根据不同环境覆盖参数。
- ``content/``：按照业务域划分内容目录，便于 Hugo 自动生成语义化 URL。
- ``layouts/``：存放全站模板与模块化 Partial，例如导航、页脚、结构化数据等。
- ``themes/ainew/``：可从开源主题衍生，集中管理组件与样式，以便后续拆分为独立主题仓库。
- ``scripts/update_content.py``：负责抓取、去重、格式化并写入 Markdown。
- ``.github/workflows/update.yml``：调度自动化抓取、构建与部署流程。
- ``package.json``：集中管理前端构建任务（如 TailwindCSS、PostCSS、图像压缩）。

7.2 Hugo 基础配置
~~~~~~~~~~~~~~~~~

.. code-block:: toml

   baseURL = "https://ainew.io/"
   languageCode = "zh-cn"
   title = "AInew — AI 工具导航与资讯"
   theme = "ainew"
   paginate = 12
   enableRobotsTXT = true
   defaultContentLanguage = "zh-cn"
   hasCJKLanguage = true
   summaryLength = 180
   canonifyURLs = true

   [outputs]
     home = ["HTML", "RSS", "SITEMAP", "JSON"]
     section = ["HTML", "RSS"]
     taxonomy = ["HTML"]
     term = ["HTML", "RSS"]

   [taxonomies]
     category = "categories"
     tag = "tags"
     pricing = "pricings"

   [params]
     siteDescription = "AI 工具与资讯一站式导航，覆盖工具、行业新闻、热门大模型与云服务。"
     featuredToolsLimit = 6
     featuredModelsLimit = 6
     latestNewsLimit = 8
     socialPreviewImage = "images/og-default.png"
     githubRepo = "your-name/ainew"
     contactEmail = "hello@ainew.io"

   [markup.goldmark.renderer]
     unsafe = true

   [[menus.main]]
     name = "AI 工具"
     url = "/ai-tools/"
     weight = 10

   [[menus.main]]
     name = "资讯"
     url = "/ai-news/"
     weight = 20

   [[menus.main]]
     name = "热门大模型"
     url = "/models/"
     weight = 30

   [[menus.main]]
     name = "MCP 导航"
     url = "/mcp-services/"
     weight = 40

   [[params.schema]]
     type = "Organization"
     name = "AInew"
     url = "https://ainew.io/"
     logo = "https://ainew.io/images/logo.png"

7.3 内容类型与前置数据
~~~~~~~~~~~~~~~~~~~~~~

AI 工具（``content/ai-tools/``）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- 目标：提供统一的工具介绍页面，强调适用场景、优势与价格。
- URL 结构：``/ai-tools/<slug>/``。

.. code-block:: yaml

   ---
   title: "OpenAI ChatGPT"
   slug: "chatgpt"
   description: "基于 GPT-4 的智能对话助手，支持多轮问答与插件扩展。"
   link: "https://chat.openai.com/"
   price: "freemium"
   categories:
     - 文本创作
     - 办公效率
   tags:
     - GPT
     - Chatbot
   platforms:
     - Web
     - iOS
     - Android
   use_cases:
     - 文案撰写
     - 技术问答
   hero_image: "https://cdn.example.com/images/chatgpt.png"
   rating: 4.7
   schema_type: "SoftwareApplication"
   last_synced: 2024-10-01
   draft: false
   ---

   ## 核心亮点
   - 结合插件市场，调用第三方服务解决复杂任务。
   - 支持多种语言与代码生成场景。

AI 资讯（``content/ai-news/``）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- 目标：收录每日行业动态与趋势分析。
- URL 结构：``/ai-news/<yyyy>/<mm>/<slug>/``。

.. code-block:: yaml

   ---
   title: "OpenAI 发布 GPT-4.5，推理速度提升 30%"
   slug: "openai-releases-gpt-4-5"
   date: 2024-10-01T08:00:00+08:00
   author: "AInew 编辑部"
   summary: "最新模型在推理速度与多模态理解上实现显著提升。"
   source: "TechCrunch"
   link: "https://techcrunch.com/..."
   tags:
     - OpenAI
     - 大模型
   schema_type: "Article"
   featured_image: "https://cdn.example.com/news/openai-gpt45.jpg"
   draft: false
   ---

   正文采用 Markdown 编写，可嵌入 ``{{< figure >}}`` 与 ``{{< youtube >}}`` 等短代码。

热门大模型（``content/models/``）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- 目标：展示领先模型的能力指标与应用案例。
- URL 结构：``/models/<slug>/``。

.. code-block:: yaml

   ---
   title: "GPT-4.5 Turbo"
   slug: "gpt-4-5-turbo"
   vendor: "OpenAI"
   release_date: 2024-09-28
   parameters: "1.8T"
   input_modalities:
     - 文本
     - 图像
   benchmarks:
     - name: "MMLU"
       score: 90.1
     - name: "BBH"
       score: 85.4
   strengths:
     - "推理速度提升 30%"
     - "插件生态更加成熟"
   limitations:
     - "暂不支持离线部署"
   schema_type: "TechArticle"
   documentation: "https://platform.openai.com/docs/models"
   draft: false
   ---

MCP 服务（``content/mcp-services/``）
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- 目标：帮助用户比较云端 AI 服务的能力、定价与集成方式。
- URL 结构：``/mcp-services/<provider>/<slug>/``。

.. code-block:: yaml

   ---
   title: "AWS SageMaker"
   slug: "aws-sagemaker"
   provider: "Amazon Web Services"
   tiers:
     - name: "按需训练"
       price: "按使用计费"
     - name: "Serverless Inference"
       price: "按调用计费"
   regions:
     - us-east-1
     - eu-west-1
   integrations:
     - "Amazon S3"
     - "AWS Lambda"
   use_cases:
     - "大规模模型训练"
     - "批量推理"
   schema_type: "Service"
   service_level: "企业级"
   contact: "https://aws.amazon.com/sagemaker/pricing/"
   draft: false
   ---

7.4 模板与组件划分
~~~~~~~~~~~~~~~~~~

- ``layouts/_default/baseof.html``：定义全局骨架，注入导航、页脚、结构化数据与追踪脚本。
- ``layouts/partials/structured-data.html``：根据 ``schema_type`` 渲染 JSON-LD。
- ``layouts/ai-tools/single.html``：呈现工具详情页，包含评价、按钮与相似工具推荐。
- ``layouts/partials/search-form.html``：站内搜索与筛选 UI，复用在工具列表页与首页。
- ``layouts/index.html``：首页聚合最新工具、资讯、热门模型与精选服务。
- ``layouts/_default/list.html``：统一处理分页、面包屑与 Meta 标签。

8. 自动化与数据抓取
------------------

8.1 ``scripts/update_content.py`` 实现要点
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code-block:: python

   import json
   import logging
   import os
   from datetime import datetime
   from pathlib import Path
   from typing import Iterable, Optional

   import feedparser
   import requests
   import yaml
   from slugify import slugify

   BASE_DIR = Path(__file__).resolve().parent.parent
   CONTENT_DIR = BASE_DIR / "content"
   DATA_CONFIG = BASE_DIR / "config" / "feeds.json"

   logging.basicConfig(level=logging.INFO, format="%(levelname)s %(message)s")

   def load_sources() -> dict:
       with DATA_CONFIG.open(encoding="utf-8") as fh:
           return json.load(fh)

   def fetch_feed(url: str) -> Iterable[feedparser.FeedParserDict]:
       logging.info("Fetching feed %s", url)
       feed = feedparser.parse(url)
       return feed.entries

   def fetch_json(url: str, headers: Optional[dict] = None) -> dict:
       response = requests.get(url, headers=headers, timeout=20)
       response.raise_for_status()
       return response.json()

   def write_markdown(directory: Path, slug: str, front_matter: dict, body: str) -> None:
       directory.mkdir(parents=True, exist_ok=True)
       filepath = directory / f"{slug}.md"
       front_matter_text = yaml.safe_dump(
           front_matter,
           allow_unicode=True,
           sort_keys=False,
       ).strip()
       content = f"---\n{front_matter_text}\n---\n\n{body.strip()}\n"
       filepath.write_text(content, encoding="utf-8")
       logging.info("Written %s", filepath)

   def process_news(entry) -> None:
       slug = slugify(entry.title)
       front_matter = {
           "title": entry.title,
           "slug": slug,
           "date": entry.get("published", datetime.utcnow().isoformat()),
           "summary": entry.get("summary", "")[:240],
           "link": entry.link,
           "source": entry.get("source", {}).get("title", ""),
           "schema_type": "Article",
       }
       body = entry.get("summary", "")
       write_markdown(CONTENT_DIR / "ai-news", slug, front_matter, body)

   def main() -> None:
       sources = load_sources()
       for rss in sources.get("rss", []):
           for entry in fetch_feed(rss):
               process_news(entry)

   if __name__ == "__main__":
       main()

关键实践：

- 使用 ``slugify`` 保证 URL 稳定性，并避免重复。
- ``write_markdown`` 方法统一输出 Front Matter 与正文，方便拓展至工具、模型与服务。
- 通过 ``feeds.json`` 管理数据源，支持新增 RSS/API 而无需修改脚本。
- 结合 GitHub Actions 缓存与 ``git diff``，仅提交新增或更新的 Markdown 文件。

8.2 数据源配置示例
~~~~~~~~~~~~~~~~~~

.. code-block:: json

   {
     "rss": [
       "https://techcrunch.com/tag/artificial-intelligence/feed/",
       "https://venturebeat.com/category/ai/feed/"
     ],
     "product_hunt": {
       "url": "https://api.producthunt.com/v2/api/graphql",
       "token_env": "PRODUCT_HUNT_TOKEN"
     },
     "google_news": {
       "keyword": "artificial intelligence",
       "region": "US"
     },
     "arxiv": {
       "query": "cat:cs.AI",
       "max_results": 25
     }
   }

8.3 监控与告警
~~~~~~~~~~~~~

- 在 GitHub Actions 中开启 ``on: workflow_dispatch``，遇到失败可手动重试。
- 通过 ``actions/cache`` 缓存 RSS/JSON 结果，缩短构建时间。
- 将抓取日志输出至 ``artifacts``，便于排查失败原因。
- 结合 UptimeRobot 或 Cronitor 监控 GitHub Pages 站点可用性。

9. 本地开发与部署
----------------

9.1 环境准备
~~~~~~~~~~~~

1. 安装 ``Hugo Extended``（版本 >= 0.124）。
2. 安装 Python 3.10+，执行 ``python -m venv .venv`` 创建虚拟环境。
3. 在虚拟环境中运行 ``pip install -r scripts/requirements.txt``（包含 ``feedparser``、``requests``、``python-slugify``、``PyYAML`` 等依赖）。
4. 可选：使用 ``npm`` 或 ``pnpm`` 安装前端依赖（如 TailwindCSS、Autoprefixer）。

9.2 本地开发流程
~~~~~~~~~~~~~~~~

1. 运行 ``python scripts/update_content.py`` 生成最新内容。
2. 执行 ``hugo server -D`` 启动本地开发服务器，默认地址为 ``http://localhost:1313``。
3. 修改 ``layouts/``、``assets/`` 或 ``content/`` 后页面将热重载。
4. 若使用 TailwindCSS，执行 ``npm run dev`` 开启监听并生成 CSS。

9.3 构建与发布
~~~~~~~~~~~~~~

1. 运行 ``hugo --gc --minify`` 产出静态文件目录 ``public/``。
2. 在本地验证核心页面：首页、工具列表、资讯列表、模型列表、MCP 服务详情。
3. 将 ``public/`` 上传至 GitHub Pages（由 Actions 自动完成）或备份至任意对象存储。
4. 发布后在 Google Search Console 和 Bing Webmaster 提交最新 ``sitemap.xml``。

9.4 质量保障清单
~~~~~~~~~~~~~~~~

- 检查 Lighthouse 指标（Performance > 90，SEO > 90）。
- 验证结构化数据无错误（使用 ``https://search.google.com/test/rich-results``）。
- 确保所有外链开启 ``rel=\"noopener\"``，并在新标签页打开。
- 针对移动端与桌面端分别进行响应式测试。
