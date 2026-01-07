Device & Hardware Telemetry (Machine Health)
- Collected to ensure the machine is alive, healthy and operational.
Examples
	- Device ID / Machine ID
	- Heartbeat timestamp (every X seconds/minutes)
	- Power status (on/off, voltage)
	- Temperature (internal, refrigeration)
	- Door open/close status
	- Motor status (jammed / running)
	- Sensor health (coin sensor, bill validator, product sensor)
	- CPU, memory, disk usage (for smart machines)
Why this is collected
	- Detect machine offline
	- Prevent failures (overheating, power instability)
	- Enable predictive maintenance
	- SLA monitoring
Transaction & Sales Events
- Collected for business revenue tracking and reconciliation.
Examples
	- Transaction ID
	- Product ID 
	- Price
	- Payment method (cash, card, QR)
	- Payment success / failure
	- Vend success / failure
	- Refund events
	- Timestamp
Why this is collected
	- Revenue reporting
	- Payment reconciliation
	- Fraud detection
	- Identify failed vend issues
Inventory & Stock Levels
- Collected to know what is running out and when to refill.
Examples
	- Slot number → product mapping
	- Current stock count per slot
	- Low-stock threshold crossed
	- Out-of-stock events
	- Refill confirmation events
Why this is collected
	- Optimize refill routes
	- Avoid lost sales
	- Improve customer satisfaction
	- Demand forecasting
Error, Fault & Alarm Events
- Collected when something goes wrong.
Examples
	- Motor jam error
	- Payment device failure
	- Door forced open
	- Cooling failure
	- Sensor malfunction
	- Repeated transaction failures
Why this is collected
	- Trigger alerts (ServiceNow incidents)
	- Root cause analysis
	- Reduce downtime
	- Compliance & audit trails
Security & Tamper Events
- Collected to protect assets and cash.
Examples
	- Unauthorized door opening
	- Shock / vibration detected
	- Power cut events
	- Firmware integrity check failed
	- Multiple failed payment attempts
Why this is collected
	- Theft detection
	- Compliance
	- Insurance & audit evidence
Configuration & Firmware Data
- Collected when machines are updated or configured.
Examples
	- Firmware version
	- Configuration version
	- Feature flags enabled
	- Last update time
	- Update success / failure
Why this is collected
	- Fleet version management
	- Rollback during failures
	- Debugging device-specific issues
Location & Connectivity Data
- Collected to understand where and how the machine operates.
Examples
	- Machine location ID (site, store, station)
	- GPS coordinates (optional)
	- Network type (4G / 5G / Ethernet)
	- Signal strength
	- Network latency
Why this is collected
	- Regional performance analysis
	- Network troubleshooting
	- Location-based reporting
Logs & Diagnostics (Smart Machines)
- Collected mainly from Linux/Android-based vending machines.
Examples
	- Application logs
	- System logs
	- Crash dumps
	- Startup/shutdown logs
Why this is collected
	- Debugging
	- Root cause analysis
	- Post-incident investigation

	
Edge Layer – Device Event Sources
Vending Machines
 (IoT Devices)
• Telemetry
• Heartbeat
• Fault & error events
• Transactional events
  Ingestion Layer – Controlled Event   intake
Event Collector 
(Amazon EKS)
• Device authentication
• Event normalization
• Schema validation
• Secure ingestion

CloudWatch Logs
• Collects logs from EKS, MSK, Glue, Lambda & SQS  • Centralized pipeline log storage  • Supports troubleshooting & debugging  • Integrates with S3 for long‑term archival
Storage Layer – System of Record
Amazon MSK 
(Kafka Cluster)
• Central event backbone
• Producer / consumer decoupling
• Multi-AZ brokers

Amazon S3 
(Raw Zone – Iceberg)
• Immutable device events
• Long-term retention
• Audit & replay
AWS Glue
(Batch ETL)
• Cleansing
• Enrichment
• Schema evolution

AWS Glue Data Catalog
• Metadata management
Amazon S3 
(Analytics - Iceberg)
• Immutable device events  • Long‑term retention  • Audit & replay




Observability &   Monitoring



Amazon S3 
(Centralized Log Archive)
• Long‑term storage of all device & system logs  • Durable, low‑cost retention  • Supports audit & compliance needs  • Source for historical analysis via Athena  • Central place for replay and forensic investigation

