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