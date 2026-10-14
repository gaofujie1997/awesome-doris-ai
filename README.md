# Awesome Doris AI

> 🌟 A curated list of AI-native tools, MCP servers, agents, skills, tutorials and practical examples for [Apache Doris](https://doris.apache.org/) — the real-time analytical database.

**[简体中文](./README.zh-CN.md)**

Since version 4.x, [Apache Doris](https://doris.apache.org/) natively integrates full-text search, vector search, AI SQL functions and an agent ecosystem into a single real-time analytical engine. This repository maps those scattered capabilities, official/community projects, tutorials and runnable examples into one practical "Doris × AI" guide.

---

## Contents

- [Official Resources](#official-resources)
- [MCP Servers & Agent Integration](#mcp-servers--agent-integration)
- [Agent CLIs & Skills](#agent-clis--skills)
- [Vector / Full-text / Hybrid Search](#vector--full-text--hybrid-search)
- [AI Functions & LLM Integration](#ai-functions--llm-integration)
- [Tutorials & Blogs](#tutorials--blogs)
- [Examples & Open-source Projects](#examples--open-source-projects)
- [Community & Events](#community--events)
- [Contributing](#contributing)
- [License](#license)

---

## Official Resources

> Official Apache Doris / SelectDB AI-related repositories, documentation and capabilities.

| Project | Stars | Description | Doris version |
| --- | --- | --- | --- |
| [apache/doris](https://github.com/apache/doris) | ★16.1k | Main Apache Doris repo (vector search, AI functions, etc.) | 4.x |
| [apache/doris-mcp-server](https://github.com/apache/doris-mcp-server) | ★352 | Official Doris MCP Server — let LLMs/agents read & write Doris over MCP | 3.0+ |
| [apache/doris-skills](https://github.com/apache/doris-skills) | ★48 | Official agent skills for Claude/Copilot and other agents | dev |
| [apache/doris-cli](https://github.com/apache/doris-cli) | ★17 | Official Doris CLI, agent-facing command line | dev |
| [Doris docs — AI section](https://doris.apache.org/docs/dev/key-features/llm-sql-functions) | — | Official docs: LLM SQL functions, vector index, MCP server | 4.x |

## MCP Servers & Agent Integration

> Connect Doris to AI agents (Claude, Copilot, DeepSeek Harness, etc.) via the Model Context Protocol.

| Project | Stars | Description |
| --- | --- | --- |
| [apache/doris-mcp-server](https://github.com/apache/doris-mcp-server) | ★352 | Official MCP server (recomended, listed above) |
| [morningman/mcp-doris](https://github.com/morningman/mcp-doris) | ★15 | Community MCP server for Apache Doris & VeloDB (PMC member) |
| [lijingfz/doris-mcp-agentcore](https://github.com/lijingfz/doris-mcp-agentcore) | — | Doris MCP deployed on AWS Bedrock AgentCore runtime |

**Quick Start**

```bash
# Run the official MCP server
uvx mcp-server-doris --host localhost --port 9030 --username root --password 123456
```

## Agent CLIs & Skills

| Project | Stars | Description |
| --- | --- | --- |
| [apache/doris-cli](https://github.com/apache/doris-cli) | ★17 | Official agent CLI |
| [apache/doris-skills](https://github.com/apache/doris-skills) | ★48 | Official agent skills |
| [velodb/agent-skills](https://github.com/velodb/agent-skills) | ★3 | Agent skills for VeloDB (Doris commercial edition) |

## Vector / Full-text / Hybrid Search

> Doris 4.x natively integrates inverted index, BM25 full-text search and HNSW vector search in one engine.

**Core capabilities**

- Full-text search: `MATCH_*` / `SEARCH()` / `score()` (BM25)
- Vector search: `ARRAY` column + HNSW index + `l2_distance_approximate` / `inner_product_approximate`
- Hybrid search: structured/FT filtering first, then vector TopN

**Quick Start**

```sql
-- Create a table with an HNSW vector index
CREATE TABLE doc_store (
  id BIGINT,
  title STRING,
  embedding ARRAY<FLOAT> NOT NULL,
  INDEX idx_vec (embedding) USING ANN PROPERTIES (
    "index_type" = "hnsw", "metric_type" = "l2_distance", "dim" = "768"
  )
) DUPLICATE KEY(id) DISTRIBUTED BY HASH(id) BUCKETS 16;

-- Vector TopN retrieval
SELECT id, l2_distance_approximate(embedding, [...] ) AS dist
FROM doc_store ORDER BY dist ASC LIMIT 10;
```

## AI Functions & LLM Integration

> Doris connects to OpenAI / DeepSeek / Gemini / local models via a `type = 'ai'` Resource and calls them directly in SQL.

**AI Functions**

`AI_CLASSIFY` · `AI_EXTRACT` · `AI_FILTER` · `AI_FIXGRAMMAR` · `AI_GENERATE` · `AI_MASK` · `AI_SENTIMENT` · `AI_SIMILARITY` · `AI_SUMMARIZE` · `AI_TRANSLATE` · `AI_AGG`

**Quick Start**

```sql
-- Create an AI Resource
CREATE RESOURCE "ai_r" PROPERTIES (
  'type' = 'ai', 'ai.provider_type' = 'openai',
  'ai.endpoint' = 'https://...', 'ai.model_name' = 'gpt-4o', 'ai.api_key' = 'sk-xxx'
);
SET default_ai_resource = 'ai_r';

-- Sentiment analysis directly in SQL
SELECT comment, AI_SENTIMENT(comment) FROM user_comment;
```

## Tutorials & Blogs

> High-quality tutorials and official/community blog posts, organized by topic.

- **Vector search practice**: [97% recall, 900 QPS: production-grade vector search in Apache Doris 4.1](https://selectdb.com/blog/1698)
- **AI integration overview**: [Doris 4.x AI: full-text & vector search and intelligent analysis in one system](https://cloud.tencent.cn/developer/article/2624355)
- **Agent operations**: [DB operations for AI agents: SelectDB CLI and Skills architecture in practice](https://cn.selectdb.com/blog/1741)
- **AI practical system**: [Doris & SelectDB for AI: building a non-structured-data intelligent-analysis system from scratch](https://bbs.huaweicloud.com/blogs/474525)
- **2026 Roadmap**: [Apache Doris 2026 Roadmap: how to choose data infrastructure in the AI era](https://blog.51cto.com/u_14767244/14823334)

## Examples & Open-source Projects

> Runnable open-source projects, demos and production practices built on Doris (RAG, intelligent analytics, LLM integration, etc.).

### RAG & Vector Store

| Project | Stars | Description |
| --- | --- | --- |
| [freemandealer/apache-doris-rag](https://github.com/freemandealer/apache-doris-rag) | ★16 | A RAG system built on Apache Doris hybrid search capabilities (Jupyter notebook) |
| [LangChain — Apache Doris vector store](https://python.langchain.com/docs/integrations/vectorstores/apache_doris) | — | Official LangChain integration using Doris as a vector store for RAG | 

**Quick Start (LangChain + Doris as vector store)**

```python
from langchain_community.vectorstores.apache_doris import ApacheDoris, ApacheDorisSettings

settings = ApacheDorisSettings()
settings.host = "localhost"; settings.port = 9030
settings.username = "root"; settings.database = "langchain"

docsearch = ApacheDoris.from_documents(split_docs, embeddings, config=settings)
qa = RetrievalQA.from_chain_type(llm=llm, retriever=docsearch.as_retriever())
```

### Intelligent Analysis & Agents

| Project | Stars | Description |
| --- | --- | --- |
| [DataAgent](https://github.com/3600818203/DataAgent) | ★60 | Multi-agent intelligent data-analysis system (data-warehouse oriented) |
| [MaloneTalk/DataAgent](https://github.com/MaloneTalk/DataAgent) | 0 | LLM-powered natural-language data agent — JDBC multi-datasource, semantic layer (supports analytical DBs incl. Doris) |

<!--
  More welcome:
  - Recommendation / risk-control sample systems
  - Non-structured data processing pipelines
  - LLM SQL-function demos
-->

## Community & Events

- [Apache Doris website](https://doris.apache.org/)
- [SelectDB](https://www.selectdb.com/)
- [Doris on GitHub](https://github.com/apache/doris)
- AI sessions at ApacheCon / Doris Summit

---

## Contributing

<!-- Please open a PR! Include contribution guidelines, classification standards, version requirements. -->

Open a PR! Please follow these guidelines:

- Annotate each project with **Stars**, a **one-line description**, and the **supported Doris version** (when applicable).
- Attach an actual runnable verification (SQL / command) for newly added content.
- Follow the [awesome list guidelines](https://github.com/sindresorhus/awesome).

## License

[MIT](./LICENSE)