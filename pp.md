Device Monitoring Requirements: 
- Stream device heartbeat data to AWS TimeStream 
- Implement predictive maintenance alerting 
- Create device trend and pattern analysis 
- Replace Azure Event Grid with AWS native services 

Technical Stack: 
- Event Collector → Kafka → TimeStream 
- SQL-based trend analysis 
- Notification and alerting system 
- Device health prediction algorithms 

Current Status: 
- Terraform modules are being created 
- Paused pending business requirements clarification 

Management requested halt for priority reassessment Today Event Collector is reading from Kafka, but the reporting is all done via Azure Event Hub. This all needs to change to a new pattern that is not tightly coupled to Azure services, while also being mindful of the value this data has within the system. IT Ops needs to see these device events to ensure that work is being done before a device failure. ODS also need to make use of this data for reporting.

MVP:

Replicate the logic that occurs in Event Hub into an AWS pattern to land SQL based insights into Service Now
Replace Apache NiFi to land device events/heartbeat data in current state as-is to ODS staging tables
Post go-live: Need to figure out what we're doing long term with the functionality that SolarWinds provides today



EventHub
---------------------

Field Device Incidents
• Field Devices send their events to Event Collector
• Azure Event Hubs ingests the raw events, applies business rules, and raises incidents where applicable
• As part of the event flow the devices operational statuses are updated to provide a single pane of device map overview.

Field Device Event Flow
- Ingest: Device Events/Heartbeats are received by Event Collector and mirrored to EventHub via Apache Kafka MirrorMaker. Topics are replicated in the EventHub namespace for consumption by Stream Analytics (alerts) and PowerBI (dashboards).
- Transform: Events are evaluated by a set of Stream Analytics jobs (TSQL Queries) to determine if they have met the prescribed threshold, if they have not no action is taken, and the events are reevaluated until there is a clear event or the threshold has been reached. Events that have met the thresholds are written into new EventHub namespace topics for consumption by Logic App. Logic App in turn formats the event json into a common schema understood by ServiceNow for the HTTPS post.
- Post: The Logic App calls to the required ServiceNow API endpoint to either open an incident, append to an already open incident (device ID, serial number and event ID must match to append to existing incidents), or update the device service status on the ServiceNow Asset Map

Ingest
Events are received by the Event Collector(1) application, forwarded to their respective Kafka queues (2), and finally mirrored to the EventHub (3) namespace for transformation.
There are redundant Kafka nodes as well as topic replication to ensure high availability of the service.

Ingest Behind the Scenes
• Ingestion of events by EventHub is made possible by utilizing Kafka Mirror Maker
• Both source and target clusters are defined as well as SASL authentication
• Mirror Maker should have all source nodes defined with only 1 running instance

Transform
During the transform phase incoming messages are ingested by Stream Analytics where a series to T-SQL queries evaluates them for any conditions that require an alert. Alerts are based on device type, event type and have either time-based rules that wait for a condition to be set for X minutes before raising an incident, or aggregation-based rules that fire based on a count of events against a particular device. 

Time based rules allow the device time to clear events before raising, this will help to account for daily maintenance activities, scheduled reboots, end of day activities, etc.


Transform (Queries)
• Stream Analytics transforms incoming data into actionable incidents based on program needs
• Queries are written in T-SQL with the caveat that not all standard SQL options are usable
• Queries can be based on time delays or aggregation (count based)


Transform Input/Output
• Transform read from the incoming raw data stream
• Transform then writes into a new topic for filtered events
• The filtered topic(s) are read by Logic App

Post
All events that are flagged by the Transform stage are then picked up by Logic App to do the post. Logic App collates the event data into a common JSON schema so that it can be posted to ServiceNow via an HTTPS post. The ServiceNow API will then check if there is an existing incident already open for the same device and event, if so it appends the existing incident, it not it creates a new incident


Post
• In LogicApp you need to define the from Topic, incoming JSON schema, outgoing JSON schema and HTTPS connection information
• In the event that more than 175 events happen at roughly the same time ‘For Each’ loops split the incoming streams

Post (HTTPS Setup)
• You must define:
• Method as POST
• Provide the URL
• JSON Body
• Authentication method

