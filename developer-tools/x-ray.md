# X-Ray Overview
AWS X-Ray provides an end-to-end visual map of the flow of requests within serverless applications. It aggregates the data gathered from individual services in the application into a single unit called a trace. You can use this trace to follow the path of an individual request as it passes through each service or tier in your application so that you can pinpoint where issues are occurring.

![](https://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-servicemap-lambda-node.png)

AWS X-Ray integrates with many AWS serverless services like DynamoDB, Lambda, API Gateway, etc. You can also instrument your own applications (running on EC2, Elastic Beanstalk environments, on-prem systems and EC2) to send data to X-Ray.

# Concepts
AWS X-Ray receives data from services as *segments*. X-Ray then groups segments that have a common request into *traces*. X-Ray processes the traces to generate a *service graph* that provides a visual representation of your application.

## Segments
A *segment* is a JSON representation of a request that your application serves. A trace segment records information about the original request, information about the work that your application does locally, and subsegments with information about downstream calls that your application makes to AWS resources, HTTP APIs, and SQL databases. 

![](https://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-segment-overview.png)

## Subsegments
Subsegments provide more granular timing information and details about downstream calls that your application made to fulfill the original request. 

![](https://docs.aws.amazon.com/xray/latest/devguide/images/scorekeep-PUTrules-timeline-subsegments.png)

For services that don't send their own segments, like Amazon DynamoDB, X-Ray uses subsegments to generate inferred segments and downstream nodes on the service map. This lets you see all of your downstream dependencies, even if they don't support tracing, or are external.

## Sampling
The X-Ray SDK applies a sampling algorithm to determine which requests get traced. It consists of a *reservoir* size which is the fixed number of requests recorded per second, and a *rate* which is the percentage of remaining requests to record per second after the reservoir has been exhausted.

Sampling formula:
```
reservoir size + ( (incoming requests per second - reservoir size) * fixed rate)
```

If an application has been configured with a reservoir size of 50 and a fixed rate of 10% to sample 100 incoming requests per second, then the total sampled requests per second is 55.

# X-Ray API
The X-Ray API provides access to all X-Ray functionality through the AWS SDK, AWS Command Line Interface, or directly over HTTPS.

## Segment Documents
A *segment document* conveys information about a segment to X-Ray. A segment document can be up to 64 kB and contain a whole segment with subsegments, a fragment of a segment that indicates that a request is in progress, or a single subsegment that is sent separately. You can send segment documents directly to X-Ray by using the `PutTraceSegments` API.

X-Ray compiles and processes segment documents to generate queryable trace summaries and full traces that you can access by using the `GetTraceSummaries` and `BatchGetTraces` APIs, respectively.

# X-Ray Daemon
The AWS X-Ray daemon is a software application that listens for traffic on UDP port 2000, gathers raw segment data, and relays it to the AWS X-Ray API.

## Configuration
AWS X-Ray requires both the AWS X-Ray SDK and the X-Ray Daemon installed on your systems. The X-Ray SDK sends data (e.g. data about incoming and outgoing HTTP requests made to a Java application) to the X-Ray daemon which buffers segments in a queue and uploads them to X-Ray in batches.

## On-Prem & EC2 Instances
Install the X-Ray daemon on your EC2 instance or on-prem server.

On an EC2 instance, use a user data script to automatically run the X-Ray daemon upon launching the instance:
```
#!/bin/bash
curl https://s3.dualstack.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.rpm -o /home/ec2-user/xray.rpm
yum install -y /home/ec2-user/xray.rpm
```

## Elastic Beanstalk
Enable the X-Ray daemon by including the `xray-daemon.config` configuration file in the `.ebextensions` directory of your source code.

**.ebextensions/xray-daemon.config:**
```
option_settings:
  aws:elasticbeanstalk:xray:
    XRayEnabled: true
```

## Elastic Container Service
Install the X-Ray daemon in its own Docker container on your ECS cluster alongside your app. Configure the port mappings and network mode settings in your task definition file to allow traffic on UDP port 2000.
