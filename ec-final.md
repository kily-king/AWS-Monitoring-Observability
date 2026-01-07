

.
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
