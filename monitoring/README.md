# Monitoring Configuration

This directory contains the monitoring configuration for the Clustr project.

## Structure

```
monitoring/
├── prometheus/
│   ├── prometheus.yml          # Main Prometheus configuration
│   └── alerts/
│       └── clustr.yml  # Alert rules
└── grafana/
    └── provisioning/
        ├── datasources/
        │   └── prometheus.yml      # Prometheus datasource config
        └── dashboards/
            ├── default.yml         # Dashboard provider config
            └── clustr-overview.json  # Main dashboard
```

## Quick Start

The monitoring stack is integrated into the main docker-compose setup:

```bash
cd backend
docker compose up -d
```

Access the services:
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3000 (admin/admin)
- Metrics: http://localhost:8000/metrics

## Configuration Files

### Prometheus

- **prometheus.yml**: Scrape configuration for API server metrics
- **alerts/clustr.yml**: Alert rules for:
  - High error rates (API and crawler)
  - Slow queries
  - Database errors
  - Circuit breaker trips
  - Crawl job issues

### Grafana

- **datasources/prometheus.yml**: Auto-configures Prometheus as datasource
- **dashboards/default.yml**: Dashboard provider configuration
- **dashboards/clustr-overview.json**: Main system dashboard with:
  - KPI metrics (nodes, links, communities, jobs)
  - Graph growth charts
  - Crawl job status and throughput
  - API performance and error rates
  - Database operation metrics

## Customization

### Adding New Metrics

1. Define metrics in `backend/internal/metrics/metrics.go`
2. Instrument code to record metrics
3. Metrics automatically appear at `/metrics` endpoint

### Adding New Alerts

Edit `prometheus/alerts/clustr.yml`:

```yaml
- alert: MyNewAlert
  expr: my_metric > threshold
  for: 5m
  labels:
    severity: warning
  annotations:
    summary: "Alert description"
    description: "Detailed description with {{ $value }}"
```

### Creating New Dashboards

1. Create/export dashboard in Grafana UI
2. Save JSON to `grafana/provisioning/dashboards/`
3. Restart Grafana or wait for auto-reload

## Documentation

See [docs/monitoring.md](../docs/monitoring.md) for complete monitoring guide including:
- Full metrics reference
- PromQL query examples
- Alert configuration
- Troubleshooting
- Performance tuning
- Data export options
