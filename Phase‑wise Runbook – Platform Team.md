Phase‑wise Runbook – Platform Team
Phase 0 — Foundations & Readiness
Objectives

Confirm scope, SLAs/SLIs, data governance, security posture.
Select IaC tooling (Terraform or CDK), naming conventions, tagging strategy.
Establish environments: Dev / Test / Prod, account structure, and guardrails.

Key Deliverables

Architecture decision record (ADR)
IaC repo skeleton(s) and CI/CD pipelines
Baseline KMS keys, CloudTrail, AWS Config, Service Control Policies (SCPs) (if using AWS Org)

Entry Criteria

Business goals and event types agreed (telemetry/heartbeat/fault/transactions)
Approval for shared services (e.g., NAT, VPC endpoints, log archive)

Exit Criteria

Signed-off ADR; repos initialized; pipeline green with a no‑op plan
Security baselines enabled (Trail/Config/KMS)


Phase 1 — Infrastructure Provisioning (Start here)
You’re in the right flow. Provision network, identities, compute, and storage first so downstream services have a secure, stable foundation.
Objectives

Build VPC with public/private subnets across ≥2 AZs, IGW, NAT per AZ, and VPC Endpoints (S3, CloudWatch Logs, Secrets Manager, DynamoDB).
Stand up Amazon EKS (private subnets), enable IRSA for pod‑level IAM.
Create Amazon MSK (multi‑AZ, TLS, IAM/SASL).
Create S3 buckets and Glue Catalog stubs for Raw and Analytics (Iceberg).

Detailed Tasks


Networking

VPC (e.g., 10.0.0.0/16), subnets (/20 public, /20 private app, /20 private data).
Route Tables with IGW on public, NAT from private to internet where required.
VPC Endpoints: S3 (Gateway), CloudWatch Logs/Secrets Manager/DynamoDB (Interface).
Security Groups:

EKS nodes: egress only; allow pod‑to‑MSK TLS (9092), node‑to‑ALB as needed.
MSK brokers: restrict to client SGs only, TLS enforced.





Identity & Access

KMS keys: one for S3 data (Raw/Analytics), optionally separate key for MSK client encryption.
IRSA roles:

Event Collector (EKS): Kafka describe/bootstrap; topic read/write; CloudWatch Logs; Secrets; KMS decrypt.
Glue Jobs: S3 get/put/list; Glue Catalog; Logs; KMS encrypt/decrypt.
Lambda Orchestrator: SQS, DynamoDB, Logs, Secrets, KMS.





Compute & Streaming

EKS control plane + node groups (private subnets).
Add‑ons: Cluster Autoscaler, Metrics Server, Fluent Bit/CloudWatch logs, Ingress (ALB) if needed.
MSK: multi‑AZ brokers, RF=3, min ISR ≥ 2, TLS, IAM/SASL auth, broker logs to CloudWatch (optional).



Storage (Data Lake)

S3: dl-raw, dl-analytics buckets.

Enable SSE‑KMS, versioning, lifecycle to Glacier per policy.


Glue Catalog: databases dl_raw, dl_analytics; empty Iceberg tables or table placeholders.



Validation

Smoke tests:

VPC endpoints reachable from private subnets (S3/Logs/Secrets).
EKS: kubectl ready; IRSA token projected; sample pod writes logs to CloudWatch.
MSK: client on a private instance can DescribeCluster and get bootstrap brokers; TLS handshake succeeds.
S3: write/read with KMS; access denied from unintended roles.



Exit Criteria

EKS, MSK, S3, Glue baselines are healthy; CloudWatch shows broker/node metrics; IaC pipelines produce repeatable plans/applies.

Risks & Mitigations

NAT bottleneck → per‑AZ NAT + endpoints for high‑volume services.
IAM sprawl → central module for roles; least‑privilege audits.
Private DNS/Egress issues → test endpoints early; document required egress (ServiceNow if via internet/NAT or via proxy).


Phase 2 — Ingestion (EKS Event Collector)
Objectives

Deploy the Event Collector (mTLS/JWT auth), normalization (add ingest_ts, schema_version), schema validation, and quarantine routing.

Tasks

Helm/Kustomize deployment with IRSA role.
ALB Ingress (public if devices call over internet; private + VPN if enterprise).
Configure Kafka producer: acks=all, enable.idempotence=true, compression=snappy, sensible batch.size/linger.ms.

Exit Criteria

Valid events land in MSK topics; invalid events in quarantine topic and S3 raw/quarantine/; CloudWatch shows validation metrics.


Phase 3 — Streaming Backbone (MSK)
Objectives

Finalize topics and consumer groups; ensure scaling/partitioning.

Tasks

Topics: iot.telemetry.v1, iot.heartbeat.v1, iot.faults.v1, iot.transactions.v1, iot.quarantine.v1.
Partition sizing to peak throughput; monitor Consumer Lag, BytesIn/Out, ISR.

Exit Criteria

