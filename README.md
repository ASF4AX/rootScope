# Observability Stack

Docker Compose-based local observability stack for metrics, logs, and dashboards.

## Services and Roles
- `node-exporter`: host CPU/memory/disk/network metrics export
- `prometheus`: scrape and store metrics (`prometheus`, `node-exporter`)
- `fluent-bit`: tail app logs from `sample-logs/*.log` and send to Elasticsearch
- `elasticsearch`: store and search ingested logs
- `grafana`: visualize metrics/logs with provisioned datasources and dashboard

## What Gets Collected and Displayed
- Metrics:
  - source: `node-exporter` and Prometheus self-metrics
  - storage/query: Prometheus
  - visualization: Grafana (Prometheus datasource)
- Logs:
  - source: `sample-logs/app.log` (JSON lines)
  - pipeline: Fluent Bit -> Elasticsearch index `logs-local-sample-app-*`
  - visualization: Grafana Explore (Elasticsearch datasource)

## Required Setup
Create `.env` from `.env.sample` and set Grafana admin credentials:

```dotenv
GF_SECURITY_ADMIN_USER=<your-admin-user>
GF_SECURITY_ADMIN_PASSWORD=<your-strong-password>
```

Optional:

```dotenv
ES_HEAP_SIZE=512m
```

If Grafana is behind a reverse proxy subpath (for example `/grafana`), also set:

```dotenv
GRAFANA_ROOT_URL=https://<host>/grafana/
GRAFANA_SERVE_FROM_SUB_PATH=true
```

## Run and Verify
1. Start services.

```bash
docker compose up -d
```

2. Verify access.
- Grafana: http://localhost:3300
- Prometheus: http://localhost:9090

3. Check log visibility in Grafana.
Open Grafana and sign in with `.env` credentials.
Go to `Explore` -> datasource `Elasticsearch` -> query type `Logs`.
Run `Lucene Query: *` with a time range including sample timestamps.

Sample log timestamps in `sample-logs/app.log`:
- `2026-02-12T10:00:00.000Z`
- `2026-02-12T10:00:10.000Z`

## Log Ingestion
- Fluent Bit tails `sample-logs/*.log` via container path `/var/log/app/*.log`
- Parser expects JSON logs with `@timestamp` in `%Y-%m-%dT%H:%M:%S.%LZ` format
- Logs are written to Elasticsearch index pattern: `logs-local-sample-app-*`
- Tail state DB path: `/fluent-bit/state/tail.db` (persisted in `fluentbit_state` volume to reduce duplicate ingestion after restarts)

Required JSON keys:
- `@timestamp`
- `level`
- `message`

## Stop and Cleanup
```bash
docker compose down
```

Remove persisted data volumes:
```bash
docker compose down -v
```
