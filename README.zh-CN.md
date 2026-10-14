# Awesome Doris AI（中文版）

> 🌟 收录 [Apache Doris](https://doris.apache.org/) AI 相关的原生工具、MCP 服务器、Agent、Skills、教程与实践示例的精选导航。

**[English](./README.md)**

自 4.x 起，[Apache Doris](https://doris.apache.org/) 在同一个实时分析引擎中原生集成了全文检索、向量搜索、AI 函数与 Agent 生态。本仓库把这些分散的能力、官方/社区项目、教程与可运行示例整理成一张「Doris × AI」实用地图。

---

## 目录

- [官方资源](#官方资源)
- [MCP Server 与 Agent 集成](#mcp-server-与-agent-集成)
- [Agent CLI 与 Skills](#agent-cli-与-skills)
- [向量 / 全文 / 混合检索](#向量--全文--混合检索)
- [AI 函数与 LLM 集成](#ai-函数与-llm-集成)
- [教程与博客](#教程与博客)
- [示例与开源实践](#示例与开源实践)
- [社区与活动](#社区与活动)
- [贡献](#贡献)
- [许可](#许可)

---

## 官方资源

> Apache Doris / SelectDB 官方提供的与 AI 相关的仓库、文档与能力汇总。

| 项目 | Stars | 说明 | Doris 版本 |
| --- | --- | --- | --- |
| [apache/doris](https://github.com/apache/doris) | ★16.1k | Apache Doris 主仓库（含向量检索、AI 函数等） | 4.x |
| [apache/doris-mcp-server](https://github.com/apache/doris-mcp-server) | ★352 | 官方 Doris MCP Server，让 LLM/Agent 通过 MCP 读写 Doris | 3.0+ |
| [apache/doris-skills](https://github.com/apache/doris-skills) | ★48 | 官方 Agent skills，供 Claude/Copilot 等调用 Doris 能力 | dev |
| [apache/doris-cli](https://github.com/apache/doris-cli) | ★17 | 官方 Doris CLI，面向 Agent 的命令行接口 | dev |
| [Doris 官方文档 - AI 章节](https://doris.apache.org/docs/dev/key-features/llm-sql-functions) | — | LLM SQL Functions、向量索引、MCP Server 等官方文档 | 4.x |

## MCP Server 与 Agent 集成

> 通过 Model Context Protocol 把 Doris 接入 Claude、Copilot、DeepSeek Harness 等 AI Agent。

| 项目 | Stars | 说明 |
| --- | --- | --- |
| [apache/doris-mcp-server](https://github.com/apache/doris-mcp-server) | ★352 | 官方 MCP Server（重点推荐，见上） |
| [morningman/mcp-doris](https://github.com/morningman/mcp-doris) | ★15 | 社区 MCP Server for Apache Doris & VeloDB（PMC 成员） |
| [lijingfz/doris-mcp-agentcore](https://github.com/lijingfz/doris-mcp-agentcore) | — | 部署在 AWS Bedrock AgentCore Runtime 的 Doris MCP |

**快速上手**

```bash
# 官方 MCP server 运行示例
uvx mcp-server-doris --host localhost --port 9030 --username root --password 123456
```

## Agent CLI 与 Skills

| 项目 | Stars | 说明 |
| --- | --- | --- |
| [apache/doris-cli](https://github.com/apache/doris-cli) | ★17 | 官方 Agent CLI |
| [apache/doris-skills](https://github.com/apache/doris-skills) | ★48 | 官方 Agent Skills |
| [velodb/agent-skills](https://github.com/velodb/agent-skills) | ★3 | VeloDB（Doris 商业版）的 Agent skills |

## 向量 / 全文 / 混合检索

> Doris 4.x 在同一引擎内原生集成倒排索引、BM25 全文检索与 HNSW 向量检索。

**核心能力**

- 全文检索：`MATCH_*` / `SEARCH()` / `score()`（BM25）
- 向量检索：`ARRAY` 向量列 + HNSW 索引 + `l2_distance_approximate` / `inner_product_approximate`
- 混合检索：先结构化/文本过滤，再向量 TopN

**快速上手**

```sql
-- HNSW 向量索引建表
CREATE TABLE doc_store (
  id BIGINT,
  title STRING,
  embedding ARRAY<FLOAT> NOT NULL,
  INDEX idx_vec (embedding) USING ANN PROPERTIES (
    "index_type" = "hnsw", "metric_type" = "l2_distance", "dim" = "768"
  )
) DUPLICATE KEY(id) DISTRIBUTED BY HASH(id) BUCKETS 16;

-- 向量 TopN 检索
SELECT id, l2_distance_approximate(embedding, [...] ) AS dist
FROM doc_store ORDER BY dist ASC LIMIT 10;
```

## AI 函数与 LLM 集成

> Doris 内置一组 AI 函数，可在 SQL 中直接调用外部大语言模型（LLM）完成文本智能分析，无需将数据导出到外部应用。分析师即可在数据库内部完成文本分类、信息提取、情感分析、语法纠错、内容生成、敏感信息脱敏、相似度计算、摘要、翻译以及跨行聚合等任务。
>
> 参考官方文档：[AI 函数总览](https://doris.apache.org/zh-CN/docs/dev/ai/ai-function-overview)

**按场景选择函数**

| 我想做什么 | 推荐函数 | 返回结果 |
| --- | --- | --- |
| 在给定标签中选出最匹配的一个 | [`AI_CLASSIFY`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-classify) | 单个标签字符串 |
| 按标签从文本中提取信息 | [`AI_EXTRACT`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-extract) | 每个标签对应的提取内容 |
| 判断文本是否符合某个语义条件 | [`AI_FILTER`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-filter) | `BOOLEAN` |
| 修复文本中的语法、拼写错误 | [`AI_FIXGRAMMAR`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-fixgrammar) | 修正后的文本 |
| 基于参数内容生成新文本 | [`AI_GENERATE`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-generate) | 生成的文本 |
| 对原文中的敏感信息进行脱敏 | [`AI_MASK`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-mask) | 将敏感信息替换为 `[MASKED]` 后的文本 |
| 分析文本情感倾向 | [`AI_SENTIMENT`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-sentiment) | `positive` / `negative` / `neutral` / `mixed` |
| 计算两段文本的语义相似度 | [`AI_SIMILARITY`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-similarity) | 0–10 的浮点数，越大越相似 |
| 对单段文本进行高度概括 | [`AI_SUMMARIZE`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-summarize) | 摘要文本 |
| 将文本翻译为指定语言 | [`AI_TRANSLATE`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/ai-functions/ai-translate) | 翻译后的文本 |
| 对多行文本进行跨行聚合分析 | [`AI_AGG`](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-functions/aggregate-functions/ai-agg) | 聚合后的文本 |

### 接入大模型：配置 AI 资源

Doris 通过[资源（Resource）机制](https://doris.apache.org/zh-CN/docs/dev/sql-manual/sql-statements/cluster-management/compute-management/CREATE-RESOURCE)集中管理 AI API 访问，统一配置厂商、模型、密钥与端点，确保密钥安全和权限可控。

**资源参数**

| 参数 | 是否必填 | 说明 |
| --- | --- | --- |
| `type` | 必填 | 必须为 `ai` |
| `ai.provider_type` | 必填 | 外部 AI 厂商类型 |
| `ai.endpoint` | 必填 | AI API 接口地址 |
| `ai.model_name` | 必填 | 模型名称 |
| `ai.api_key` | 除 `local` 外必填 | API 密钥 |
| `ai.temperature` | 可选 | 控制随机性，0–1；默认 `-1`（不设置）|
| `ai.max_tokens` | 可选 | 默认 `-1`（不设置）；Anthropic 默认 `2048` |
| `ai.max_retries` | 可选 | 默认 `3` |
| `ai.retry_delay_second` | 可选 | 默认 `0` |

**支持的厂商**：OpenAI · Anthropic · Gemini · DeepSeek · Local · MoonShot · MiniMax · Zhipu · Qwen · Baichuan。若厂商不在列表中，但其 API 格式与 OpenAI / Anthropic / Gemini 相同，可选择三者中格式相同的 `ai.provider_type`。

### 快速上手

**步骤 1：创建 AI 资源**（OpenAI 与 DeepSeek 示例）

```sql
-- OpenAI
CREATE RESOURCE 'openai_example' PROPERTIES (
  'type' = 'ai', 'ai.provider_type' = 'openai',
  'ai.endpoint' = 'https://api.openai.com/v1/responses',
  'ai.model_name' = 'gpt-4.1', 'ai.api_key' = 'xxxxx'
);

-- DeepSeek
CREATE RESOURCE 'deepseek_example' PROPERTIES (
  'type' = 'ai', 'ai.provider_type' = 'deepseek',
  'ai.endpoint' = 'https://api.deepseek.com/chat/completions',
  'ai.model_name' = 'deepseek-chat', 'ai.api_key' = 'xxxxx'
);
```

**步骤 2：设置默认资源（可选）**

```sql
SET default_ai_resource = 'ai_resource_name';
```

**步骤 3：在 SQL 中调用 AI 函数**

*示例 A：语义相关性评分 —— 返回与 Apache Doris 相关性最高的 10 条文档*

```sql
SELECT
  c,
  CAST(AI_GENERATE(CONCAT(
    'Please score the relevance of the following document content to Apache Doris, ',
    'with a floating-point number from 0 to 10, output only the score. Document:', c
  )) AS DOUBLE) AS score
FROM doc_pool
ORDER BY score DESC
LIMIT 10;
```

*示例 B：候选人 / 岗位语义匹配 —— 用 `AI_FILTER` 筛选合适候选人*

```sql
SELECT c.candidate_id, c.name, j.job_id, j.title
FROM candidate_profiles AS c
JOIN job_requirements AS j
WHERE AI_FILTER(CONCAT(
  'Does the following candidate self-introduction match the job description?',
  'Job: ', j.jd_text, ' Candidate: ', c.self_intro
));
```

### 设计原理

- **`<resource_name>`**：当前仅支持传入字符串常量。
- **`system_prompt`**：不同函数使用不同系统提示词；`user_prompt` 仅包含输入参数。
- **请求体**：用户未设置的可选参数（如 `ai.temperature`、`ai.max_tokens`）不会包含在请求体中（Anthropic 除外，必须传递 `max_tokens`，Doris 内部默认 `2048`）。
- **超时控制**：单次请求超时限制与发送时剩余的查询时间一致，总查询时间由会话变量 `query_timeout` 决定，超时可适当延长。
- **厂商抽象**：Doris 为每种服务实现了请求构造、鉴权、响应解析等核心方法，用户只需声明厂商类型。

## 教程与博客

> 按主题组织的高质量中英文教程与官方/社区博客。

- **向量检索实践**：[97% 召回率、900 QPS：Apache Doris 4.1 生产级向量检索的工程实践](https://selectdb.com/blog/1698)
- **AI 集成概览**：[Doris 4.x AI：一站式搞定文本/向量搜索+智能分析](https://cloud.tencent.cn/developer/article/2624355)
- **Agent 运维**：[面向 AI Agent 的数据库运维：SelectDB CLI 与 Skills 架构设计及实践](https://cn.selectdb.com/blog/1741)
- **AI 实操系统**：[Doris & SelectDB for AI 实操：从零搭建非结构化数据智能分析洞察系统](https://bbs.huaweicloud.com/blogs/474525)
- **2026 Roadmap**：[Apache Doris 2026 Roadmap：AI 时代数据基础设施如何选型](https://blog.51cto.com/u_14767244/14823334)

## 示例与开源实践

> 基于 Doris 的可运行开源项目、Demo 与生产实践（RAG、智能分析、LLM 集成等）。

### RAG 与向量存储

| 项目 | Stars | 说明 |
| --- | --- | --- |
| [freemandealer/apache-doris-rag](https://github.com/freemandealer/apache-doris-rag) | ★16 | 基于 Doris 混合检索能力构建的 RAG 系统（Jupyter notebook） |
| [LangChain — Apache Doris 向量存储](https://python.langchain.com/docs/integrations/vectorstores/apache_doris) | — | LangChain 官方集成，将 Doris 用作 RAG 的向量存储 |

**快速上手（LangChain + Doris 作为向量存储）**

```python
from langchain_community.vectorstores.apache_doris import ApacheDoris, ApacheDorisSettings

settings = ApacheDorisSettings()
settings.host = "localhost"; settings.port = 9030
settings.username = "root"; settings.database = "langchain"

docsearch = ApacheDoris.from_documents(split_docs, embeddings, config=settings)
qa = RetrievalQA.from_chain_type(llm=llm, retriever=docsearch.as_retriever())
```

### 智能分析与 Agent

| 项目 | Stars | 说明 |
| --- | --- | --- |
| [DataAgent](https://github.com/3600818203/DataAgent) | ★60 | 多代理协作的智能数据分析系统（面向数仓） |
| [MaloneTalk/DataAgent](https://github.com/MaloneTalk/DataAgent) | 0 | LLM 驱动的自然语言数据代理——JDBC 多数据源 + 语义层（支持 Doris 等分析型库） |

<!--
  更多欢迎补充：
  - 推荐/风控示例系统
  - 非结构化数据处理流水线
  - LLM SQL 函数 Demo
-->

## 社区与活动

- [Apache Doris 官网](https://doris.apache.org/)
- [SelectDB](https://www.selectdb.com/)
- [Doris GitHub](https://github.com/apache/doris)
- ApacheCon / Doris Summit 相关 AI Session

---

## 贡献

<!-- 请提交 PR！说明贡献规范、分类标准、版本标注要求 -->

欢迎提交 PR！请遵循以下规范：

- 每个项目标注 **Stars**、**一句话说明**、**支持的 Doris 版本**（如适用）
- 新加入的内容请附上实际可运行的验证记录（SQL / 命令）
- 遵守 [awesome 列表规范](https://github.com/sindresorhus/awesome)

## 许可

[MIT](./LICENSE)