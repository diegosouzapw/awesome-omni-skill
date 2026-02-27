---
name: grafana-prometheus
version: 1.0.0
description: Observability and monitoring with Prometheus metrics and Grafana dashboards
author: Code Buddy
tags: grafana, prometheus, monitoring, observability, metrics, alerting, dashboards, devops
env:
  GRAFANA_URL: ""
  GRAFANA_API_TOKEN: ""
  PROMETHEUS_URL: ""
---

# Grafana + Prometheus Monitoring

Complete observability stack with Prometheus metrics collection, PromQL queries, and Grafana visualization dashboards.

## Direct Control (CLI / API / Scripting)

### Prometheus Configuration

```yaml
# prometheus.yml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
  external_labels:
    cluster: 'production'
    environment: 'prod'

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - alertmanager:9093

# Load rules
rule_files:
  - "alerts/*.yml"
  - "recording_rules/*.yml"

# Scrape configurations
scrape_configs:
  # Prometheus self-monitoring
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  # Node exporter (system metrics)
  - job_name: 'node'
    static_configs:
      - targets:
          - 'node1:9100'
          - 'node2:9100'
          - 'node3:9100'
        labels:
          env: 'production'

  # Kubernetes service discovery
  - job_name: 'kubernetes-pods'
    kubernetes_sd_configs:
      - role: pod
    relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__

  # Application metrics
  - job_name: 'app'
    static_configs:
      - targets:
          - 'app1:8080'
          - 'app2:8080'
    metrics_path: '/metrics'
    scrape_interval: 10s

  # Blackbox exporter (endpoint monitoring)
  - job_name: 'blackbox'
    metrics_path: /probe
    params:
      module: [http_2xx]
    static_configs:
      - targets:
          - https://example.com
          - https://api.example.com/health
    relabel_configs:
      - source_labels: [__address__]
        target_label: __param_target
      - source_labels: [__param_target]
        target_label: instance
      - target_label: __address__
        replacement: blackbox-exporter:9115
```

### PromQL Queries

```promql
# CPU Usage
100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)

# Memory Usage Percentage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk Space Available
node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} * 100

# HTTP Request Rate
rate(http_requests_total[5m])

# HTTP Request Rate by Status Code
sum by (status) (rate(http_requests_total[5m]))

# 95th Percentile Request Duration
histogram_quantile(0.95, sum by (le) (rate(http_request_duration_seconds_bucket[5m])))

# Error Rate
sum(rate(http_requests_total{status=~"5.."}[5m])) / sum(rate(http_requests_total[5m])) * 100

# Pod Restarts (Kubernetes)
sum by (namespace, pod) (kube_pod_container_status_restarts_total)

# Network Traffic
rate(node_network_receive_bytes_total[5m])
rate(node_network_transmit_bytes_total[5m])

# Active Alerts
sum by (alertname, severity) (ALERTS{alertstate="firing"})

# Query per second by database
sum by (database) (rate(mysql_global_status_queries[5m]))

# Container CPU Usage
sum by (pod_name) (rate(container_cpu_usage_seconds_total[5m]))

# Top 10 endpoints by request count
topk(10, sum by (endpoint) (rate(http_requests_total[1h])))

# SLA calculation (uptime percentage)
avg_over_time((up{job="api"}[30d])) * 100
```

### Prometheus HTTP API (cURL)

```bash
# Query instant value
curl -G http://localhost:9090/api/v1/query \
  --data-urlencode 'query=up' \
  --data-urlencode 'time=2024-01-01T20:10:30.781Z'

# Query range
curl -G http://localhost:9090/api/v1/query_range \
  --data-urlencode 'query=rate(http_requests_total[5m])' \
  --data-urlencode 'start=2024-01-01T00:00:00Z' \
  --data-urlencode 'end=2024-01-01T23:59:59Z' \
  --data-urlencode 'step=15s'

# Get series labels
curl -G http://localhost:9090/api/v1/series \
  --data-urlencode 'match[]=up' \
  --data-urlencode 'start=2024-01-01T00:00:00Z' \
  --data-urlencode 'end=2024-01-01T23:59:59Z'

# Get label values
curl http://localhost:9090/api/v1/label/job/values

# Get targets
curl http://localhost:9090/api/v1/targets

# Get alerts
curl http://localhost:9090/api/v1/alerts

# Get rules
curl http://localhost:9090/api/v1/rules

# Health check
curl http://localhost:9090/-/healthy

# Reload configuration
curl -X POST http://localhost:9090/-/reload
```

### Grafana HTTP API

