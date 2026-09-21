# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!


---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)

#Task 1
We are monitoring a payment service based on certain parameters trying to identify the actual problem if a request is timed out. The purpose of AIOPS is to automate the error detection work and to detect anomalies in the metrices to identify the reason behind failed services.

#Task 2
1)The numeric operational metrics are:
response_time_ms:Measures latency for each request.
cpu_percent:CPU utilization percentage.
memory_percent:Memory utilization percentage.
These are the fields that describe runtime health and system load. The service name is operational metadata not a metric.

2)The log-related fields are:

log_level:Values such as INFO and ERROR.
message:Human-readable event text, such as “Payment request processed successfully” or “Database connection timeout”.
These fields capture the semantic status of the system, while the numeric fields capture its performance.

3)Timestamps are ISO-8601 strings in one-minute intervals:

2026-09-20T10:00:00
2026-09-20T10:01:00
...
2026-09-20T10:09:00

They are used to:
order observations chronologically,correlate metric spikes with log events,identify whether degradation is brief or sustained over time and anchor anomaly detection to a specific event time.
The sequence shows a normal baseline, then a sharp degradation around 10:05 and 10:06, followed by recovery.

4)The normal-looking records are the ones with:
low response times
low CPU and memory use
INFO log level
success messages
Specifically, the normal windows are:
10:00 through 10:04
10:07 through 10:09
Examples:At 10:00: response_time_ms = 120, cpu_percent = 42, memory_percent = 51, log_level = INFO
At 10:04: response_time_ms = 130, cpu_percent = 46, memory_percent = 54, log_level = INFO
At 10:09: response_time_ms = 145, cpu_percent = 50, memory_percent = 57, log_level = INFO
These all match the expected steady-state behavior.

5)The unusual observations are:
10:05:00
response_time_ms = 610
cpu_percent = 75
memory_percent = 70
log_level = ERROR
message = “Payment service timeout”

10:06:00
response_time_ms = 640
cpu_percent = 94
memory_percent = 91
log_level = ERROR
message = “Database connection timeout”

#Task 3
A limitation of the current detection approach is that it is purely threshold-based and does not model time-series context or multi-signal severity beyond simple comparisons. A useful improvement would be to detect sustained abnormal patterns over consecutive samples and incorporate ERROR log severity more explicitly into the scoring logic.

#Task 4
Changed consumer_topic = EventTopic(consumer.name) to consumer_topic = EventTopic(producer_topic.name)       ;(in aiops_pipeline.py)
Edited for event in result['anomalies_detected']: (in aiops_pipeline.py)

The execution result is:
Records processed: 10
Anomalies detected: 2
The anomaly objects are successfully created and displayed
Event delivery through the same topic to the consumer is not fully complete in the current code because the producer and consumer are attached to different topic instances.

#Task 5
 if record["log_level"] == "ERROR":
            reasons.append("Error log detected")
remove consumer_topic and edit consumer
    #consumer_topic = EventTopic(producer_topic.name)

    consumer = EventConsumer(producer_topic)
#Task 6
run aiops_pipeline.py

#Task 7 



## 1. Scenario
This repository models a lightweight AIOps workflow for a payment service. The goal is to detect when a payment request times out, inspect the related operational data, and identify the abnormal behavior that explains the failure.

The workflow uses operational metrics and log records to detect anomalies and route them through an in-memory event pipeline.

## 2. Operational data
The operational dataset is stored in `service_data.json` and contains 10 records collected over a short period.

Each record includes:
- `timestamp`
- `service`
- `response_time_ms`
- `cpu_percent`
- `memory_percent`
- `log_level`
- `message`

The key operational fields are:
- `response_time_ms`: request latency
- `cpu_percent`: CPU utilization
- `memory_percent`: memory utilization
- `log_level`: severity such as INFO or ERROR
- `message`: human-readable log message

