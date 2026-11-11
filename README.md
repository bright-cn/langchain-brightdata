<div align="center">

# langchain-brightdata

**Bright Data 网页数据 API 的 LangChain 集成**

[![PyPI 版本](https://img.shields.io/pypi/v/langchain-brightdata?color=blue)](https://pypi.org/project/langchain-brightdata/) [![Python](https://img.shields.io/pypi/pyversions/langchain-brightdata)](https://pypi.org/project/langchain-brightdata/) [![许可证：MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT) [![下载量](https://img.shields.io/pypi/dm/langchain-brightdata)](https://pypi.org/project/langchain-brightdata/)

[安装](#安装) • [快速开始](#快速开始) • [工具](#工具) • [配置](#配置) • [资源](#资源)

</div>

---

## 概述

**langchain-brightdata** 为 [Bright Data](https://www.bright.cn/) 的网页数据 API 提供 LangChain 工具，让你的 AI 智能体能够：

- **搜索** - 使用地理位置定向和语言自定义功能查询搜索引擎
- **解锁** - 访问受地理限制或机器人防护的网站
- **抓取** - 从 Amazon、LinkedIn 和 100 多个域名提取结构化数据

---

## 安装

```bash
pip install langchain-brightdata
```

**要求：** Python 3.9+

---

## 快速开始

### 1. 获取 API 密钥

在 [Bright Data](https://www.bright.cn/) 注册，并从控制面板获取 API 密钥。

### 2. 设置身份验证

```python
import os
os.environ["BRIGHT_DATA_API_KEY"] = "your-api-key"
```

或直接传入：

```python
from langchain_brightdata import BrightDataSERP
tool = BrightDataSERP(bright_data_api_key="your-api-key")
```

### 3. 与 LangChain 智能体配合使用

```python
from langchain_brightdata import BrightDataSERP, BrightDataUnlocker, BrightDataWebScraperAPI
from langchain.agents import initialize_agent, AgentType
from langchain_openai import ChatOpenAI

# Initialize tools
tools = [
    BrightDataSERP(),
    BrightDataUnlocker(),
    BrightDataWebScraperAPI()
]

# Create agent
llm = ChatOpenAI(model="gpt-4")
agent = initialize_agent(tools, llm, agent=AgentType.OPENAI_FUNCTIONS)

# Run
agent.run("Search for the latest AI news and summarize the top result")
```

---

## 工具

### BrightDataSERP

获取支持地理位置定向和自定义的搜索引擎结果。

```python
from langchain_brightdata import BrightDataSERP

serp = BrightDataSERP()

# Simple search
results = serp.invoke("latest AI research")

# Advanced search
results = serp.invoke({
    "query": "electric vehicles",
    "country": "de",
    "language": "de",
    "search_type": "news",
    "results_count": 20
})
```

#### 参数

| 参数 | 类型 | 默认值 | 说明 |
|-----------|------|---------|-------------|
| `query` | str | 必填 | 搜索查询 |
| `zone` | str | `"serp"` | Bright Data 区域名称 |
| `search_engine` | str | `"google"` | 搜索引擎（`google`、`bing`、`yahoo`） |
| `country` | str | `"us"` | 两位国家/地区代码 |
| `language` | str | `"en"` | 两位语言代码 |
| `results_count` | int | `10` | 结果数量（最多 100 条） |
| `search_type` | str | `None` | `None`（网页）、`"isch"`（图片）、`"shop"`、`"nws"`（新闻）、`"jobs"` |
| `device_type` | str | `None` | `None`（桌面端）、`"mobile"`、`"ios"`、`"android"` |
| `parse_results` | bool | `False` | 返回结构化 JSON |

---

### BrightDataUnlocker

访问任何公开网站，绕过地理限制和机器人防护。

```python
from langchain_brightdata import BrightDataUnlocker

unlocker = BrightDataUnlocker()

# Simple access
content = unlocker.invoke("https://example.com")

# With options
content = unlocker.invoke({
    "url": "https://example.com/restricted",
    "country": "gb",
    "data_format": "markdown"
})
```

#### 参数

| 参数 | 类型 | 默认值 | 说明 |
|-----------|------|---------|-------------|
| `url` | str | 必填 | 要访问的 URL |
| `zone` | str | `"unlocker"` | Bright Data 区域名称 |
| `country` | str | `None` | 两位国家/地区代码 |
| `data_format` | str | `None` | `None`（HTML）、`"markdown"`、`"screenshot"` |

---

### BrightDataWebScraperAPI

从热门网站提取结构化数据。

```python
from langchain_brightdata import BrightDataWebScraperAPI

scraper = BrightDataWebScraperAPI()

# Amazon product
product = scraper.invoke({
    "url": "https://www.amazon.com/dp/B08L5TNJHG",
    "dataset_type": "amazon_product"
})

# LinkedIn profile
profile = scraper.invoke({
    "url": "https://www.linkedin.com/in/satyanadella/",
    "dataset_type": "linkedin_person_profile"
})
```

#### 参数

| 参数 | 类型 | 默认值 | 说明 |
|-----------|------|---------|-------------|
| `url` | str | 必填 | 要抓取的 URL |
| `dataset_type` | str | 必填 | 要提取的数据类型 |
| `zipcode` | str | `None` | 用于指定位置数据的邮政编码 |

#### 支持的数据集类型（44 个数据集）

<details>
<summary><strong>电商（10 个数据集）</strong></summary>

| 类型 | 说明 | 必填输入 |
|------|-------------|-----------------|
| `amazon_product` | 商品详情、价格和规格 | `url`（包含 /dp/） |
| `amazon_product_reviews` | 客户评论和评分 | `url`（包含 /dp/） |
| `amazon_product_search` | Amazon 搜索结果 | `keyword`, `url` |
| `walmart_product` | Walmart 商品数据 | `url`（包含 /ip/） |
| `walmart_seller` | Walmart 卖家信息 | `url` |
| `ebay_product` | eBay 商品数据 | `url` |
| `homedepot_products` | Home Depot 商品数据 | `url` |
| `zara_products` | Zara 商品数据 | `url` |
| `etsy_products` | Etsy 商品数据 | `url` |
| `bestbuy_products` | Best Buy 商品数据 | `url` |

</details>

<details>
<summary><strong>LinkedIn（5 个数据集）</strong></summary>

| 类型 | 说明 | 必填输入 |
|------|-------------|-----------------|
| `linkedin_person_profile` | 职业资料数据 | `url` |
| `linkedin_company_profile` | 公司信息 | `url` |
| `linkedin_job_listings` | 职位列表详情 | `url` |
| `linkedin_posts` | 帖子内容和互动数据 | `url` |
| `linkedin_people_search` | 搜索人员 | `url`, `first_name`, `last_name` |

</details>

<details>
<summary><strong>商业智能（2 个数据集）</strong></summary>

| 类型 | 说明 | 必填输入 |
|------|-------------|-----------------|
| `crunchbase_company` | 公司融资、投资者和指标 | `url` |
| `zoominfo_company_profile` | B2B 公司情报 | `url` |

</details>

<details>
<summary><strong>Instagram（4 个数据集）</strong></summary>

| 类型 | 说明 | 必填输入 |
|------|-------------|-----------------|
| `instagram_profiles` | 个人资料数据和统计信息 | `url` |
| `instagram_posts` | 帖子内容和互动数据 | `url` |
| `instagram_reels` | Reels 内容和指标 | `url` |
| `instagram_comments` | 帖子评论 | `url` |

</details>

<details>
<summary><strong>Facebook（4 个数据集）</strong></summary>

| 类型 | 说明 | 必填输入 |
|------|-------------|-----------------|
| `facebook_posts` | 帖子内容和互动数据 | `url` |
| `facebook_marketplace_listings` | Marketplace 商品信息数据 | `url` |
| `facebook_company_reviews` | 公司评价 | `url`, `num_of_reviews` |
| `facebook_events` | 活动详情 | `url` |

</details>

<details>
<summary><strong>TikTok（4 个数据集）</strong></summary>

| 类型 | 说明 | 必填输入 |
|------|-------------|-----------------|
| `tiktok_profiles` | 个人资料数据和统计信息 | `url` |
| `tiktok_posts` | 视频内容和指标 | `url` |
| `tiktok_shop` | 商店商品数据 | `url` |
| `tiktok_comments` | 视频评论 | `url` |

</details>

<details>
<summary><strong>YouTube（3 个数据集）</strong></summary>

| 类型 | 说明 | 必填输入 |
|------|-------------|-----------------|
| `youtube_profiles` | 频道资料数据 | `url` |
| `youtube_videos` | 视频内容和指标 | `url` |
| `youtube_comments` | 视频评论 | `url`、`num_of_comments`（默认值：10） |

</details>

<details>
<summary><strong>Google（3 个数据集）</strong></summary>

| 类型 | 说明 | 必填输入 |
|------|-------------|-----------------|
| `google_maps_reviews` | 地图中的商家评价 | `url`、`days_limit`（默认值：3） |
| `google_shopping` | 购物商品数据 | `url` |
| `google_play_store` | 应用商店数据 | `url` |

</details>

<details>
<summary><strong>其他平台（9 个数据集）</strong></summary>

| 类型 | 说明 | 必填输入 |
|------|-------------|-----------------|
| `apple_app_store` | iOS 应用数据 | `url` |
| `x_posts` | X（Twitter）帖子数据 | `url` |
| `reddit_posts` | Reddit 帖子数据 | `url` |
| `github_repository_file` | GitHub 文件内容 | `url` |
| `yahoo_finance_business` | 金融企业数据 | `url` |
| `reuter_news` | 新闻文章数据 | `url` |
| `zillow_properties_listing` | 房地产房源数据 | `url` |
| `booking_hotel_listings` | 酒店房源数据 | `url` |

</details>

---

## 配置

### 区域配置

Bright Data 使用“区域”管理不同的 API 配置。你可以在初始化时或针对每个请求设置区域。

#### 初始化时设置区域

```python
from langchain_brightdata import BrightDataSERP, BrightDataUnlocker

# SERP with custom zone
serp = BrightDataSERP(
    bright_data_api_key="your-api-key",
    zone="my_serp_zone"
)

# Unlocker with custom zone
unlocker = BrightDataUnlocker(
    bright_data_api_key="your-api-key",
    zone="my_unlocker_zone"
)
```

#### 针对每个请求设置区域

```python
# Override zone for a specific request
results = serp.invoke({
    "query": "AI news",
    "zone": "different_zone"
})
```

#### 默认区域

| 工具 | 默认区域 |
|------|--------------|
| `BrightDataSERP` | `serp` |
| `BrightDataUnlocker` | `unlocker` |

> <strong>注意：</strong>区域名称必须与 [Bright Data 控制面板](https://www.bright.cn/cp/zones)中配置的区域一致。

---

## 资源

- [Bright Data 文档](https://docs.brightdata.com/integrations/langchain)
- [LangChain 工具指南](https://docs.langchain.com/oss/python/integrations/providers/brightdata)
- [API 参考](https://docs.brightdata.com/api-reference)

---

## 许可证

MIT 许可证 - 详情请参阅 [LICENSE](LICENSE)。