CloudWatch Metrics
• Pipeline health monitoring  • Real‑time failure and performance alerts  • Ingestion rate and latency tracking  • Error and retry count visibility  • Grafana dashboard integration for insights
Amazon Managed Grafana
(Operational Dashboards)
• Visualizes pipeline and device health  • Real‑time dashboards using CloudWatch Metrics  • Operational insights for alerts, latency & throughput  • Central monitoring for MSK, Glue, Lambda
AWS Athena 
(Analytics)
• Ad‑hoc SQL querying on Iceberg datasets stored in S3  • Fast analytics on device last‑seen and incident history  • Powering dashboards (Power BI / Grafana) without servers  • Enables quick troubleshooting of telemetry and fault events  • Cost‑efficient pay‑per‑query model for operational reporting

Real-Time Stream Processing 
AWS Glue Streaming Job
• Stream-based ETL
• Lightweight transformations
• Incident Identification

Amazon EventBridge 
(Business Event Bus)
Rule-based filtering
• Event routing & fan-out
• Retry & DLQ support

Operational Alerting – Real-Time

AWS Lambda
(Incident Orchestration)
• Business rules
• Deduplication
• Stateful logic
• Rate limiting
• DLQ handling
• Controlled retries
• ServiceNow safety

ServiceNow 
(Incident & Asset APIs)
• Centralized creation and updating of incident tickets  • Stores and tracks device‑level asset information  • Enables correlation between alerts and existing incidents  • Provides workflow automation for remediation and notifications  • Acts as the authoritative ITSM system for audit and compliance

Batch Processing & Analytics
Amazon SQS 
(Dead Letter Queue)
• Buffering of incoming alert events  • Smooths traffic spikes before reaching Lambda  • Decouples Glue from downstream failures  • Guarantees message durability until processed  • Supports controlled rate of incident creation  • Ensures reliable delivery
Amazon DynamoDB (Optional State Store)
• Device last-seen  • Open incident tracking  • Deduplication  • ServiceNow incident reference mapping  • Last alert timestamp 
• Alert state lifecycle (open → update → resolved)  • Severity history for incident updates






# Confulence 

# 🧩 Solution Design – Vending Machine Telemetry, Observability & Incident Management

---

## 1. Purpose of This Document

This document describes the **end-to-end design** of the Vending Machine Telemetry and Incident Management platform.
It explains **how data flows**, **why each AWS service is used**, and **how incidents are detected and acted upon**, based on the **approved architecture diagram**.

This page is intended for:

* Architecture & platform teams
* Operations & NOC teams
* Security & compliance reviewers
* New engineers onboarding to the platform

---

## 2. High-Level Flow Overview (End-to-End)

" Here i will paste the LucidChart Diagram "

Each layer is **loosely coupled** and can scale or fail independently.

# High-Level Architecture Overview
1. The solution follows an event-driven, layered architecture:
2. Edge Layer – Vending machines generate telemetry
3. Ingestion Layer – Secure, normalized event intake
4. Streaming Backbone – Durable event transport
5. Stream Processing – Real-time analytics and incident detection
6. Alerting & Orchestration – Controlled delivery to ITSM
7. Storage & Analytics – Long-term and analytical data storage
8. Observability – Logs, metrics, dashboards

Each layer has a single responsibility, reducing coupling and operational risk.
---

## 3. Edge Layer – Device Event Sources

### Services / Components

* **Vending Machines (IoT / Smart Devices)**

### What happens here

Vending machines generate multiple categories of events:

* Machine health (heartbeat, temperature, power)
* Transactions and sales
* Inventory and refill status
* Errors, faults, and alarms
* Security and tamper events
* Firmware and configuration updates
* Location and connectivity data
* Logs and diagnostics (smart machines)

### Why this layer exists

* Represents the **source of truth** from the physical world
* No analytics or decisions are made at the device level
* Keeps devices simple and lightweight

---

## 4. Ingestion Layer – Controlled Event Intake

### Services

* **Event Collector Application**
* **Amazon EKS**

### Responsibilities

* Secure device authentication
* Schema validation and normalization
* Metadata enrichment (device ID, timestamps, site)
* Controlled ingress into the cloud platform

### Why EKS is used

* Supports scalable ingestion workloads
* Enables rolling upgrades and isolation
* Avoids exposing Kafka directly to devices

📌 **Key principle:**

> Devices never talk directly to Kafka.

---

## 5. Storage Layer – System of Record

### 5.1 Amazon MSK (Kafka Cluster)

**Role**

* Central event backbone
* Durable, ordered, replayable event storage
* Decouples producers and consumers

**Why MSK**

