# Grafana Dashboards Guide

> A curated reference of the most impactful Grafana dashboards used across SRE, DevOps, Backend, Data, Database, and Security teams at leading tech companies — with real use cases, PromQL queries, and ready-to-deploy configs.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Dashboards](https://img.shields.io/badge/Dashboards-24%2B-orange)](dashboards/)
[![Teams](https://img.shields.io/badge/Teams-6-blue)](docs/)

---

## 🔍 What's Inside

| Team | Dashboards | Use Cases |
|------|-----------|-----------|
| SRE / Platform Eng | Node Exporter, k8s Cluster, USE Method, Alertmanager | Host health, capacity planning, incident triage |
| DevOps / CI-CD | GitHub Actions, ArgoCD, DORA Metrics, Terraform | CI cost, drift detection, DORA benchmarking |
| Backend / API | RED Method, NGINX, Jaeger/Tempo, gRPC | SLO tracking, latency regressions, error budgets |
| Data / ML | Kafka, MLflow, Spark Streaming, NVIDIA GPU | Consumer lag, training monitoring, GPU utilization |
| Database | PostgreSQL, Redis, Elasticsearch, ClickHouse | Query degradation, cache efficiency, replication lag |
| Security / SecOps | Falco, Auth0, WAF/DDoS, Cert Expiry | Runtime threats, brute force, cert rotation |

---

## 📁 Repository Structure

```
grafana-dashboards-guide/
├── index.html                    # Static website (open in browser)
├── README.md
├── LICENSE
├── dashboards/
│   ├── sre/
│   │   ├── node-exporter-full.json
│   │   ├── kubernetes-cluster.json
│   │   ├── use-method-system.json
│   │   └── alertmanager-overview.json
│   ├── devops/
│   │   ├── github-actions.json
│   │   ├── argocd-status.json
│   │   ├── dora-metrics.json
│   │   └── terraform-runs.json
│   ├── backend/
│   │   ├── red-method.json
│   │   ├── nginx-ingress.json
│   │   ├── jaeger-tempo.json
│   │   └── grpc-server.json
│   ├── data-ml/
│   │   ├── kafka-overview.json
│   │   ├── mlflow-experiments.json
│   │   ├── spark-streaming.json
│   │   └── nvidia-gpu-cluster.json
│   ├── database/
│   │   ├── postgresql-overview.json
│   │   ├── redis-exporter.json
│   │   ├── elasticsearch-cluster.json
│   │   └── clickhouse-performance.json
│   └── security/
│       ├── falco-runtime.json
│       ├── auth0-login-metrics.json
│       ├── waf-ddos-overview.json
│       └── cert-expiry-monitor.json
├── queries/
│   ├── sre.promql
│   ├── backend.promql
│   ├── database.promql
│   ├── data-ml.promql
│   └── security.promql
├── exporters/
│   ├── node-exporter/
│   │   └── daemonset.yaml
│   ├── kube-state-metrics/
│   │   └── deployment.yaml
│   ├── postgres-exporter/
│   │   └── deployment.yaml
│   ├── redis-exporter/
│   │   └── deployment.yaml
│   └── dcgm-exporter/
│       └── daemonset.yaml
├── helm/
│   ├── grafana-values.yaml
│   └── prometheus-values.yaml
└── docs/
    ├── setup-guide.md
    ├── alert-rules.md
    └── team-onboarding.md
```

---

## 🚀 Quick Start

### 1. Install Grafana via Helm

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install grafana grafana/grafana -f helm/grafana-values.yaml
```

### 2. Install Prometheus

```bash
helm install prometheus prometheus-community/kube-prometheus-stack \
  -f helm/prometheus-values.yaml
```

### 3. Deploy Exporters

```bash
kubectl apply -f exporters/node-exporter/
kubectl apply -f exporters/kube-state-metrics/
kubectl apply -f exporters/postgres-exporter/
```

### 4. Import Dashboards

Import via Grafana UI (Dashboards → Import) using the dashboard IDs below, or use the API:

```bash
curl -X POST http://admin:admin@localhost:3000/api/dashboards/import \
  -H 'Content-Type: application/json' \
  -d @dashboards/sre/node-exporter-full.json
```

---

## 📊 Dashboard IDs (Grafana Community)

Import these directly from [grafana.com/grafana/dashboards](https://grafana.com/grafana/dashboards):

| Dashboard | ID | Team |
|-----------|-----|------|
| Node Exporter Full | `1860` | SRE |
| Kubernetes Cluster Overview | `7249` | SRE |
| USE Method: System | `405` | SRE |
| Alertmanager Overview | `9578` | SRE |
| ArgoCD Application Status | `14584` | DevOps |
| NGINX Ingress Controller | `9614` | Backend |
| gRPC Server Metrics | `9733` | Backend |
| Apache Kafka Overview | `7589` | Data |
| Spark Structured Streaming | `12477` | Data |
| NVIDIA DCGM GPU Metrics | `12239` | Data / ML |
| PostgreSQL Overview | `9628` | Database |
| Redis Exporter | `763` | Database |
| Elasticsearch Cluster | `2322` | Database |
| ClickHouse Performance | `14192` | Database |
| Certificate Expiry Monitor | `13230` | Security |

---

## ⚡ Essential PromQL Queries

### CPU Utilization

```promql
100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)
```

### API Error Rate (RED Method)

```promql
sum(rate(http_requests_total{status=~"5.."}[1m])) /
sum(rate(http_requests_total[1m])) * 100
```

### p99 Latency

```promql
histogram_quantile(0.99, sum by (service, le) (rate(http_duration_bucket[5m])))
```

### Kafka Consumer Lag

```promql
sum by (consumergroup, topic) (kafka_consumergroup_lag{topic!="__consumer_offsets"})
```

### PostgreSQL Cache Hit Ratio

```promql
sum(pg_stat_bgwriter_buffers_clean) /
(sum(pg_stat_bgwriter_buffers_clean) + sum(pg_stat_bgwriter_buffers_backend)) * 100
```

### Redis Hit Rate

```promql
rate(redis_keyspace_hits_total[5m]) /
(rate(redis_keyspace_hits_total[5m]) + rate(redis_keyspace_misses_total[5m])) * 100
```

### Pod OOMKilled

```promql
increase(kube_pod_container_status_restarts_total{reason="OOMKilled"}[15m])
```

### GPU Utilization

```promql
avg by (gpu, instance) (DCGM_FI_DEV_GPU_UTIL{job="dcgm-exporter"})
```

---

## 📖 Documentation

- [Setup Guide](docs/setup-guide.md) — Full Grafana + Prometheus + exporters setup
- [Alert Rules](docs/alert-rules.md) — Pre-built alerting rules for each team
- [Team Onboarding](docs/team-onboarding.md) — Which dashboards to set up per team

---

## 🤝 Contributing

PRs welcome! To add a dashboard:

1. Export the dashboard JSON from Grafana (Share → Export → Save to file)
2. Add to the relevant `dashboards/<team>/` folder
3. Update this README's dashboard table
4. Open a PR with a description of what the dashboard monitors

---

## 📄 License

MIT — see [LICENSE](LICENSE)

---

> Not affiliated with Grafana Labs. Dashboard IDs reference community dashboards on grafana.com.