```bash
# Authentication
export GRAFANA_TOKEN="your-api-token"
export GRAFANA_URL="http://localhost:3000"

# Create dashboard
curl -X POST "$GRAFANA_URL/api/dashboards/db" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d @dashboard.json

# Get dashboard by UID
curl "$GRAFANA_URL/api/dashboards/uid/my-dashboard" \
  -H "Authorization: Bearer $GRAFANA_TOKEN"

# Search dashboards
curl "$GRAFANA_URL/api/search?query=production&type=dash-db" \
  -H "Authorization: Bearer $GRAFANA_TOKEN"

# Delete dashboard
curl -X DELETE "$GRAFANA_URL/api/dashboards/uid/my-dashboard" \
  -H "Authorization: Bearer $GRAFANA_TOKEN"

# Create data source
curl -X POST "$GRAFANA_URL/api/datasources" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Prometheus",
    "type": "prometheus",
    "url": "http://prometheus:9090",
    "access": "proxy",
    "isDefault": true
  }'

# List data sources
curl "$GRAFANA_URL/api/datasources" \
  -H "Authorization: Bearer $GRAFANA_TOKEN"

# Create organization
curl -X POST "$GRAFANA_URL/api/orgs" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "Production Org"}'

# Add user to organization
curl -X POST "$GRAFANA_URL/api/orgs/1/users" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"loginOrEmail": "user@example.com", "role": "Viewer"}'

# Create API key
curl -X POST "$GRAFANA_URL/api/auth/keys" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name": "automation-key", "role": "Admin", "secondsToLive": 86400}'

# Create alert notification channel
curl -X POST "$GRAFANA_URL/api/alert-notifications" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Slack Alerts",
    "type": "slack",
    "isDefault": true,
    "settings": {
      "url": "https://hooks.slack.com/services/xxx/yyy/zzz",
      "recipient": "#alerts"
    }
  }'

# Create folder
curl -X POST "$GRAFANA_URL/api/folders" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title": "Production Dashboards"}'

# Snapshot dashboard
curl -X POST "$GRAFANA_URL/api/snapshots" \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d @snapshot.json

# Health check
curl "$GRAFANA_URL/api/health"
```

### Alert Rules Configuration

```yaml
# alerts/app_alerts.yml
groups:
  - name: application
    interval: 30s
    rules:
      # High error rate
      - alert: HighErrorRate
        expr: |
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          / sum(rate(http_requests_total[5m])) > 0.05
        for: 5m
        labels:
          severity: critical
          team: backend
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value | humanizePercentage }} on {{ $labels.instance }}"

      # API latency
      - alert: HighLatency
        expr: |
          histogram_quantile(0.95,
            sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
          ) > 1
        for: 10m
        labels:
          severity: warning
          team: backend
        annotations:
          summary: "API latency is high"
          description: "95th percentile latency is {{ $value }}s"

      # Service down
      - alert: ServiceDown
        expr: up{job="app"} == 0
        for: 2m
        labels:
          severity: critical
          team: sre
        annotations:
          summary: "Service {{ $labels.instance }} is down"
          description: "{{ $labels.job }} on {{ $labels.instance }} has been down for more than 2 minutes"

  - name: infrastructure
    interval: 1m
    rules:
      # High CPU
      - alert: HighCPU
        expr: |
          100 - (avg by (instance) (irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 5m
        labels:
          severity: warning
          team: sre
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
          description: "CPU usage is {{ $value }}%"

      # Low disk space
      - alert: LowDiskSpace
        expr: |
          (node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"}) * 100 < 15
        for: 5m
        labels:
          severity: warning
          team: sre
        annotations:
          summary: "Low disk space on {{ $labels.instance }}"
          description: "Only {{ $value }}% disk space remaining"

      # High memory
      - alert: HighMemory
        expr: |
          (1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100 > 90
        for: 5m
        labels:
          severity: critical
          team: sre
        annotations:
          summary: "High memory usage on {{ $labels.instance }}"
          description: "Memory usage is {{ $value }}%"
```

### Python Client (Prometheus & Grafana)

