# k8s-observability-stack

A production-ready Kubernetes observability stack with Prometheus, Grafana, AlertManager, and OpenTelemetry. Designed for platform teams who need real SRE capabilities out of the box.

## 🎯 Features

- **Prometheus** — Metrics collection with pre-configured scrape configs
- **Grafana** — Dashboards for SLOs, error budgets, and infrastructure health
- **AlertManager** — Routing to PagerDuty, Slack, or email with severity-based escalation
- **OpenTelemetry Collector** — Unified telemetry pipeline for metrics, traces, and logs
- **Pre-built SRE Dashboards** — Golden signals, USE method, RED method
- **Multi-tenant ready** — Namespace-based isolation with RBAC

## 📋 Prerequisites

- Kubernetes 1.25+
- Helm 3.10+
- kubectl configured for your cluster

## 🚀 Quick Start

```bash
# Add Helm repos
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Install the stack
helm install observability ./charts/observability-stack \
  --namespace monitoring \
  --create-namespace \
  -f values/production.yaml
```

## 📁 Repository Structure

```
.
├── charts/
│   └── observability-stack/      # Main Helm chart
├── dashboards/                   # Grafana dashboard JSON files
│   ├── slo-overview.json
│   ├── golden-signals.json
│   └── kubernetes-cluster.json
├── alerts/                       # PrometheusRule definitions
│   ├── slo-alerts.yaml
│   ├── kubernetes-alerts.yaml
│   └── application-alerts.yaml
├── otel/                         # OpenTelemetry configs
│   └── collector-config.yaml
├── values/                       # Environment-specific values
│   ├── development.yaml
│   ├── staging.yaml
│   └── production.yaml
└── docs/
    ├── SETUP.md
    ├── ALERTING.md
    └── DASHBOARDS.md
```

## 📊 Included Dashboards

| Dashboard | Description |
|-----------|-------------|
| SLO Overview | Service level objectives with error budget burn rate |
| Golden Signals | Latency, traffic, errors, saturation for all services |
| Kubernetes Cluster | Node health, pod status, resource utilization |
| Cost Attribution | Resource usage by namespace/team for FinOps |

## 🔔 Alerting

Alerts are organized by severity:

- **Critical** → PagerDuty (immediate page)
- **Warning** → Slack #alerts channel
- **Info** → Slack #alerts-low-priority

Configure your receivers in `values/production.yaml`:

```yaml
alertmanager:
  config:
    receivers:
      - name: pagerduty-critical
        pagerduty_configs:
          - service_key: ${PAGERDUTY_SERVICE_KEY}
      - name: slack-warnings
        slack_configs:
          - api_url: ${SLACK_WEBHOOK_URL}
            channel: '#alerts'
```

## 🔧 Configuration

### Enable OpenTelemetry

```yaml
opentelemetry:
  enabled: true
  receivers:
    otlp:
      protocols:
        grpc:
          endpoint: 0.0.0.0:4317
        http:
          endpoint: 0.0.0.0:4318
```

### SLO Definitions

Define your SLOs in `values/production.yaml`:

```yaml
slos:
  - name: api-availability
    target: 99.9
    service: api-gateway
    metric: http_requests_total
    errorMetric: http_requests_total{status=~"5.."}
    
  - name: api-latency-p99
    target: 99
    service: api-gateway
    metric: http_request_duration_seconds_bucket
    threshold: 0.5  # 500ms
```

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Your Applications                        │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐        │
│  │ Service │  │ Service │  │ Service │  │ Service │        │
│  └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘        │
└───────┼────────────┼────────────┼────────────┼──────────────┘
        │            │            │            │
        ▼            ▼            ▼            ▼
┌─────────────────────────────────────────────────────────────┐
│              OpenTelemetry Collector                         │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                   │
│  │ Metrics  │  │  Traces  │  │   Logs   │                   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                   │
└───────┼─────────────┼─────────────┼─────────────────────────┘
        │             │             │
        ▼             ▼             ▼
┌───────────┐  ┌───────────┐  ┌───────────┐
│Prometheus │  │  Jaeger   │  │   Loki    │
│           │  │ (optional)│  │ (optional)│
└─────┬─────┘  └───────────┘  └───────────┘
      │
      ▼
┌───────────┐      ┌───────────────┐
│  Grafana  │ ◄────│ AlertManager  │──► PagerDuty/Slack
└───────────┘      └───────────────┘
```

## 📈 Metrics Collected

- Kubernetes metrics (kube-state-metrics, node-exporter)
- Application metrics (via ServiceMonitor CRDs)
- Custom business metrics (via PushGateway or OTLP)

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Submit a pull request

## 📄 License

MIT License - see [LICENSE](LICENSE)

---

Built with ☕ by [Dipo Oginni](https://dipops.com) | Platform Engineer & SRE
