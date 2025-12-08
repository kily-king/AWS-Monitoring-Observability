AWS Migration Plan
Executive Summary
This document outlines the migration from Azure EventHub, Stream Analytics, Logic Apps, and NiFi to
AWS MSK, Kafka Streams/Flink, Lambda, EventBridge, MSK Connect, and Timestream.
Phase 0 – Discovery & Requirements (1–2 weeks)
• Collect rules, schemas, SNOW API details.
• Export SAQL logic and Logic Apps workflow.
• Validate device event formats and ODS schemas.
Phase 1 – Core AWS Infrastructure (2 weeks)
• Deploy VPC, subnets, security groups.
• Deploy Amazon MSK.
• Create Kafka topics.
• Deploy Timestream.
• Configure IAM roles and Secrets Manager.
Phase 2 – ODS Ingestion (NiFi Replacement) (2 weeks)
• Deploy MSK Connect with JDBC Sink.
• Map topics to ODS staging tables.
• Validate ingestion at scale.
Phase 3 – Rules Engine (Stream Analytics Replacement) (2 weeks)
• Implement delay timer rules.
• Implement aggregation rules.
• Implement status/mode rules.
• Emit alerts into Kafka topics.
Phase 4 – ServiceNow Integration (Logic Apps Replacement) (2
weeks)
• Build Lambda → SNOW API integration.
• Implement incident, append, and asset update logic.
• Add retries, DLQ, logging, and metrics.
Phase 5 – Heartbeat → Timestream (2 weeks)
• Deploy heartbeat consumer.
• Store raw and aggregated metrics in Timestream.
• Build QuickSight/Grafana dashboards.
Phase 6 – Parallel Run & Cutover (2 weeks)
• Validate AWS pipeline matches Azure outputs.
• Compare incidents, alerts, ODS, dashboards.
• Toggle traffic to AWS.
• Decommission Azure components.
Risks & Mitigations
• Rule mismatch → mitigated with parallel run.
• SNOW integration errors → mitigated with Lambda retries & DLQs.
• Topic schema drift → mitigated with schema registry governance.
Final Outcome
A complete AWS-native streaming, alerting, monitoring, and data analytics pipeline replacing all Azure
dependencies.
