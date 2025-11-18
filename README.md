
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

### Centralize logs from EKS
- Use:
     - FluentBit DaemonSet
     - Output to: CloudWatch Logs / S3 / OpenSearch
     - Use structured JSON logs with trace-id correlation
     - CloudWatch subscription filters → Firehose → S3/OpenSearch
     - Use AWS-managed add-ons for simplicity

### Distributed tracing
- Use
     - OpenTelemetry SDK or language auto-instrumentation
     - Deploy ADOT Collector as sidecar/DaemonSet
     - Propagate W3C trace headers
- Export to:
     - X-Ray
     - or Datadog/New Relic/Tempo
- Output: spans, service map, latency breakdown, root cause.

### SLOs & Error Budgets
- Define success metrics (e.g., 99.9% availability)
- Extract via CloudWatch Metrics (e.g., 5xx count / total requests)
- Use Metric Math to compute SLO compliance
- Burn-rate alerts:
     - 1h fast burn
     - 6h slow burn
- Grafana SLO panels for visibility

### CloudWatch log cost
- Use log expiration
- Use subscription filters → S3 for long-term retention
- Compress logs at source (FluentBit)
- Reduce log verbosity in prod (info vs debug)
- Use CloudWatch Metric Filters instead of scanning logs

### Monitor Lambda
- Out-of-box metrics (invocations, duration, errors, throttles)
- p90/p95 latency metrics
- Custom metrics via EMF
- X-Ray tracing
- Log-based alerts (timeouts, memory leaks)

### Monitor EKS cluster and node health
- Container Insights for CPU/memory/disk/network
- Prometheus → AMP
- Node-exporter
- Kube-state-metrics
- FluentBit for logs
- Grafana dashboards
- Integrate Cluster Autoscaler logs/metrics

### correlate logs, metrics, and traces
- Use trace IDs in logs
- Propagate W3C headers
- CloudWatch→X-Ray integration
- Grafana Observability stack to join logs/metrics/traces

### VPC Flow Logs
- Network-level observability:
     - Track traffic in/out of ENIs
     - Troubleshoot blocked traffic, latency
     - Feed to S3/OpenSearch for security analysis
     - Integrate with GuardDuty

### API calls
- Use AWS CloudTrail:
     - Records every API call
     - Sends events to S3
     - Use CloudTrail Lake for queries
     - Integrate with GuardDuty & Security Hub

## Questions

🔥 Incident & Troubleshooting Scenarios
1. Your API’s p99 latency suddenly spikes, but CPU, memory, and traffic are normal. How do you troubleshoot end-to-end?
2. You see intermittent 5xx errors from a microservice in EKS, but only from one node. Logs look clean. What do you investigate?
3. CloudWatch shows sudden drop in request count for an ALB, but app logs show traffic coming in normally. How do you debug?
4. A service is slow only during deployments. How do you instrument and find the bottleneck?
5. Your worker queue (SQS → Lambda) is backed up. CloudWatch metrics show Lambda duration normal. What next?
🔥 Log & Metric Issues
6. Your CloudWatch Logs bill suddenly doubled. How do you find which service caused it and fix the pipeline?
7. You receive incomplete logs from EKS (missing lines or out of order). What are the root causes you suspect?
8. OpenSearch query latency is high and dashboards are slow. What do you check?
9. CloudWatch Metrics for CPU differ from node-exporter metrics in Prometheus. Why?
10. You see high cardinality metrics exploding CloudWatch usage. How do you control it?
🔥 Distributed Tracing & Correlation Scenarios
11. Traces from a single service are missing from X-Ray. Others are fine. What steps do you take?
12. You see broken trace graphs (orphan spans, missing parents). What are the causes?
13. Trace IDs are generated, but logs don’t show them. Where do you look?
14. How do you debug a scenario where ADOT Collector is dropping spans under high load?
15. You see long latencies in a microservice but trace shows each hop is fast. What else could be slow?
🔥 Alerts, SLOs, and Error Budgets
16. Your on-call receives high-severity alerts at night, but by the time you check, everything is normal. How do you stop “flapping” alerts?
17. A service meets its SLA but violates SLO. Explain how you detect this operationally.
18. Traffic doubled unexpectedly. Error budget burn rate is high. What are the next steps?
19. You need to design multi-window burn rate alerts for an API. What windows do you choose and why?
20. A team wants to tighten SLO from 99.5% to 99.9%. What trade-offs do you highlight?
🔥 Observability Architecture & Design Scenarios
21. Your logs are stored in S3, but queries are slow. How do you speed up investigation?
22. How would you design log ingestion for 50k EKS pods generating 5 TB/day logs?
23. You need to migrate from CloudWatch Logs to OpenSearch with zero downtime. What’s the approach?
24. A service uses Lambda, DynamoDB, SQS, and API Gateway. How do you implement full transaction-level tracing?
25. You see packet drops in EKS but node logs show nothing. How do you debug networking?
🔥 Performance & Cost Optimization Scenarios
26. CloudWatch alarm evaluation is slow (delayed alarms). What do you check?
27. Prometheus scraping is slow and missing metrics. How do you debug scrape performance in EKS?
28. You need to reduce observability cost by 40% without losing visibility. What do you cut first?
29. OpenSearch indexing rate is at max, writes failing. How do you scale it safely?
30. You notice CloudWatch Logs ingestion throttling during peak traffic. Fix?