```python
import requests
from datetime import datetime, timedelta

class PrometheusClient:
    def __init__(self, url="http://localhost:9090"):
        self.url = url

    def query(self, promql):
        """Execute instant query"""
        response = requests.get(
            f"{self.url}/api/v1/query",
            params={"query": promql}
        )
        return response.json()

    def query_range(self, promql, start, end, step="15s"):
        """Execute range query"""
        response = requests.get(
            f"{self.url}/api/v1/query_range",
            params={
                "query": promql,
                "start": start.isoformat(),
                "end": end.isoformat(),
                "step": step
            }
        )
        return response.json()

    def get_targets(self):
        """Get scrape targets"""
        response = requests.get(f"{self.url}/api/v1/targets")
        return response.json()

class GrafanaClient:
    def __init__(self, url, token):
        self.url = url
        self.headers = {
            "Authorization": f"Bearer {token}",
            "Content-Type": "application/json"
        }

    def create_dashboard(self, dashboard_json):
        """Create or update dashboard"""
        response = requests.post(
            f"{self.url}/api/dashboards/db",
            headers=self.headers,
            json={"dashboard": dashboard_json, "overwrite": True}
        )
        return response.json()

    def get_dashboard(self, uid):
        """Get dashboard by UID"""
        response = requests.get(
            f"{self.url}/api/dashboards/uid/{uid}",
            headers=self.headers
        )
        return response.json()

    def search_dashboards(self, query="", tags=None):
        """Search dashboards"""
        params = {"query": query}
        if tags:
            params["tag"] = tags
        response = requests.get(
            f"{self.url}/api/search",
            headers=self.headers,
            params=params
        )
        return response.json()

    def create_annotation(self, dashboard_uid, time, text, tags=None):
        """Create annotation"""
        data = {
            "dashboardUID": dashboard_uid,
            "time": int(time.timestamp() * 1000),
            "text": text,
            "tags": tags or []
        }
        response = requests.post(
            f"{self.url}/api/annotations",
            headers=self.headers,
            json=data
        )
        return response.json()

# Usage example
prom = PrometheusClient("http://prometheus:9090")
grafana = GrafanaClient("http://grafana:3000", "your-api-token")

# Query CPU usage
result = prom.query('100 - (avg(irate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)')
print(f"CPU Usage: {result['data']['result'][0]['value'][1]}%")

# Get dashboard
dashboard = grafana.get_dashboard("my-dashboard-uid")
```

## MCP Server Integration

### Configuration (.codebuddy/mcp.json)

```json
{
  "mcpServers": {
    "grafana": {
      "command": "npx",
      "args": ["-y", "@grafana/mcp-server"],
      "env": {
        "GRAFANA_URL": "http://localhost:3000",
        "GRAFANA_API_TOKEN": "your-api-token-here"
      }
    }
  }
}
```

### Available MCP Tools

**grafana_create_dashboard**
- Creates or updates a Grafana dashboard
- Parameters: `dashboard` (JSON object), `folder_id` (optional), `overwrite` (boolean)
- Returns: Dashboard UID, URL, version

**grafana_get_dashboard**
- Retrieves dashboard by UID
- Parameters: `uid` (string)
- Returns: Full dashboard JSON with metadata

**grafana_search_dashboards**
- Searches dashboards by query and tags
- Parameters: `query` (string), `tags` (array), `folder_ids` (array)
- Returns: List of matching dashboards

**grafana_delete_dashboard**
- Deletes a dashboard
- Parameters: `uid` (string)
- Returns: Deletion confirmation

**grafana_create_datasource**
- Creates a new data source
- Parameters: `name` (string), `type` (string), `url` (string), `settings` (object)
- Returns: Data source ID and details

**grafana_query_datasource**
- Executes query against data source
- Parameters: `datasource_uid` (string), `query` (string), `time_range` (object)
- Returns: Query results

**grafana_create_alert**
- Creates alert rule
- Parameters: `rule_name` (string), `folder_uid` (string), `condition` (object), `notifications` (array)
- Returns: Alert rule ID

**grafana_list_alerts**
- Lists all alert rules
- Parameters: `folder_uid` (optional), `state` (optional)
- Returns: Array of alert rules

**grafana_create_annotation**
- Creates dashboard annotation
- Parameters: `dashboard_uid` (string), `time` (timestamp), `text` (string), `tags` (array)
- Returns: Annotation ID

**prometheus_query**
- Executes PromQL instant query
- Parameters: `query` (string), `time` (optional timestamp)
- Returns: Query result values

**prometheus_query_range**
- Executes PromQL range query
- Parameters: `query` (string), `start` (timestamp), `end` (timestamp), `step` (string)
- Returns: Time series data

**prometheus_get_metrics**
- Lists available metrics
- Parameters: `filter` (optional string)
- Returns: Array of metric names

**prometheus_get_targets**
- Lists scrape targets and their status
- Parameters: None
- Returns: Target health and labels

## Common Workflows

### 1. Complete Monitoring Stack Setup