No under‑replicated partitions; lag within targets; ACLs enforced.


Phase 4 — Storage & Batch (S3 Iceberg + Glue Batch)
Objectives

Persist immutable raw; curate ODS in analytics.

Tasks

Write raw streams → S3 Raw (Iceberg) (append‑only).
Glue Batch ETL: cleansing, enrichment (device metadata), schema evolution; write curated ODS → S3 Analytics (Iceberg); enable compaction jobs.

Exit Criteria

ODS tables queryable via Athena; batch completes within SLA (e.g., 06:00 local).


Phase 5 — Real‑Time Stream Processing (Glue Streaming)
Objectives

Detect anomalies and incident candidates.

Tasks

Configure windowing (e.g., heartbeat miss N minutes; fault burst ≥3 in 10 minutes).
Emit candidates to a stream/Kinesis/SQS for Lambda.

Exit Criteria

Candidate events generated at low latency (P95 ≤ 2 min).


Phase 6 — Operational Alerting (Lambda + SQS DLQ + DynamoDB + ServiceNow)
Objectives

Create/aggregate incidents reliably and within rate limits.

Tasks

Lambda rules: dedup (device + incident type), severity classification, throttling/backoff.
DynamoDB: device_last_seen, open_incidents (TTL for windows).
SQS DLQ for failures; scheduled replay.

Exit Criteria

ServiceNow incidents created within P95 ≤ 5 min from detection; DLQ depth remains near zero.


Phase 7 — Consumption (Athena & BI)
Objectives

Self‑service analytics + operational reporting.

Tasks

Athena workgroups; Glue DBs: dl_raw, dl_analytics.
Power BI datasets on curated ODS; refresh every 15–60 minutes for ops; daily for finance.

Exit Criteria

BI dashboards stable; query hygiene enforced (partition pruning, column selection).


Phase 8 — Observability & Monitoring
Objectives

End‑to‑end visibility and proactive alerts.

Tasks

CloudWatch Logs/Metrics for EKS, MSK, Glue (batch/stream), Lambda.
Central S3 log archive; Managed Grafana dashboards:

Edge‑to‑Incident funnel
Heartbeat Health map
Fault Burst tracker
Consumer Lag watch


Alarms: ingestion 4xx/5xx, under‑replicated partitions, lag thresholds, DLQ depth.

Exit Criteria

All dashboards populated; alarms tuned to actionable thresholds.


Phase 9 — Operations: Procedures, Playbooks & Checklists
Day‑to‑Day Ops Procedures

Morning health checks (EKS pods/ingress, MSK ISR/lag, Glue streaming latency, Lambda errors/DLQ, Athena smoke query).
Schema rollout via canaries; incident flow test (synthetic fault).
Replay from raw partitions when needed.

Failure Playbooks

Ingestion: auth failures → rotate certs/keys; malformed payloads → quarantine + vendor notice; scale pods via HPA.
MSK: under‑replication → rebalance/replace broker; AZ event → failover validation.
Glue lag: boost DPUs/parallelism; split streams; reduce transformation cost.
ServiceNow throttling: exponential backoff; aggregate incidents; DLQ replay.
Data Quality: rollback schema; quarantine; corrective ETL.

Operational Checklists

Daily: dashboards review, lag, DLQ, heartbeat distribution.
Weekly: cost review, Iceberg compaction, schema backlog triage, severity mapping audits.
Monthly: DR drill (tabletop/partial), IAM/Kafka ACL reviews, DQ sampling.


Security & Compliance

Encryption: TLS in transit; SSE‑KMS at rest (S3/MSK).
Least‑Privilege: IRSA per workload; Kafka ACLs; restricted workgroups in Athena.
PII: tokenize/redact at ingestion if applicable; segregated buckets; stricter access.
Retention: Raw 365–730 days; faults ≥180 days; transactions per finance (e.g., 7 years) with Glacier.
Audit: CloudTrail, Glue Catalog change logs; GitOps PR approvals.


Capacity & Cost Management

Kafka partitioning per peak throughput; monitor BytesIn/Out.
Glue autoscaling/DPUs; streaming checkpoints; batch compaction to reduce small files.
S3 lifecycle transitions; Athena query hygiene (avoid SELECT *, partition pruning).
Grafana panel refresh tuning; alarm suppression policies.


SLOs & SLIs (Examples)

Ingestion availability (Edge→Kafka): ≥ 99.9% monthly.
Time to detect candidate: P95 ≤ 2 min from event_ts.
Time to incident creation (SNOW): P95 ≤ 5 min from detection.
ODS freshness: batch complete by 06:00 local.
DQ: invalid events < 0.5%; duplicates < 0.2%.


DR & Continuity

MSK: Multi‑AZ, topic config snapshots, consumer offset backups.
S3: Versioning + optional CRR for Raw/Analytics.
IaC: redeploy scripts for DR region; secrets replicated.
Quarterly drills: simulate regional impact; target RPO ≤ 1h, RTO ≤ 4h.
