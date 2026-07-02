# k8s-observability-stack

An opinionated, production-minded observability layer for Kubernetes:
Prometheus Operator, Grafana with dashboards as code, Alertmanager, and
SLO burn-rate alerting that pages on real user pain instead of noisy
thresholds.

I built this to package the monitoring patterns I rely on in production into
something reusable: recording rules that keep alerts cheap, multi-window
burn-rate alerts, and a starter SLO dashboard.

## What is inside

| Path | Purpose |
|------|---------|
| [`kube-prometheus-stack/values.yaml`](kube-prometheus-stack/values.yaml) | Opinionated Helm values (retention, resources, cross-namespace rule discovery, locked-down Grafana) |
| [`recording-rules/`](recording-rules) | Precomputed error ratios over 5m / 30m / 1h / 6h windows |
| [`alerts/slo-burn-rate.rules.yaml`](alerts/slo-burn-rate.rules.yaml) | Multi-window, multi-burn-rate availability alerts |
| [`alerts/infra.rules.yaml`](alerts/infra.rules.yaml) | Node, pod, and volume health alerts |
| [`dashboards/`](dashboards) | Grafana "Service SLO Overview" dashboard as JSON |
| [`docs/slo-methodology.md`](docs/slo-methodology.md) | The burn-rate approach and how to tune it |

## Quick start

```bash
# 1. Install the operator + Prometheus + Grafana + Alertmanager
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install obs prometheus-community/kube-prometheus-stack \
  -f kube-prometheus-stack/values.yaml

# 2. Apply the recording rules and alerts (Prometheus Operator picks them up)
kubectl apply -f recording-rules/
kubectl apply -f alerts/

# 3. Import dashboards/service-slo-dashboard.json into Grafana
```

## Design choices

- **Burn-rate SLOs, not static thresholds.** Alerts map to error-budget spend,
  so a page means users are actually hurting. See
  [docs/slo-methodology.md](docs/slo-methodology.md).
- **Recording rules keep alerts cheap.** Error ratios are precomputed once and
  reused by every alert window.
- **Dashboards as code.** The Grafana dashboard lives in Git and is imported,
  not hand-built and lost.
- **No secrets in the repo.** Grafana admin password and Alertmanager receiver
  credentials are injected at deploy time, never committed.

## Requirements

- Kubernetes 1.26+
- Helm 3
- An app exposing `http_requests_total{code=...}` (adjust selectors otherwise)

## CI

`.github/workflows/lint.yml` validates every YAML file and runs `promtool` over
the rule files on each push and pull request.

## License

MIT. See [LICENSE](LICENSE).
