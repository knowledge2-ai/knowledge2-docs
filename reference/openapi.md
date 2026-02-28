# OpenAPI

The API is defined by FastAPI and available at runtime:

- `GET /openapi.json`
- `GET /docs` (Swagger UI)

To export the spec to a file, run:

```bash
python scripts/export_openapi.py > docs/openapi.json
```
