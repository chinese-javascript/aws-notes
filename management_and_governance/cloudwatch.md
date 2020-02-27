# CloudWatch Overview
Amazon CloudWatch monitors your AWS resources and applications you run on AWS. With CloudWatch, you gain system-wide visibility into resource utilization, application performance, and operational health. AWS services publish metrics to CloudWatch, but you can also put custom metrics into the repository. 

![](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/images/CW-Overview.png)

The CloudWatch console aggregates metrics and displays statistics in a graphical format. CloudWatch also supports creating *alarms* which detect when a threshold is breached, and send out *notifications* to automatically make changes to your monitored AWS resources.

# Concepts
## Namespaces
A *namespace* is an isolated container for CloudWatch metrics, typically used for distinguishing different applications' metrics so as to not mistakenly aggregate them together. AWS' own namespaces typically follow the naming convention of `AWS/service` e.g. EC2 pubslishes metrics to the `AWS/EC2` namespace.

## Metrics
*Metrics* represent a time-ordered set of datapoints that are published to CloudWatch. Custom metrics are supported, on top of the default metrics that get published automatically by AWS services.

## Alarms
An *alarm* watches a single metric over a time period, and performs specified actions based on the value of the metric relative to a threshold. When creating an alarm, select a period greater than or equal to the frequency of the metric to be monitored.

When you create an alarm, you specify three settings to enable CloudWatch to evaluate when to change the alarm state:

* **Period** - the length of time to evaluate the metric or expression to create each individual data point for an alarm.
* **Evaluation Period** - the number of the most recent periods to evaluate when determining alarm state.
* **Datapoints to Alarm** - the number of data points within the evaluation period that must be breaching to cause the alarm to go to the `ALARM` state.

![](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/images/alarm_graph.png)

# Using Metrics
By default, several services provide free metrics for resources (e.g. EC2 instances, EBS volumes). You can also enable detailed monitoring, or publish your own application metrics. **Metric data is kept for 15 months.** CloudWatch does not monitor the memory, swap, and disk space utilization of your instances. If you need to track these metrics, you can install custom CloudWatch monitoring scripts to your instances.

## Publishing Custom Metrics
Publish your own metrics via the AWS CLI or API, by calling `PutMetricData`. 

`aws cloudwatch put-metric-data --namespace "Usage Metrics" --metric-data file://metric.json`

CloudWatch stores data about a metric as a series of data points. Each data point has an associated time stamp. You can even publish an aggregated set of data points called a statistic set.

### High-Resolution Metrics
Each metric is either:
* **Standard resolution**, with 1m granularity
* **High resolution**, with 1s granularity. Can be retrieved with a period of 1s, 5s, 10s, 30s, or any multiple of 60s.

Metrics produced by AWS services are standard resolution by default. You can specify the resolution with custom metrics.