* Handles high-volume, bursty device traffic
* Allows multiple downstream consumers
* Supports reprocessing after failures or logic changes

---

### 5.2 Amazon S3 – Raw Zone (Iceberg)

**Role**

* Immutable storage of raw device events
* Long-term retention for audit and replay

**Why Iceberg**

* Schema evolution
* Time-travel queries
* Works natively with Athena and Glue

---

## 6. Real-Time Stream Processing

### Service

* **AWS Glue Streaming Job**

### Responsibilities

* Consume events from Kafka
* Perform lightweight stream transformations
* Identify incident conditions (rules, thresholds, patterns)
* Emit **business-level incident events**

### Why Glue Streaming

* Fully managed streaming ETL
* No cluster management
* Suitable for near-real-time detection (seconds to minutes)

📌 This is where **raw telemetry becomes operational signals**.

---

## 7. Business Event Routing

### Service

* **Amazon EventBridge**

### Responsibilities

* Acts as the **business event bus**
* Rule-based routing of incident events
* Fan-out to downstream consumers
* Built-in retry and DLQ support

### Why EventBridge

* Decouples detection from action
* Enables future integrations without refactoring
* Supports governance and auditability

---

## 8. Operational Alerting & Orchestration

### 8.1 AWS Lambda – Incident Orchestration

**Responsibilities**

* Apply business rules
* Deduplicate alerts
* Rate-limit ServiceNow API calls
* Handle retries and transient failures
* Ensure controlled incident creation

📌 Lambda does **not** perform analytics.

---

### 8.2 Amazon SQS – Dead Letter Queue

**Role**

* Buffers failed or delayed alert events
* Smooths traffic spikes
* Guarantees message durability
* Enables safe retries without data loss

---

## 9. IT Service Management

### Service

* **ServiceNow (Incident & Asset APIs)**

### Responsibilities

* Create and update incidents
* Correlate alerts with device assets
* Track incident lifecycle and SLAs
* Provide audit and compliance trail
* Drive human remediation workflows

📌 ServiceNow is the **system of action**, not analytics.

---

## 10. Optional State Management

### Service

* **Amazon DynamoDB**

### What it stores

* Device last-seen timestamps
* Open incident references
* Deduplication keys
* Alert lifecycle state
* Severity history

### Why it’s optional

* Used only when idempotency and replay safety are required
* Keeps orchestration reliable during retries

---

## 11. Batch Processing & Analytics

### Services

* **AWS Glue (Batch ETL)**
* **AWS Glue Data Catalog**
* **Amazon S3 – Analytics Zone (Iceberg)**

### Responsibilities

* Cleanse and enrich historical data
* Manage schema evolution
* Prepare analytics-ready datasets

---

## 12. Querying & Reporting

### Services

* **AWS Athena**
* **Power BI**

### What this enables

* Ad-hoc SQL analytics
* Incident history analysis
* Device last-seen reporting
* SLA and trend reporting
* Cost-efficient, serverless analytics

---

## 13. Observability & Monitoring

### 13.1 CloudWatch Logs

* Centralized logs from:

  * EKS
  * MSK
  * Glue
  * Lambda
  * SQS
* Used for debugging and audits

---

### 13.2 Amazon S3 – Centralized Log Archive

* Long-term log retention
* Compliance and forensic analysis
* Source for Athena queries

---

### 13.3 CloudWatch Metrics

* Ingestion rate and latency
* Error and retry counts
* Pipeline health monitoring

---

### 13.4 Amazon Managed Grafana

* Visualizes CloudWatch metrics
* Real-time operational dashboards
* Single pane of glass for NOC teams

---

## 14. Security & Secrets Management

### Services

* **AWS Secrets Manager**
* **IAM**
* **Encryption (in transit & at rest)**

### Key controls

* Device authentication at ingestion
* Least-privilege IAM roles
* Centralized secret storage
* Immutable audit data

---

## 15. Failure Handling & Resilience (Summary)

* Kafka absorbs ingestion spikes
* Glue Streaming handles backpressure
* EventBridge retries and DLQs
* Lambda isolates ServiceNow failures
* SQS buffers traffic spikes
* DynamoDB ensures idempotency
* No direct dependency between analytics and ITSM

---

## 16. Key Design Principles

* Event-driven
* Loose coupling
* Single responsibility per service
* Managed services over self-managed
* Audit and compliance ready
* Scalable by design

---

## 17. Open Items (Next Phase)

* SLIs and SLO definitions
* Cost optimization thresholds
* DR and multi-region strategy
* Rollout and migration plan