ServiceNow Monitoring API
 The SeviceNow API was written for EventHub specifically
• The required values must always be provided, conditional values are nice to have
• If existing incident is open for the same device/event a new incident is not created

Result
The result of the HTTPS post is the creation or appending of a ServiceNow incident. When a new incident is created the occurred time is synced with the first event that caused the failure mode.
Occurred time of the incident is set to the timestamp of the first event


Build Out 
• EventHub build can be done entirely manually or partially automated (first time I recommend manual)

You will need
• Access to project Kafka nodes
• Access to Azure subscription
• Ability to create resource groups/resources in Azure
• Ruleset from the program
• ServiceNow API user


Getting the data into EventHub (MirrorMaker)
• Configuration is done on ONE of the Kafka nodes in the environment***
• There are two configuration files source and target
• Kafka must have the ability to reach the Azure broker on port 9093
• Kafka access policy must exist on the namespace in Azure


EventHub Monitoring 
• EventHub has several components that all work in tandem to produce incidents in ServiceNow
• Event Streams, Logic App Runs, Stream Analytics Jobs, and the Data Warehouse (where applicable) should all be monitored
• Alerts should be sent into ServiceNow as with other systems


Events Flow
• The flow of events is critical to all other Event Hub operations, without uninterrupted flow incidents will not be created.
• Should issues occur operations should check Kafka, ZooKeeper and Azure health


Stream Analytics Jobs
• Stream Analytics jobs are responsible for determining if events are actionable and must remain in a running state at all times
• Any critical/error condition will cause the job to fail and requires immediate investigation/resolution
• Errors could be due to Azure outages, data serialization, JSON formatting etc.


Logic Apps 
• Logic Apps send HTTPS API calls to ServiceNow to create incidents.
• High number of failures can be an indicator of potential issues
• Runs history can be used to check why each run failed including any applicable ServiceNow error codes


Directing Alerts to Logic App
• Alerts are sent via Logic Apps to ServiceNow using the table API and HTTPS post
• Logic App must exist before creating the action group
• Logic App can be tested to confirm operation



Stream Analytics Properties 
• Input(s) – Used to pull in data from one or more Kafka topics
• Output(s) – Used to write events out to a new topic (within the Azure namespace)
• Function(s) – Functions can be used to run JavaScript code such as retrieving current time

Stream Analytics SAQL
• Stream Analytics Query Language is a subset of TSQL syntax
• Designed to process high volumes of json/avro/csv formatted data
• SAQL supports many of the common TSQL functions


Common Query Patterns
• Delay Timer style queries - Delay timer queries keep track of device event set and clear conditions only firing if an event has been set and not cleared within the timer window
• Aggregation style queries - Aggregation queries are based on the count of an event over time, it can look for reoccurrences such as 10 times in 30 minutes or ensure that an event has occurred at least once in 24 hours.
• Asset Status queries - Asset status queries are used to set a device’s operational status on the ServiceNow device map
• Asset Mode queries - Asset mode queries are used to set a device’s mode on the ServiceNow map, modes examples are entry, exit, entry/exit


Main Parts of a Query
• Select: Used to build the JSON payload that is required by ServiceNow
• Into: Your filtered data
• From: Your source data
• Where: Conditions to trigger the alert
• Group By: Used in Aggregation style queries only 
• Having: Used in Aggregation style queries only


Where Clause by Device 
Type
• In an effort to make query 
management easier where clauses are 
separated by device type using the 
first 3 letters as the identifier
• Commissioning IDs are excluded 
• Event codes relevant to the 
timer/aggregation rule are included
• Time wait, or count of events are 
included


Delay Timer Query Detailed Review 
• Normalize Data: This first step sanitizes the data ensure a consistent data set
• Find Set Events: Find all events where the setOrClear flag = 1** • Find Clear Events: Find all events where the setOrClear flag = 0**
• Join Set and Clears:
• Find qualifying events (Where clause)


Delay Timer Query Detailed Review Continued
• Normalize Data: This first step sanitizes the data ensure a consistent data set
• Find Set Events: Find all events where the setOrClear flag = 1** 
• Find Clear Events: Find all events where the setOrClear flag = 0**
• Join Set and Clears:
• Find qualifying events (Where clause




