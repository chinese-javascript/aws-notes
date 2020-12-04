# Lambda Overview
AWS Lambda is an event-driven serverless computing platform. It will run code provided by the user in response to events, and all the resources required when running a Lambda instance are automatically provisioned and managed by AWS. You can set an execution time to determine how long a function should run before timing out.

AWS Lambda currently supports the following languages for its function code:
* **NodeJS**
* **C#**
* **Java**
* **Python**
* **Go**

# Managing Functions
## Function Configuration
In the AWS Lambda resource model, you choose the amount of memory you want for your function, and are allocated proportional CPU power and other resources. 

## Version Control
Lambda allows publishing multiple versions of the same function. 
* The latest version is referred to as `$latest`.
* Once a version is published it is immutable.

![](https://udemy-images.s3.amazonaws.com/redactor/raw/2019-02-04_04-08-17-2af9e1dbd8f4b24dd1eeb3a57e80ef08.png)

Traffic can be split to different function versions (e.g. for Blue/Green testing). Traffic cannot be split to `$latest` directly, instead create an alias that points to $latest and split traffic to there.

If your application uses an alias to refer to $latest, make sure to update it when uploading the code as the alias will not automatically point to the new code.

### Traffic Shifting Using Aliases
You can configure an alias to shift traffic between 2 function versions based on percentage weights in the `CreateAlias` or `UpdateAlias` operations, by using the `routing-config` parameter.

The following example points 98% of invocation traffic to version 1 of a Lambda function, with the remaining 2% pointing to version 2.

`aws lambda create-alias --name MyAlias --function-name MyFunction --function-version 1 --routing-config AdditionalVersionWeights={"2"=0.02}`

To route all traffic to version 2, change the `function-version` value to point to version 2, and also reset the `routing-config` value.

`aws lambda update-alias --name MyAlias --function-name MyFunction --function-version 2 --routing-config AdditionalVersionWeights={}`

## Reserved Concurrency
AWS dynamically scales function execution in response to increased traffic, up to the concurrency limit. Default is 1000 concurrent executions per second within a region. Exceeding the limit returns a `429 TooManyRequestsException` error. **You can contact AWS support to raise the limit.**

*Reserved concurrency* guarantees a set number of concurrent executions are always available to a critical function. To configure reserved concurrency on a function, use the `put-function-concurrency` command.

`$ aws lambda put-function-concurrency --function-name my-function --reserved-concurrent-executions 100`

Take note that AWS Lambda will keep the unreserved concurrency pool at a **minimum of 100** so that functions without specific limits can still process requests. Hence, your maximum total concurrent execution limits across all functions is 900.

## Layers
You can configure your Lambda function to pull in additional code and content in the form of layers. A layer is a ZIP archive that contains libraries, a custom runtime, or other dependencies. This allows using libraries without including them in your deployment package.

![](https://d2908q01vomqb2.cloudfront.net/1b6453892473a467d07372d45eb05abc2031647a/2019/02/04/sam-layers-diag.png)

A function can use up to 5 layers at a time. The total unzipped size of the function and all layers can't exceed the unzipped deployment package size limit of 250 MB.

# Invoking Functions
Lambda functions can be directly invoked, *synchronously* or *asynchronously*, via the Lambda console, Lambda API, AWS SDK, and AWS CLI.

Lambda also integrates with other AWS services to directly invoke functions. Triggers that can invoke functions include:

* Certain resource lifecycle events
* Uploading data to S3
* API Gateway receiving a HTTP request
* CloudWatch events that run on a timer

## Synchronous Invocation
With *synchronous invocation*, you wait for the function to process the event and return a response. This is the default invocation mode when running the `invoke` command.

```$ aws lambda invoke --function-name my-function --payload '{ "key": "value" }' response.json```

## Asynchronous Invocation
With *asynchronous invocation*, Lambda queues the event for processing and returns a response immediately with only the statuscode. To invoke a function asynchronously, use the `--invocation-type Event` option when running the `invoke` command.

```$ aws lambda invoke --function-name my-function  --invocation-type Event --payload '{ "key": "value" }' response.json```

Lambda handles retries and can send failed events to a dead-letter queue.

## Event Source Mapping
Lambda can also be configured to read from a stream or queue to invoke a function, via an *event source mapping*. The following services are supported:

* Kinesis
* DynamoDB
* SQS


# Lambda Runtimes
When a Lambda function is invoked, an *execution context* is launched based on configuration settings you provided (e.g. memory, execution time). The execution context is a temporary runtime environment for initialising external dependencies (e.g. DB connections, HTTP endpoints). This enables "warm-starts" after the initial "cold-start". Once a Lambda function completes, the service freezes the execution context in anticipation for subsequent function invocations. This has the following implications:

* Objects declared outside the function's handler method remain initialised, e.g. DB connections.
* The `/tmp` directory can be used as temporary storage between invocations.
* Incomplete processes initiated by a previous invocation will resume.

# Monitoring
## Using CloudWatch Logs
AWS Lambda automatically monitors functions on your behalf, reporting metrics through Amazon CloudWatch. Logging statements in the function code will also be sent to CloudWatch Logs, under the log group `/aws/lambda/<function name>`.

## Using AWS X-Ray
Lambda uses environment variables to communicate with the X-Ray daemon and configure the X-Ray SDK.

* **_X_AMZN_TRACE_ID** - The tracing header, which includes the sampling decision, trace ID, and parent segment ID.
* **AWS_XRAY_CONTEXT_MISSING** - The behaviour in the event that a tracing header is not available. Defaults to `LOG_ERROR`.
* **AWS_XRAY_DAEMON_ADDRESS** - The X-Ray daemon's address in the format: `IP_ADDRESS:PORT`.

# Lambda and VPCs
To enable a Lambda function to access resources inside a private VPC, you need to provide the VPC config to the function which include:
* private subnet ID (recommended to select 2 subnets for high availability mode)
* security group ID

Also ensure that the Lambda function has a service execution role that allows the creation of ENIs.

Lambda will use the VPC information to set up Elastic Network Interfaces (ENIs) using an IP from the **private** subnet CIDR range. Therefore, if your Lambda function requires Internet access, configure a NAT instance or use the VPC NAT gateway.

# Common Errors
## Larger workloads are not being processed.
Increase the function's execution time.