## 3. Observations from logs and metrics
### Normal behavior
The normal records show:
- low response time
- moderate CPU usage
- moderate memory usage
- INFO log entries
- successful messages like “Payment request processed successfully”

Examples:
- 2026-09-20T10:00:00 → response_time_ms = 120, cpu_percent = 42, memory_percent = 51, log_level = INFO
- 2026-09-20T10:04:00 → response_time_ms = 130, cpu_percent = 46, memory_percent = 54, log_level = INFO
- 2026-09-20T10:09:00 → response_time_ms = 145, cpu_percent = 50, memory_percent = 57, log_level = INFO

### Unusual behavior
The abnormal records are:
- 2026-09-20T10:05:00
  - response_time_ms = 610
  - cpu_percent = 75
  - memory_percent = 70
  - log_level = ERROR
  - message = “Payment service timeout”

- 2026-09-20T10:06:00
  - response_time_ms = 640
  - cpu_percent = 94
  - memory_percent = 91
  - log_level = ERROR
  - message = “Database connection timeout”

These values are clearly above the stable operating range and align with a service degradation or outage.

## 4. Anomaly-detection findings
The detection logic in `anomaly_detector.py` flags records that exceed thresholds for:
- response time > 500 ms
- CPU utilization > 80%
- memory utilization > 80%
- error-level log events

The detector identifies:
- 2026-09-20T10:05:00 → ANOMALY
- 2026-09-20T10:06:00 → ANOMALY

The resulting reasons include:
- High response time
- High CPU utilization
- High memory utilization
- Error log detected

## 5. Event-processing flow
The event flow is implemented using the project’s existing components:

- `EventProducer` publishes anomaly events
- `EventTopic` stores the in-memory event messages
- `EventConsumer` reads from the topic
- `Event/message` carries the anomaly payload

The flow works as:
1. Data is loaded from the operational dataset.
2. The anomaly detector evaluates each record.
3. A matching anomaly produces an event.
4. The producer sends the event to the topic.
5. The consumer reads the event from the topic.
6. The event reaches downstream AIOps logic for handling.

## 6. Final workflow execution result
Running the pipeline produced the following outcome:
- Records processed: 10
- Anomalies detected: 2

Sample output:
```
Service: payment-service
Timestamp: 2026-09-20T10:05:00
Type: ANOMALY
Reasons: High response time, Error log detected

Service: payment-service
Timestamp: 2026-09-20T10:06:00
Type: ANOMALY
Reasons: High response time, High CPU utilization, High memory utilization, Error log detected
```

This confirms that the system identified abnormal operational behavior and generated anomaly events.

## 7. Issues identified and corrected
Two issues were identified in the workflow and corrected without replacing the architecture:

1. Topic wiring problem
   - Affected component: `aiops_pipeline.py`
   - Cause: the producer and consumer were connected to different in-memory topic instances
   - Correction: the consumer was wired to the same topic the producer publishes to

2. Log evaluation issue
   - Affected component: `anomaly_detector.py`
   - Cause: anomaly reason generation did not correctly treat error-level logs as relevant evidence
   - Correction: error-level events were included as a proper anomaly signal

## 8. Limitation / improvement
A limitation of the current approach is that it uses fixed thresholds and simple rules. It works well for this dataset, but it does not model time-series trends or contextual severity across multiple observations.

A possible improvement would be:
- detect sustained abnormal behavior over multiple consecutive samples
- combine metric and log severity into a weighted anomaly score
- reduce false positives caused by one-off spikes

## 9. Reproduction steps
To reproduce the demonstration:

1. Open a terminal in the repository root.
2. Run:

```bash
cd /workspaces/github-skills-challenge
python src/aiops_pipeline.py
```

3. Review the output.
4. Confirm that the pipeline processes all operational records and reports the two anomaly events.
5. Optionally validate with:

```bash
pytest -q
```

This project demonstrates a minimal AIOps workflow in which operational data is analyzed, anomalies are detected, and the resulting events are passed through an in-memory event-stream pipeline.