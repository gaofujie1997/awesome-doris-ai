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

> Doris provides built-in AI functions that call external LLMs directly inside SQL — no data export needed. Run text classification, extraction, sentiment analysis, grammar fixing, content generation, masking, similarity, summarization, translation and cross-row aggregation right in the database.
>
> Official docs: [AI Function Overview](https://doris.apache.org/docs/dev/ai/ai-function-overview)

**Pick a function by use-case**

| What you want to do | Function | Returns |
| --- | --- | --- |
| Pick the best match from given labels | [`AI_CLASSIFY`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-classify) | a label string |
| Extract info from text by tags | [`AI_EXTRACT`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-extract) | extracted content per tag |
| Check if text meets a semantic condition | [`AI_FILTER`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-filter) | `BOOLEAN` |
| Fix grammar / spelling errors | [`AI_FIXGRAMMAR`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-fixgrammar) | corrected text |
| Generate new text from input | [`AI_GENERATE`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-generate) | generated text |
| Mask sensitive info | [`AI_MASK`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-mask) | text with `[MASKED]` |
| Analyze sentiment | [`AI_SENTIMENT`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-sentiment) | `positive`/`negative`/`neutral`/`mixed` |
| Semantic similarity of two texts | [`AI_SIMILARITY`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-similarity) | 0–10 float, higher = more similar |
| Summarize a text | [`AI_SUMMARIZE`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-summarize) | summary text |
| Translate to a target language | [`AI_TRANSLATE`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/ai-functions/ai-translate) | translated text |
| Cross-row aggregation over multiple rows | [`AI_AGG`](https://doris.apache.org/docs/dev/sql-manual/sql-functions/aggregate-functions/ai-agg) | aggregated text |

### Connect an LLM: configure an AI Resource

Doris centralizes LLM access through the [Resource](https://doris.apache.org/docs/dev/sql-manual/sql-statements/cluster-management/compute-management/CREATE-RESOURCE) mechanism — one place for provider, model, key and endpoint, keeping keys secure and access controlled.

**Resource parameters**

| Parameter | Required | Description |
| --- | --- | --- |
| `type` | yes | must be `ai` |
| `ai.provider_type` | yes | external provider |
| `ai.endpoint` | yes | AI API endpoint URL |
| `ai.model_name` | yes | model name |
| `ai.api_key` | yes (except `local`) | API key |
| `ai.temperature` | no | 0–1; default `-1` (unset) |
| `ai.max_tokens` | no | default `-1` (unset); Anthropic default `2048` |
| `ai.max_retries` | no | default `3` |
| `ai.retry_delay_second` | no | default `0` |

**Supported providers**: OpenAI · Anthropic · Gemini · DeepSeek · Local · MoonShot · MiniMax · Zhipu · Qwen · Baichuan. If your provider follows OpenAI/Anthropic/Gemini API format, you can pick the matching `ai.provider_type`.

### Quick Start

**Step 1 — create an AI Resource** (OpenAI and DeepSeek examples)

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

**Step 2 — set a default resource (optional)**

```sql
SET default_ai_resource = 'ai_resource_name';
```

**Step 3 — call AI functions in SQL**

*Example A — semantic relevance scoring: return the top 10 docs most relevant to "Apache Doris"*

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

*Example B — candidate / job semantic matching with `AI_FILTER`*

```sql
SELECT c.candidate_id, c.name, j.job_id, j.title
FROM candidate_profiles AS c
JOIN job_requirements AS j
WHERE AI_FILTER(CONCAT(
  'Does the following candidate self-introduction match the job description?',
  'Job: ', j.jd_text, ' Candidate: ', c.self_intro
));
```

### Design

- **Resource `<resource_name>`** must be a string constant.
- **`system_prompt`** differs per function; `user_prompt` carries only the input parameters.
- **Request body**: unset optional params (e.g. `ai.temperature`, `ai.max_tokens`) are omitted (except Anthropic, which requires `max_tokens`, default 2048).
- **Timeout**: per-request timeout equals the remaining query time (controlled by the `query_timeout` session variable); extend it if calls time out.
- **Provider abstraction**: Doris implements request construction, auth and response parsing per provider, so you only declare the provider type.

## Tutorials & Blogs

> Curated from the official Doris blog, the community, Chinese tech platforms (Zhihu, Tencent Cloud, Aliyun, Huawei Cloud, CSDN, modb.pro) and English sources — beyond the official forum.

### Overview & Roadmap

- **AI integration overview**: [Doris 4.x AI: full-text & vector search and intelligent analysis in one system](https://cloud.tencent.cn/developer/article/2624355) (Tencent Cloud)
- **2026 Roadmap**: [Apache Doris 2026 Roadmap: how to choose data infrastructure in the AI era](https://blog.51cto.com/u_14767244/14823334)
- **AI adoption at scale**: [ByteDance × Apache Doris: exploring a one-stop AI data engine](https://zhuanlan.zhihu.com/p/1989553954347049439) (Zhihu) / [ByteDance: Apache Doris + AI one-stop fusion data engine in practice](https://selectdb.com/blog/1539) (SelectDB)
- **AI capabilities deep-dive**: [Apache Doris 4.0 AI capabilities (2): AI functions designed for enterprise](https://www.modb.pro/db/1964890236467556352) (modb.pro)

### Vector Search & RAG

- **Vector search practice**: [97% recall, 900 QPS: production-grade vector search in Apache Doris 4.1](https://selectdb.com/blog/1698)
- **RAG from basic to knowledge-graph**: [Doris & SelectDB for AI: from basic RAG to knowledge-graph-enhanced implementation](https://cn.selectdb.com/blog/1682)
- **Production multimodal retrieval**: [Kuaishou: billion-scale multimodal retrieval on Apache Doris](https://cloud.tencent.cn/developer/article/2747664) (Tencent Cloud)
- **Lakehouse vector search**: [Kuaishou: Doris querying Paimon indexes — lakehouse vector retrieval built together](https://developer.aliyun.com/article/1762678) (Alibaba Cloud)
- **Official practical guide**: [Vector Index Practical Guide](https://doris.apache.org/docs/4.x/table-design/index/vector-index/practical-guide)

### AI Agent / MCP / Dify Integration

- **MCP + Dify**: [Connect AI Agents and real-time data warehouse: Apache Doris MCP Server + Dify in practice](https://cloud.tencent.com.cn/developer/article/2698145) (Tencent Cloud)
- **Zero-to-master**: [Dify + DeepSeek + Doris MCP Server: second-level SQL calls](https://cloud.tencent.com.cn/developer/article/2558425) (Tencent Cloud)
- **Data quality via MCP**: [Doris MCP + Dify make AI+ data-quality governance simple](https://cloud.tencent.com.cn/developer/article/2553599) (Tencent Cloud)
- **Agent operations**: [DB operations for AI agents: SelectDB CLI and Skills architecture in practice](https://cn.selectdb.com/blog/1741)

### AI SQL Functions & Practice

- **DeepSeek / ChatGPT / Claude in SQL**: [AI + data analytics: Doris lets SQL call DeepSeek, ChatGPT, Claude directly](https://cloud.tencent.cn/developer/article/2564490) (Tencent Cloud)
- **AI practical system**: [Doris & SelectDB for AI: build a non-structured-data intelligent-analysis system from scratch](https://cloud.tencent.com.cn/developer/article/2632290) (Tencent Cloud)
- **Official LLM SQL functions**: [LLM SQL Functions](https://doris.apache.org/docs/dev/key-features/llm-sql-functions)

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