```bash
# Docker Compose setup
cat > docker-compose.yml <<'EOF'
version: '3.8'

services:
  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - ./alerts:/etc/prometheus/alerts
      - prometheus-data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.enable-lifecycle'

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
      - GF_USERS_ALLOW_SIGN_UP=false
    volumes:
      - grafana-data:/var/lib/grafana
      - ./grafana/provisioning:/etc/grafana/provisioning

  node-exporter:
    image: prom/node-exporter:latest
    ports:
      - "9100:9100"
    command:
      - '--path.rootfs=/host'
    volumes:
      - '/:/host:ro,rslave'

  alertmanager:
    image: prom/alertmanager:latest
    ports:
      - "9093:9093"
    volumes:
      - ./alertmanager.yml:/etc/alertmanager/alertmanager.yml

volumes:
  prometheus-data:
  grafana-data:
EOF

# Start stack
docker-compose up -d

# Wait for services
sleep 10

# Create Grafana API token
GRAFANA_TOKEN=$(curl -X POST http://admin:admin@localhost:3000/api/auth/keys \
  -H "Content-Type: application/json" \
  -d '{"name": "automation", "role": "Admin"}' | jq -r '.key')

# Add Prometheus data source
curl -X POST http://localhost:3000/api/datasources \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Prometheus",
    "type": "prometheus",
    "url": "http://prometheus:9090",
    "access": "proxy",
    "isDefault": true
  }'

echo "Monitoring stack ready!"
echo "Grafana: http://localhost:3000 (admin/admin)"
echo "Prometheus: http://localhost:9090"
```

### 2. Create Custom Application Dashboard

```bash
# Generate dashboard JSON
cat > app-dashboard.json <<'EOF'
{
  "dashboard": {
    "title": "Application Metrics",
    "tags": ["application", "production"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "graph",
        "gridPos": {"x": 0, "y": 0, "w": 12, "h": 8},
        "targets": [
          {
            "expr": "sum(rate(http_requests_total[5m])) by (status)",
            "legendFormat": "{{status}}",
            "refId": "A"
          }
        ],
        "yaxes": [
          {"format": "reqps", "label": "Requests/sec"},
          {"format": "short"}
        ]
      },
      {
        "id": 2,
        "title": "Error Rate",
        "type": "singlestat",
        "gridPos": {"x": 12, "y": 0, "w": 6, "h": 8},
        "targets": [
          {
            "expr": "sum(rate(http_requests_total{status=~\"5..\"}[5m])) / sum(rate(http_requests_total[5m])) * 100",
            "refId": "A"
          }
        ],
        "format": "percent",
        "thresholds": "1,5",
        "colors": ["#299c46", "#e5ac0e", "#bf1b00"]
      },
      {
        "id": 3,
        "title": "Response Time (p95)",
        "type": "graph",
        "gridPos": {"x": 0, "y": 8, "w": 12, "h": 8},
        "targets": [
          {
            "expr": "histogram_quantile(0.95, sum by (le, endpoint) (rate(http_request_duration_seconds_bucket[5m])))",
            "legendFormat": "{{endpoint}}",
            "refId": "A"
          }
        ],
        "yaxes": [
          {"format": "s", "label": "Duration"},
          {"format": "short"}
        ]
      },
      {
        "id": 4,
        "title": "Active Connections",
        "type": "graph",
        "gridPos": {"x": 12, "y": 8, "w": 12, "h": 8},
        "targets": [
          {
            "expr": "sum(app_active_connections) by (instance)",
            "legendFormat": "{{instance}}",
            "refId": "A"
          }
        ]
      }
    ],
    "refresh": "30s",
    "time": {"from": "now-6h", "to": "now"}
  },
  "overwrite": true
}
EOF

# Upload dashboard
curl -X POST http://localhost:3000/api/dashboards/db \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d @app-dashboard.json
```

### 3. Set Up Alerting Pipeline

```bash
# Configure Alertmanager
cat > alertmanager.yml <<'EOF'
global:
  resolve_timeout: 5m
  slack_api_url: 'https://hooks.slack.com/services/xxx/yyy/zzz'

route:
  group_by: ['alertname', 'cluster']
  group_wait: 10s
  group_interval: 10s
  repeat_interval: 12h
  receiver: 'default'
  routes:
    - match:
        severity: critical
      receiver: 'pagerduty'
      continue: true
    - match:
        severity: warning
      receiver: 'slack'

receivers:
  - name: 'default'
    slack_configs:
      - channel: '#alerts'
        title: 'Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'

  - name: 'slack'
    slack_configs:
      - channel: '#alerts-warning'
        title: 'Warning: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.summary }}{{ end }}'
        send_resolved: true

  - name: 'pagerduty'
    pagerduty_configs:
      - service_key: 'your-pagerduty-key'
        description: '{{ .GroupLabels.alertname }}'

inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'instance']
EOF

# Reload Alertmanager
curl -X POST http://localhost:9093/-/reload

# Test alert
curl -X POST http://localhost:9093/api/v1/alerts \
  -H "Content-Type: application/json" \
  -d '[
    {
      "labels": {
        "alertname": "TestAlert",
        "severity": "warning"
      },
      "annotations": {
        "summary": "This is a test alert"
      }
    }
  ]'

# Verify alert rules
curl http://localhost:9090/api/v1/rules | jq '.data.groups[].rules[] | select(.type=="alerting")'

# Check active alerts
curl http://localhost:9090/api/v1/alerts | jq '.data.alerts[] | select(.state=="firing")'
```

