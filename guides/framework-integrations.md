# SDK Framework Integrations

This guide covers first-class SDK adapters for LangChain and LlamaIndex.

## Installation

From repo root:

```bash
pip install -e ".[langchain]"
pip install -e ".[llamaindex]"
# or both
pip install -e ".[integrations]"
```

## Environment setup

Required:

- `K2_API_KEY`
- `K2_CORPUS_ID`

Optional:

- `K2_BASE_URL` (default: `https://api.knowledge2.ai`)
- `K2_ADMIN_TOKEN` (bootstrap-only; self-hosted/private deployments and internal automation)

### SaaS vs self-hosted authentication

- **SaaS (hosted Knowledge2):** use a **Console-created API key** (`K2_API_KEY`). Most applications should not need an admin token. Create your Project/Corpus in the Console, then use the API key + `K2_CORPUS_ID` in LangChain/LlamaIndex.
- **Private/self-hosted deployments:** an **admin token** (`K2_ADMIN_TOKEN` / `X-Admin-Token`) exists to **bootstrap** the first organization and/or mint initial org-wide API keys. Do not embed admin tokens in client apps.

Notes:

- “Org-wide” in this repo means an API key with unrestricted org scopes (e.g. `{"all": true}`), which is **still an API key** (sent as `X-API-Key`) and is distinct from the admin token (`X-Admin-Token`).

Local stack defaults:

- `K2_BASE_URL=http://api:8000`
- `K2_API_KEY=local-admin-token`

## LangChain adapter

`K2LangChainRetriever` returns standard LangChain `Document` objects.

Supported options:

- `top_k`
- `filters`
- `hybrid`
- `graph_rag`
- `rerank`

Each `Document.metadata` includes:

- `chunk_id`, `score`, `raw_score`
- `offset_start`, `offset_end`, `page_start`, `page_end`
- merged K2 chunk metadata

## LlamaIndex adapters

### `K2LlamaIndexRetriever`

- Implements `BaseRetriever`
- Returns `NodeWithScore`
- Supports text query retrieval over K2 search

### `K2LlamaIndexVectorStore`

Doc-centric mapping to K2 APIs:

- `add(nodes)` -> K2 document ingest
- `query(VectorStoreQuery)` -> K2 search
- `delete(ref_doc_id)` -> K2 document delete

Not supported:

- embedding-only lookup by id (`get(text_id)`) because K2 does not expose raw vector lookup endpoints

## Ingestion paths (what to use)

K2 supports multiple ingestion styles. Pick the one that matches your app:

- Upload a **directory of files** (recommended for most users):
  - Use `Knowledge2.upload_files_batch(corpus_id, files=[(filename, bytes), ...])`, then `build_indexes(...)`.
- Upload a **single file**:
  - Use `Knowledge2.upload_document(corpus_id, file_path=...)`, then `build_indexes(...)`.
- Ingest **raw text** (agent/tool workflows, demos, tests):
  - LangChain: `k2_ingest_text(raw_text=...)`
  - LlamaIndex tools: `k2_ingest_text(raw_text=...)`
- You already have **text/nodes in memory** and want to stay inside LlamaIndex APIs:
  - Use `K2LlamaIndexVectorStore.add(nodes)` (doc-centric mapping to K2 ingestion).

Notes:

- Batch upload is typically faster for folders, but it does not provide the same per-file control as individual `upload_document(...)` calls.

## Metadata filter mapping

### LangChain

Pass K2-native dict filters directly (key -> scalar or list).

### LlamaIndex

`MetadataFilters` are converted to K2 dict filters.

Supported:

- `FilterCondition.AND`
- `FilterOperator.EQ`
- `FilterOperator.IN`

Unsupported conditions/operators raise `ValueError` with an explicit message.

## Tool helpers

### LangChain

`create_k2_langchain_tools(...)` returns:

- `k2_search`
- `k2_ingest_text`
- `k2_build_indexes`
- `k2_generate_answer` (grounded generation via `/search:generate`, requires backend LLM config)

### LlamaIndex

`create_k2_llamaindex_tools(...)` returns `FunctionTool` wrappers for the same operations.

## Examples

- `examples/09_langchain_k2_sdk`
- `examples/10_llamaindex_k2_sdk`

Both examples run against local or dev by switching `K2_BASE_URL`.
