# Tempo / OTel span conventions for LedgerMem

These conventions apply to every service in the LedgerMem stack and to any third-party service that talks to LedgerMem and wants traces to stitch correctly.

## Span naming

| Operation | Span name |
| --- | --- |
| HTTP server | `<METHOD> <route>` (e.g. `POST /v1/memories`) |
| HTTP client to LedgerMem | `LedgerMem <METHOD> <route>` |
| DB query | `db.<verb> <table>` (e.g. `db.select memories`) |
| Vector store query | `vector.<verb> <collection>` |
| Embedding | `embed <model>` |
| LLM call | `llm <provider>.<model>` |

## Required attributes

Every span:

- `service.name` — kebab-case service name (`ledgermem-api`, `ledgermem-worker`)
- `service.namespace` — always `ledgermem`
- `deployment.environment` — `prod` / `staging` / `dev`

API request spans:

- `workspace.id`
- `actor.id` (if scoped)
- `http.route` (templated, not raw URL)
- `http.status_code`

Memory operations:

- `memory.operation` — `add` / `search` / `get` / `update` / `delete` / `list`
- `memory.id` (when applicable)
- `memory.strategy` — for searches: `vector` / `bm25` / `hybrid` / `graph` / ...

Retrieval spans:

- `retrieval.k` — top-k
- `retrieval.recall_at_k` — when measured
- `retrieval.fused` — boolean

LLM spans:

- `llm.provider` — `openai` / `anthropic` / `bedrock` / `local`
- `llm.model`
- `llm.input_tokens`
- `llm.output_tokens`
- `llm.cached_tokens` (when applicable)

## Forbidden attributes

- ❌ `api.key`, `Authorization`, raw API tokens of any kind
- ❌ Full memory content (use `memory.id` and `memory.size_bytes` instead)
- ❌ User PII unless explicitly hashed and the workspace has opted in
- ❌ Raw embedding vectors

## Trace context propagation

- Inbound: read W3C `traceparent` + `tracestate` from request headers
- Outbound (calls to LedgerMem API): inject `traceparent`
- Background jobs: link to the parent request span via span links, do not extend it
