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

> Doris 通过 `type = 'ai'` 的 Resource 统一接入 OpenAI / DeepSeek / Gemini / 本地模型等，在 SQL 中直接调用。

**AI 函数**

`AI_CLASSIFY` · `AI_EXTRACT` · `AI_FILTER` · `AI_FIXGRAMMAR` · `AI_GENERATE` · `AI_MASK` · `AI_SENTIMENT` · `AI_SIMILARITY` · `AI_SUMMARIZE` · `AI_TRANSLATE` · `AI_AGG`

**快速上手**

```sql
-- 创建 AI Resource
CREATE RESOURCE "ai_r" PROPERTIES (
  'type' = 'ai', 'ai.provider_type' = 'openai',
  'ai.endpoint' = 'https://...', 'ai.model_name' = 'gpt-4o', 'ai.api_key' = 'sk-xxx'
);
SET default_ai_resource = 'ai_r';

-- SQL 内直接做情感分析
SELECT comment, AI_SENTIMENT(comment) FROM user_comment;
```

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