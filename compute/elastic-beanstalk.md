# Elastic Beanstalk
Elastic Beanstalk is a service for deploying and scaling web applications, and handles the provisioning of the underlying resources needed e.g. S3, ELB, EC2 with Auto-Scaling.

**Languages Supported**: Java, PHP, Python, Ruby, Go, Docker, .NET, Node.js

# Concepts
## Application
An *application* is a collection of *environments*, *versions* and *environment configurations*.

## Application Version
An *application version* is a labeled iteration of deployable code for a web application (stored in S3).

## Environment
An *environment* is a collection of AWS resources running an application version e.g. EC2 instances with Auto-Scaling behind an ELB. Each environment runs only one application version at a time.

## Environment Configuration
An *environment configuration* is a collection of parameters and settings that define how an environment behaves.

# Using Elastic Beanstalk
1. Create an application
2. Upload an application version source bundle (e.g. a Java .war file)
3. Elastic Beanstalk automatically launches an environment to run the uploaded application version
4. You can now manage your environment, and deploy new application versions.

![](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/clearbox-flow-00.png)

# Environment Configuration
## Adding an RDS instance
There are 2 options for adding an RDS instances to an Elastic Beanstalk environment:
* Launch within Elastic Beanstalk
  * The RDS instances lives and dies (terminates) with the environment.
  * Suitable for dev/test environments.
  * Don't even think about using this in production.
* Launch outside of Elastic Beanstalk
  * Launch the RDS instance independently then connect to the environment. Additional configuration steps required - security group & connection info.
  * Suitable for production environments.

## Advanced Configuration
You can add Elastic Beanstalk configuration files to a web application's source code to customise the AWS resources that it contains. Configuration files are `YAML`/`JSON`-formatted with a `.config` extension, placed in `/.ebextensions` located in the top level directory of the application source code bundle.

## Worker Environments
You can offload long-running tasks to a dedicated *worker environment* to ensure the application stays responsive under load. Each instance runs a daemon process that reads from an SQS queue managed by Elastic Beanstalk. The daemon is responsible for pulling requests from an Amazon SQS queue and then sending the data to the web application running in the worker environment tier that will process those messages. If you have multiple instances in your worker environment tier, each instance has its own daemon, but they all read from the same Amazon SQS queue. The application running on the worker instance will then asynchronously process the long-running task.

![](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-architecture_worker.png)

You can define periodic tasks in a file named cron.yaml in your source bundle to add jobs to your worker environment's queue automatically at a regular interval.

**Example cron.yaml**
```
version: 1
cron:
 - name: "backup-job"
   url: "/backup"
   schedule: "0 */12 * * *"
 - name: "audit"
   url: "/audit"
   schedule: "0 23 * * *"
```

# Managing Environments
## Deployments
Elastic Beanstalk supports the several options for processing deployments.

### All at once
Deploy the new version to all instances simultaneously. All instances in the environment are out of services during deployment.

### Rolling
Deploy the new version in batches (of instances). Each batch is taken out of service during deployment, while the remaining batches stay online. Reduced capacity during deployment as there are less instances running at once.

### Rolling with additional batch
Deploy the new version in batches, but first launch a new batch of instances to ensure full capacity during deployment.

### Immutable
Deploy the new version to a fresh group of instances. Maintains full capacity, preferred option for mission critical production systems.

## Platform Updates
Whenever new platform versions are released, Elastic Beanstalk recommends one of two methods for updating:
* **Update your Environment's Platform Version** - recommended when updating to the latest platform version **without** a change in runtime, web server, or application versions, and without a change in the major platform version.
  1. Overview -> Platform -> Change
![](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-env-dashboard-changetonewplatform.png)
  2. Choose the Platform Version.
![](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/images/aeb-env-updateplatform-rollingon.png)
* **Perform a Blue/Green Deployment** - recommended when updating to a different runtime, web server, or application server versions, or to a different major platform version. E.g. updating from *Python 2.7* to *Python 3.6*, or updating from *Java 7* to *Java 8*.
  1. Create a new environment using the new target platform version and deploy the package to it.
  2. Iteratively test and fix any compatibility issues and ensure any customisations made using configuration files work correctly in the new environment.
  3. Swap CNAMEs between the old and new production environments.
  4. Terminate the old production environment.

# The Elastic Beanstalk CLI
Provides a simple CLI for interacting with EB environments. All commands start with the `eb` prefix.

Basic commands:
* `eb create`
* `eb status`
* `eb health`
* `eb events`
* `eb logs`
* `eb open`
* `eb deploy`
* `eb config`
* `eb terminate`
