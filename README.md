
# Core Concepts
- **Monitoring** = Collecting known signals to track system health.
- **Observability** = Ability to understand why the system behaves a certain way, using logs, metrics, traces.

### AWS Native Observability Stack
- Metrics (Time-Series Data)
     - Amazon CloudWatch Metrics
          - Infra metrics (EC2, ALB, RDS, Lambda)
          - Custom metrics via CloudWatch Agent / AWS Distro for OpenTelemetry (ADOT)
          - Supports p90/p95/p99 percentiles, anomaly detection
     - CloudWatch Contributor Insights
          - Top N contributors causing load/latency/errors
- Logs
     - CloudWatch Logs
          - Central log storage for app/system logs
          - Log Insights for querying
     - AWS OpenSearch / Managed Elasticsearch (Loki alternative)
          - Log indexing + KQL queries
          - Useful for high-volume, multi-team log search
- Traces
     - AWS X-Ray
          - Application traces, service map
          - Root cause for slow segments
     - ADOT (OpenTelemetry Collector)
          - Vendor-neutral tracing/metrics/logs export
          - Supports exporting to Datadog, New Relic, Dynatrace, Grafana Tempo, etc.

### Enterprise Observability Architecture (Typical)
- Data Collectors
     - CloudWatch Agent (logs/metrics)
     - ADOT Collector (traces/metrics/logs)
     - FluentBit (custom log routing)
- Data Stores
     - Metrics → CloudWatch
     - Logs → CloudWatch Logs → S3 → (Optional) OpenSearch
     - Traces → X-Ray or external APM (Datadog/New Relic)
- Analytics / Dashboard
     - CloudWatch Dashboards
     - QuickSight (BI dashboards)
     - Grafana (Managed Grafana on AWS)

### What Platform Engineers Typically Build
- Observability-as-a-Service
     - Central ADOT collector
     - Common dashboards & alerts
     - Standard log formats (JSON structured logs)
- Golden Signals
     - Platform team standardizes:
          - Latency (p50/p90/p95/p99)
          - Traffic
          - Errors
          - Saturation
- SLO/SLA/SLA Enforcement
     - Error budget calculation
     - SLO dashboards via CloudWatch or Grafana
- Alerting Standards
     - Unified alert routing: SNS → PagerDuty/Slack
     - Alert playbooks & auto-remediation (Lambda / SSM runbooks)

### SRE/Platform Engineer
- Works on
     - Unified logging architecture (FluentBit → CW/S3/OpenSearch)
     - Central ADOT collector for traces & metrics
     - Standard dashboard templates
     - Alert routing standards
     - SLO/Error-budget dashboards
     - Auto-remediation (Lambda/SSM documents)
     - Observability-as-a-Service

### Golden Signals
 - Latency (p50/p90/p95/p99)
 - Traffic (RPS, throughput)
 - Errors (4xx/5xx, exceptions)
 - Saturation (CPU/memory/queue length)

### calculate p90/p95/p99
- Percentiles represent the value below which X% of observations fall.
- AWS provides percentile metrics via:
     - CloudWatch Metric Math
     - CloudWatch EMF logs
     - ALB/NLB/Lambda built-in percentile metrics
- Example:
     - p95 latency = 95th percentile request duration in a 5-min window.
