# Continuous Integration & Continuous Delivery/Deployment with AWS

*Continuous Integration* is about integrating/merging code changes frequently (at least once per day). This page serves as an all-in-one reference for all the services provided by AWS that facilitate a CI/CD workflow.

*Continous Delivery* is about automating the **build**, **test**, and **deployment** functions.

*Continous Deployment* fully automates the entire release process, triggering deployment into production whenever new code gets uploaded and passes through the release pipeline.

AWS offers services for all stages in a CI/CD pipeline:

* **AWS CodeCommit** - Git Repository
* **AWS CodeBuild** - Compile source code, run tests and package code
* **AWS CodeDeploy** - Automate deployment to EC2, on-prem systems and Lambda
* **AWS CodePipeline** - Full CI/CD workflow tool

## AWS CodeCommit
Fully-managed source control service that hosts Git-based respositories.

## AWS CodeDeploy
Fully-managed automated deployment service.

AWS CodeDeploy supports two different types of deployment:
* **In-Place (Rolling) Update** - The application is stopped on each instance and the latest revision installed. EC2 & on-prem systems only.
* **Blue / Green** - New instances are provisioned for the new revision to be installed on. Traffic gets routed to the new instances. Supported for EC2, on-prem systems, and Lambda.

### AppSpec.yml
The `appspec.yml` file defines all the parameters needed for the deployment. Written in YAML, but Lambda supports JSON as well.

#### Lambda
**For Lambda deployments**, appspec.yml can be written in YAML or JSON and contains the following fields:
* `version`: Reserved for future use. Currently the only allowed value is `0.0`.
* `resources`: The name & properties of the Lambda function to deploy
* `hooks`: Specifies Lambda functions to run at set points in the deployment lifecycle e.g. validation tests to run before allowing traffic to be sent to newly deployed instances

The following hooks are available for use:
* **BeforeAllowTraffic**: The functions to run before traffic is routed to the newly deployed Lambda function. e.g. test to validate that the function has been deployed correctly.
* **AfterAllowTraffic**: The functions to run after the traffic is routed to the newly deployed Lambda function. e.g. test to validate that the function is accepting traffic correctly and behaving as expected. 

Example `appspec.yml` File for Lambda
```yml
version: 0.0
resources:
  - myLambdaFunction:
    Type: AWS::Lambda::Function
    Properties:
      Name: "myLambdaFunction"
      Alias: "myLambdaFunctionAlias"
      CurrentVersion: "1"
      TargetVersion: "2"
hooks:
  - BeforeAllowTraffic: "FunctionToValidateBeforeTrafficShift"
  - AfterAllowTraffic: "FunctionToValidateAfterTrafficShift"
```

#### EC2 & On-Prem
**For EC2 / On-Prem systems**, appspec.yml must be placed in the root directory of your revision. It can be written in YAML only and contains the following fields:
* `version`: Reserved for future use. Currently the only allowed value is `0.0`.
* `os`: The OS version you are using e.g. `linux`, `windows`
* `files`: The source and destination folders for files that need to be copied.
* `hooks`: Specify scripts that need to run at set points in the deployment lifecycle e.g. to unzip the application files prior to deployment, run functional tests on the newly deployed application, and to de-register and re-register instances with a load balancer.

The run order for hooks in a CodeDeploy deployment:
1. **BeforeBlockTraffic** -> **BlockTraffic** -> **AfterBlockTraffic**
2. **ApplicationStop**
3. **BeforeInstall** -> **Install** -> **AfterInstall**
4. **ApplicationStart**
5. **ValidateService**
6. **BeforeAllowTraffic** -> **AllowTraffic** -> **AfterAllowTraffic**

Example `appspec.yml` File for EC2
```yml
version: 0.0
os: linux
files:
  - source: Config/config.txt
    destination: /webapps/Config
  - source: Source
    destination: /webapps/myApp
hooks:
  BeforeInstall:
    - location: Scripts/UnzipResourceBundle.sh
    - location: Scripts/UnzipDataBundle.sh
  AfterInstall:
    - location: Scripts/RunResourceTests.sh
      timeout: 180
```

### CodeDeploy Agent
The CodeDeploy agent is a software package that, when installed and configured on an instance, makes it possible for that instance to be used in CodeDeploy deployments. The CodeDeploy agent communicates outbound using **HTTPS over port 443**. [GitHub Repo.](https://github.com/aws/aws-codedeploy-agent)

## AWS CodePipeline
CI/CD workflow tool allowing you to model and configure the different stages of a software release process. Can be configured to automatically trigger your pipeline as soon as a change is detected in the source code repo.

![](https://d1.awsstatic.com/Projects/CICD%20Pipeline/setup-cicd-pipeline2.5cefde1406fa6787d9d3c38ae6ba3a53e8df3be8.png)
