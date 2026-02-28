# Getting Started

This guide helps you run your first Knowledge2 workflow quickly.

## 1. Choose your endpoint

- Production: `https://api.knowledge2.ai`
- Staging: `https://api-staging.knowledge2.ai`
- Dev: `https://api-dev.knowledge2.ai`

## 2. Install the Python SDK

```bash
pip install knowledge2
```

## 3. Create a client

```python
from sdk import Knowledge2

client = Knowledge2(api_key="k2_...")
```

## 4. Ingest, index, and search

```python
project = client.create_project("My Project")
corpus = client.create_corpus(project["id"], "My Corpus")

client.upload_document(
    corpus["id"],
    raw_text="Knowledge2 lets you build hybrid retrieval pipelines.",
)
client.build_indexes(corpus["id"])

results = client.search(corpus["id"], "hybrid retrieval", top_k=5)
print(results)
```

## 5. Next guides

- [Python SDK Guide](sdk-python.md)
- [Framework Integrations](framework-integrations.md)
- [OpenAPI Reference](../reference/openapi.md)
