- Vending Machine : it gathers events 
- Event Collector : Devices send telemetry and heartbeat data to an event collector.
- Kafka MirrorMaker : Mirroring Kafka topics from an on-prem or external Kafka cluster
		    (Kafka Connect Sink → Kinesis Data Streams instead of MirrorMaker if you want direct integration)
- Kinesis Data Streams: ingestion layer for both Device Events and Device Heartbeats
- Kinesis Data Analytics : its for real-time filtering, aggregation, anomaly detection
- AWS Step Functions (Logic Apps equivalent) : its a serverless orchestration service that uses visual workflows to coordinate the components of distributed applications and microservices
- Monitoring (Grafana): to visualize metrics from CloudWatch or Prometheus integrated with AWS.
- ServiceNow : Correct: Typically integrated via AWS Lambda or API Gateway for ticketing and incident management.






		Vending Machine
		    |
		Event Collector
		    |
		Kafka MirrorMaker
		    |
	Amazon Kinesis Data Streams
	|			|
Device Events		Device HeartBeat
	\			/
	 \		       /
	Amazon Kinesis Data Analytics
		    /\
		   /  \
		  /    \
	   Filtered   Filtered
	    Events     Heartbeats
		\	  /
		 \       /
	AWS Step Functions (Logic Apps)
		    |
		Monitoring (Grafana)
		    |
		ServiceNow

