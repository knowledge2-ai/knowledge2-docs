# Knowledge² Python SDK

This SDK wraps the public Knowledge2 API and is intended for Knowledge² customers who want a
fully programmatic end-to-end lifecycle (ingest → index → tune → deploy → hybrid search).

## Endpoints

- Production (default): `https://api.knowledge2.ai`
- Staging: `https://api-staging.knowledge2.ai`
- Dev: `https://api-dev.knowledge2.ai`

Website: `https://knowledge2.ai`  
Support: `contact@knowledge2.ai`

## Install (from this repo)

```bash
pip install -e .
```

## Authentication

- API key: set `api_key` when creating the client (uses `X-API-Key`).
- Console endpoints: set `bearer_token` to send `Authorization: Bearer <token>`.
- Admin endpoints: set `admin_token` to send `X-Admin-Token`.

### SaaS vs self-hosted

- **SaaS (hosted Knowledge2):** your application should use a **Console-created API key** (`X-API-Key`). You typically will not have (or need) an admin token.
- **Private/self-hosted deployments:** the **admin token** (`X-Admin-Token`) exists for **bootstrap/admin operations** (for example, creating the first org and minting an initial org-wide API key). Do not embed admin tokens in client apps.

Note: in this codebase, “org-wide” refers to an API key with unrestricted org scopes (for example `{"all": true}`), which is distinct from the admin token.

## Basic usage

```python
from sdk import Knowledge2

client = Knowledge2(api_key="<api-key>")

project = client.create_project("demo-project")
corpus = client.create_corpus(project["id"], "demo-corpus")

docs = [
    {
        "source_uri": "doc://overview",
        "raw_text": "Knowledge2 organizes knowledge into projects and corpora. Documents are chunked into passages that can be indexed for search. Stable source_uris let you update content without duplicates.",
        "metadata": {"topic": "overview", "product": "knowledge2"},
    },
    {
        "source_uri": "doc://ingestion",
        "raw_text": "Batch ingestion accepts documents with source_uri, raw_text, and optional metadata. Use idempotency keys to prevent duplicate ingest jobs. Upload in batches to stay within API limits.",
        "metadata": {"topic": "ingestion", "product": "knowledge2"},
    },
    {
        "source_uri": "doc://indexing",
        "raw_text": "Dense indexes capture semantic similarity while sparse indexes capture keyword matches. Building both enables hybrid retrieval. Rebuild indexes after large content updates.",
        "metadata": {"topic": "indexing", "product": "knowledge2"},
    },
    {
        "source_uri": "doc://hybrid-search",
        "raw_text": "Hybrid retrieval blends dense and sparse scores using RRF or weighted fusion. Adjust dense_weight and sparse_weight to balance semantics vs exact terms. You can request scores and provenance in the response.",
        "metadata": {"topic": "search", "product": "knowledge2"},
    },
    {
        "source_uri": "doc://tuning",
        "raw_text": "Tuning runs train a better embedding model from query-document pairs. Training data can be auto-built or uploaded as JSONL. Successful runs can be promoted to a deployable model.",
        "metadata": {"topic": "tuning", "product": "knowledge2"},
    },
    {
        "source_uri": "doc://deployments",
        "raw_text": "Deployments attach a tuned model to a corpus and optionally trigger reindexing. Track the reindex job until it succeeds. Once complete, searches use the tuned model.",
        "metadata": {"topic": "deployments", "product": "knowledge2"},
    },
    {
        "source_uri": "doc://evaluation",
        "raw_text": "Evaluation runs compute metrics like nDCG, recall, and MRR on labeled data. Compare baseline and tuned models before promotion. Keep eval sets representative of real queries.",
        "metadata": {"topic": "evaluation", "product": "knowledge2"},
    },
    {
        "source_uri": "doc://security",
        "raw_text": "API keys authenticate requests and can be rotated without downtime. Admin tokens are required for org bootstrap and should be stored securely. Audit logs and usage endpoints help with governance.",
        "metadata": {"topic": "security", "product": "knowledge2"},
    },
]
job = client.upload_documents_batch(corpus["id"], docs)
print("ingest job:", job["job_id"])
```

## End-to-end lifecycle example

The full, runnable example lives at `sdk/examples/e2e_lifecycle.py`. It covers:

- Project + corpus creation
- Batch document ingest and job polling
- Dense + sparse index build
- Hybrid search configuration
- Tuning run (auto build or uploaded JSONL)
- Promotion + deployment
- Optional eval run and metrics

Run it with:

```bash
export K2_BASE_URL=https://api.knowledge2.ai
export K2_API_KEY=<api-key>
python sdk/examples/e2e_lifecycle.py
```

Optional inputs:

```bash
export K2_PROJECT_ID=<project-id>
export K2_TUNING_JSONL=/path/to/training.jsonl
export K2_EVAL_JSONL=/path/to/eval.jsonl
```

## Framework integrations (LangChain + LlamaIndex)

Install extras from repo root:

```bash
pip install -e ".[langchain]"
pip install -e ".[llamaindex]"
```

LangChain retriever:

```python
from sdk.integrations.langchain import K2LangChainRetriever

retriever = K2LangChainRetriever(
    api_key="<api-key>",
    api_host="https://api.knowledge2.ai",
    corpus_id="<corpus-id>",
    top_k=5,
    filters={"topic": "search"},
    hybrid={"enabled": True, "fusion_mode": "rrf", "dense_weight": 0.7, "sparse_weight": 0.3},
)
docs = retriever.invoke("How does hybrid retrieval work?")
```

LlamaIndex retriever:

```python
from sdk.integrations.llamaindex import K2LlamaIndexRetriever

retriever = K2LlamaIndexRetriever(
    api_key="<api-key>",
    api_host="https://api.knowledge2.ai",
    corpus_id="<corpus-id>",
    top_k=5,
)
nodes = retriever.retrieve("How does hybrid retrieval work?")
```

See `docs/sdk_framework_integrations.md` for full adapter capabilities, filter mapping, and tool helpers.

## Hybrid retrieval notes

Hybrid retrieval is configured per request:

```python
results = client.search(
    corpus_id,
    "What is hybrid retrieval?",
    top_k=5,
    hybrid={
        "enabled": True,
        "fusion_mode": "rrf",
        "rrf_k": 60,
        "dense_weight": 0.6,
        "sparse_weight": 0.4,
    },
    graph_rag={
        "enabled": True,
        "seed_chunks": 5,
        "max_related_docs": 10,
        "timeout_ms": 250,
        # Optional staging analysis mode (no ranking impact):
        # "shadow_mode": True,
    },
    return_config={"include_text": True, "include_scores": True, "include_provenance": True},
)
```

You can also request index rebuilds that include graph artifact generation:

```python
client.build_indexes(corpus_id, dense=True, sparse=True, graph=True, wait=True)
```

For more retrieval tuning options, see `docs/retrieval_config.md`, `docs/graphrag.md`,
`docs/graphrag_eval.md`, and `docs/graphrag_rollout.md`.
