# CloudFormation
AWS CloudFormation allows you to provision AWS infrastructure via a template JSON/YML file.

The main sections in a CloudFormation template:
* Parameters - specify variables requiring user input
* Conditions - define entities based on a condition e.g. provision resources based on environment, or specify AMI for and EC2 instance to deploy based on region
* Resources - the AWS resources to create
* Mappings - create custom mappings e.g. RegionMap for Region : AMI
* Transforms - reference code located in S3 e.g. Lambda code or reusable snipets of CloudFormation code

# Working with Stacks
## Nested Stacks
Allows you to re-use your CloudFormation code. Upload the template to S3 and it can be referenced in the Resources section in a CloudFormation template using the **Stack** resource type.

# Resource Type Reference
## AWS::Lambda::Function
Creates a Lambda function. Requires a *deployment package* (contains the function code) and an *execution role* (grants the function permission to use AWS services).

Under the`AWS::Lambda::Function` resource, there is a `Code` property which contains the deployment package for a Lambda function. For all runtimes, this can be specified as the location of an object in S3 (as a .zip file). For Node.js and Python functions, you can specify the function code inline in the template under the `Code.Zipfile` property (limit 4096 characters).

![](https://udemy-images.s3.amazonaws.com/redactor/raw/2019-06-16_13-40-22-da5b60140125634d546815752f88b63c.png)

# SAM (Serverless Application Model)
Allows you to define and provision serverless applciations using CloudFormation. The AWS SAM CLI lets you locally build, test, and debug serverless applications that are defined by AWS SAM templates. The CLI provides a Lambda-like execution environment locally.

Uses the SAM CLI commands to package and deploy:
* `samp package` - packages your application and uploads to S3
* `sam deploy` - deploys your serverless app using CloudFormation

# CodeDeploy Integration
SAM comes built-in with CodeDeploy to ensure safe Lambda deployments. There are various deployment types to choose from: *Canary*, *Linear*, and *All-at-once*.

* **Canary** - __Traffic is shifted in 2 increments__. Takes the form `CanaryXPercentYMinutes` i.e. Shift X% of the traffic **immediately**, then after Y minutes shift all the remaining traffic.
* **Linear** - __Traffic is shifted in equal increments between intervals of equal duration__. Takes the form `LinearXPercentEveryYMinutes` i.e. shift X% of the traffic every Y minutes.
* **All-at-once** - All traffic is shifted to the updated function version at once.
