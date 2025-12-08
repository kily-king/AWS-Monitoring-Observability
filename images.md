Hello



<img width="1536" height="1024" alt="image-2" src="https://github.com/user-attachments/assets/bc7314ba-8d2c-4cc4-a464-6a40ba997996" />


<img width="1536" height="1024" alt="image-1" src="https://github.com/user-attachments/assets/508e21ea-17c4-4648-91bb-2c0c1d636e38" />


<img width="536" height="405" alt="Image-3" src="https://github.com/user-attachments/assets/4466b562-114f-4d9d-8c86-cf9fddc01715" />


<img width="423" height="461" alt="Image-4" src="https://github.com/user-attachments/assets/242cee99-d77e-4b2b-b61e-aad07f5fadcd" />


<img width="1536" height="1024" alt="Image-5" src="https://github.com/user-attachments/assets/c884af73-b400-4ddb-a15e-b30f8d5d5e8f" />



## **Observability Architecture**

1. Source Accounts (Resource Layer)
• The source accounts host the following workloads:
  ◦ API Gateway, Amazon EC2 instances with monitoring agent, databases, EKS clusters, and Amazon MSK.
• The EKS clusters are configured with AWS Distro for OpenTelemetry (ADOT) to export:
  ◦ JSON logs, metrics, and traces to:
    ▪ Amazon CloudWatch (logs and metrics)
    ▪ AWS X-Ray (distributed traces)
2. Shared Account – Centralized Observability
• Centralized, managed observability services:
  ◦ Amazon Managed Grafana, deployed in a private subnet, currently using AWS SSO for authentication and adding to Active Directory.
  ◦ Amazon Managed Prometheus as the central metrics store.
• Grafana queries:
  ◦ Amazon Managed Prometheus via PrivateLink
  ◦ CloudWatch Logs, Metrics, and X-Ray Traces from source accounts, also via PrivateLink
• CloudWatch and X-Ray remain in their respective source accounts (not centralized) and are accessed cross-account through Grafana's native integrations.
• Grafana dashboards and data sources are visible only to users who have been granted the appropriate permissions and role assignments.
3. Alerting and ServiceNow Integration
• Alarms are configured within the observability stack and are routed to Amazon SNS topics.
• When triggered, the SNS topic invokes a Lambda function, which formats the alert and sends a payload to ServiceNow.
• The support team receives the alert in ServiceNow and proceeds with the predefined incident response workflow.
4. IAM Permissions – Cross-Account via StackSet
• IAM roles needed for cross-account observability are provisioned using CloudFormation StackSets, managed via the AFT (Account Factory for Terraform) management account.
• StackSets are applied to specific Organizational Units (OUs) depending on the environment (production or non-production).
• After deployment, all account IDs that received the roles are stored in SSM Parameter Store, which is shared with the central observability account.
• This shared data is used in Terraform to generate trusted IAM roles in the shared account, enabling automated trust relationships with source accounts.
• This approach removes the need for manual updates every time a new account is added.

Device Monitoring – Workload Extension
5. Heartbeat Ingestion from Field Devices
• Field devices send HTTPS heartbeat messages to an Event Collector application.
• This application is deployed in EKS cluster.
• The Event Collector enriches the message.
6. Publishing to Kafka (MSK)
• The enriched messages are published to Kafka topics within the Amazon MSK cluster.
• MSK acts as the message bus for both real-time and historical event ingestion.
7. Stream Processing and Time-Series Storage
• An Amazon MSK Connect connector subscribes to the Kafka topics and writes the data to Amazon Timestream.
• Amazon Timestream is used for time-series optimized storage of heartbeat and event data:
  ◦ Supports long-term retention and real-time querying.
  ◦ Can optionally offload historical data to Amazon S3 for cold storage or analytics.
  ◦ Supports SQL queries.
8. Visualization and Reporting
• Amazon Timestream is connected to:
  ◦ Amazon Managed Grafana for real-time operational dashboards used by the operations team.
  ◦ IT can connect with Power BI for customer-facing reports and analysis dashboards via ODBC.



----

#### Lucid Arc

Final Architecture: Multi-Account Observability + Device Monitoring

Swimlane: AWS Management Account (AFT)
CloudFormation StackSets -> Cross-Account IAM Roles (Source Accounts)
CloudFormation StackSets -> Cross-Account IAM Roles (Shared Observability Account)
Cross-Account IAM Roles (Source Accounts) -> SSM Parameter Store (List of Accounts)
SSM Parameter Store (List of Accounts) -> Terraform in Shared Observability Account

Swimlane: Source Accounts (Workloads + Devices)
Field Devices -> Event Collector (EKS)
Event Collector (EKS) -> Amazon MSK (Kafka Topics)

Amazon MSK (Kafka Topics) -> MSK Connect (Timestream Sink)
MSK Connect (Timestream Sink) -> Amazon Timestream

Amazon MSK (Kafka Topics) -> MSK Connect (JDBC Sink to ODS)   # optional
MSK Connect (JDBC Sink to ODS) -> ODS Staging Tables          # optional

EKS Workloads -> ADOT (AWS Distro for OpenTelemetry)
ADOT (AWS Distro for OpenTelemetry) -> Amazon CloudWatch Logs
ADOT (AWS Distro for OpenTelemetry) -> Amazon CloudWatch Metrics
ADOT (AWS Distro for OpenTelemetry) -> AWS X-Ray
ADOT (AWS Distro for OpenTelemetry) -> Amazon Managed Prometheus (remote write)

API Gateway -> Amazon CloudWatch Metrics
EC2 Instances -> Amazon CloudWatch Metrics
Databases -> Amazon CloudWatch Metrics

Swimlane: Shared Observability Account
Amazon CloudWatch Logs -> Amazon Managed Grafana (via PrivateLink)
Amazon CloudWatch Metrics -> Amazon Managed Grafana (via PrivateLink)
AWS X-Ray -> Amazon Managed Grafana (via PrivateLink)
Amazon Managed Prometheus -> Amazon Managed Grafana (via PrivateLink)
Amazon Timestream -> Amazon Managed Grafana (data source)

Amazon Managed Grafana -> Ops Dashboards
Ops Dashboards -> Ops Team

Amazon Timestream -> Power BI (via ODBC)
Power BI (via ODBC) -> Customer-Facing Reports

Alert Rules (CloudWatch / Prometheus / Grafana) -> Amazon SNS (Alert Topics)
Amazon SNS (Alert Topics) -> AWS Lambda (ServiceNow Alert Handler)
AWS Lambda (ServiceNow Alert Handler) -> ServiceNow Incidents

Swimlane: External Systems
ServiceNow Incidents -> Support Team



-----





