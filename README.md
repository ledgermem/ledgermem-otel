# `ledgermem-otel`

OpenTelemetry collector config + Grafana dashboards + Prometheus alert rules for **LedgerMem self-hosters**.

> Versioned configs, importable by ID. Not a service — just YAML and JSON you point your tooling at.

## What's here

```
collector/
  otel-collector-config.yaml      # recommended pipeline (OTLP → Prometheus + Loki + Tempo)
dashboards/
  ledgermem-api-health.json       # API latency, error rate, throughput
  ledgermem-retrieval.json        # search recall, strategy breakdown, vector store latency
  ledgermem-cost.json             # tokens / requests / $ per workspace
prometheus/
  alerts.yml                      # SLO + saturation alert rules
loki/
  pipeline.yaml                   # log parsing for the API service
tempo/
  conventions.md                  # span naming + tag conventions for LedgerMem services
```

## Use the dashboards

### Grafana.com (when published)

```
Import → enter dashboard ID:
  ledgermem-api-health      —  ID: pending
  ledgermem-retrieval       —  ID: pending
  ledgermem-cost            —  ID: pending
```

### Manual

```
Grafana → Dashboards → New → Import → Upload JSON file → pick from dashboards/
```

## Use the collector config

```bash
docker run -p 4317:4317 -p 4318:4318 \
  -v $(pwd)/collector/otel-collector-config.yaml:/etc/otel-collector-config.yaml \
  otel/opentelemetry-collector-contrib:latest \
  --config=/etc/otel-collector-config.yaml
```

Then point your LedgerMem services' `OTEL_EXPORTER_OTLP_ENDPOINT` at this collector.

## Use the alert rules

```yaml
# prometheus.yml
rule_files:
  - /etc/prometheus/alerts.yml
```

## Span conventions

LedgerMem services emit spans following the conventions in [`tempo/conventions.md`](tempo/conventions.md). If you're building a service that talks to LedgerMem, follow the same shape so traces stitch correctly.

## License

Apache 2.0 (so it can be embedded in commercial Grafana installations without conflict).