### 4. Instrument Application with Metrics

```python
# Python Flask application with Prometheus metrics
from flask import Flask, request
from prometheus_client import Counter, Histogram, Gauge, generate_latest, REGISTRY
import time

app = Flask(__name__)

# Metrics
REQUEST_COUNT = Counter(
    'http_requests_total',
    'Total HTTP requests',
    ['method', 'endpoint', 'status']
)

REQUEST_DURATION = Histogram(
    'http_request_duration_seconds',
    'HTTP request duration',
    ['method', 'endpoint']
)

ACTIVE_REQUESTS = Gauge(
    'http_requests_active',
    'Active HTTP requests'
)

# Middleware
@app.before_request
def before_request():
    request.start_time = time.time()
    ACTIVE_REQUESTS.inc()

@app.after_request
def after_request(response):
    duration = time.time() - request.start_time

    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown',
        status=response.status_code
    ).inc()

    REQUEST_DURATION.labels(
        method=request.method,
        endpoint=request.endpoint or 'unknown'
    ).observe(duration)

    ACTIVE_REQUESTS.dec()
    return response

# Metrics endpoint
@app.route('/metrics')
def metrics():
    return generate_latest(REGISTRY)

# Application endpoints
@app.route('/api/users')
def users():
    return {'users': []}

@app.route('/health')
def health():
    return {'status': 'healthy'}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
```

```bash
# Add to Prometheus scrape config
cat >> prometheus.yml <<'EOF'
  - job_name: 'my-app'
    static_configs:
      - targets: ['app:8080']
    metrics_path: '/metrics'
    scrape_interval: 10s
EOF

# Reload Prometheus
curl -X POST http://localhost:9090/-/reload
```

### 5. Performance Analysis and Troubleshooting

```bash
# Check if metrics are being scraped
curl 'http://localhost:9090/api/v1/query?query=up{job="app"}'

# Analyze request patterns
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=topk(10, sum by (endpoint) (rate(http_requests_total[1h])))'

# Find slow endpoints
curl -G 'http://localhost:9090/api/v1/query' \
  --data-urlencode 'query=histogram_quantile(0.99, sum by (le, endpoint) (rate(http_request_duration_seconds_bucket[5m]))) > 1'

# Memory leak detection
curl -G 'http://localhost:9090/api/v1/query_range' \
  --data-urlencode 'query=process_resident_memory_bytes{job="app"}' \
  --data-urlencode 'start=2024-01-01T00:00:00Z' \
  --data-urlencode 'end=2024-01-01T23:59:59Z' \
  --data-urlencode 'step=1h' | jq '.data.result[0].values'

# CPU spike investigation
curl -G 'http://localhost:9090/api/v1/query_range' \
  --data-urlencode 'query=rate(process_cpu_seconds_total{job="app"}[5m])' \
  --data-urlencode 'start=2024-01-01T10:00:00Z' \
  --data-urlencode 'end=2024-01-01T11:00:00Z' \
  --data-urlencode 'step=30s'

# Correlate errors with deployments (using annotations)
curl -X POST http://localhost:3000/api/annotations \
  -H "Authorization: Bearer $GRAFANA_TOKEN" \
  -H "Content-Type: application/json" \
  -d "{
    \"time\": $(date +%s)000,
    \"text\": \"Deployed v2.1.0\",
    \"tags\": [\"deployment\", \"v2.1.0\"]
  }"

# Export metrics for analysis
curl -G 'http://localhost:9090/api/v1/query_range' \
  --data-urlencode 'query=http_requests_total' \
  --data-urlencode 'start=2024-01-01T00:00:00Z' \
  --data-urlencode 'end=2024-01-01T23:59:59Z' \
  --data-urlencode 'step=5m' | jq -r '.data.result[] | [.metric.instance, .values[][1]] | @csv' > metrics.csv
```
