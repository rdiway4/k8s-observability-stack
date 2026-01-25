# Setup Guide

This guide walks you through deploying the observability stack to your Kubernetes cluster.

## Prerequisites

- Kubernetes 1.25+
- Helm 3.10+
- `kubectl` configured for your cluster
- Sufficient cluster resources (see sizing guide below)

## Quick Start

### 1. Add Helm repositories

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add open-telemetry https://open-telemetry.github.io/opentelemetry-helm-charts
helm repo update
```

### 2. Create namespace and secrets

```bash
# Create namespace
kubectl create namespace monitoring

# Create secrets for alerting integrations
kubectl create secret generic alertmanager-secrets \
  --namespace monitoring \
  --from-literal=pagerduty-service-key=YOUR_PAGERDUTY_KEY \
  --from-literal=slack-webhook-url=YOUR_SLACK_WEBHOOK
```

### 3. Install the stack

```bash
# For development
helm install observability ./charts/observability-stack \
  --namespace monitoring \
  -f values/development.yaml

# For production
helm install observability ./charts/observability-stack \
  --namespace monitoring \
  -f values/production.yaml \
  --set kube-prometheus-stack.grafana.adminPassword=YOUR_SECURE_PASSWORD
```

### 4. Verify installation

```bash
# Check all pods are running
kubectl get pods -n monitoring

# Check services
kubectl get svc -n monitoring
```

### 5. Access Grafana

```bash
# Port forward to Grafana
kubectl port-forward svc/observability-grafana 3000:80 -n monitoring

# Open http://localhost:3000
# Default credentials: admin / prom-operator (or your custom password)
```

## Sizing Guide

### Development / Testing

- **Prometheus**: 1 replica, 10Gi storage, 512Mi-2Gi memory
- **Grafana**: 1 replica, no persistence
- **AlertManager**: 1 replica
- **Total**: ~4Gi memory, 15Gi storage

### Production (Small: up to 50 nodes)

- **Prometheus**: 2 replicas, 100Gi storage each, 4-8Gi memory
- **Grafana**: 2 replicas, 20Gi storage
- **AlertManager**: 3 replicas, 5Gi storage
- **Total**: ~20Gi memory, 250Gi storage

### Production (Large: 50-200 nodes)

- **Prometheus**: 2 replicas, 500Gi storage each, 16-32Gi memory
- **Grafana**: 2 replicas with external database
- **Consider**: Thanos or Cortex for long-term storage

## Upgrading

```bash
# Update Helm repos
helm repo update

# Upgrade the release
helm upgrade observability ./charts/observability-stack \
  --namespace monitoring \
  -f values/production.yaml
```

## Uninstalling

```bash
# Uninstall the release
helm uninstall observability -n monitoring

# Optionally delete PVCs (WARNING: this deletes all data)
kubectl delete pvc -n monitoring -l app.kubernetes.io/name=prometheus
kubectl delete pvc -n monitoring -l app.kubernetes.io/name=grafana
kubectl delete pvc -n monitoring -l app.kubernetes.io/name=alertmanager
```

## Troubleshooting

### Prometheus not scraping targets

1. Check ServiceMonitor/PodMonitor exists
2. Verify labels match Prometheus selector
3. Check target's metrics endpoint is accessible

```bash
kubectl get servicemonitor -A
kubectl logs -n monitoring prometheus-observability-prometheus-0 -c prometheus
```

### Grafana dashboards not loading

1. Check dashboard ConfigMaps exist
2. Verify sidecar is running
3. Check Grafana logs

```bash
kubectl logs -n monitoring -l app.kubernetes.io/name=grafana -c grafana-sc-dashboard
```

### AlertManager not sending alerts

1. Verify alertmanager config
2. Check secrets are mounted
3. Test with amtool

```bash
kubectl exec -it -n monitoring alertmanager-observability-alertmanager-0 -- amtool config show
```
