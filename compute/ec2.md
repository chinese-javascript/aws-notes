# Elastic Compute Cloud
## Overview
EC2 essentially provides a resizable VM in the cloud, which are backed by a root volume in Amazon EBS.

EC2 instances can be tied to one or more AZs in the same region, and are region-locked unless specified (incurs additional charges).

## Launching an EC2 Instance
### 1. Select the AMI to launch the instance from
An Amazon Machine Image (AMI) is a template instance that provides the configuration info required to launch an instance. It includes the following:
* A template for the root volume of the instance (can be EBS snaphots, an OS, or an application server)
* Launch permissions that control which AWS accounts can use the AMI
* A block device mapping that specifies the volumes to attach to the instance on launch

You can create your own AMI, search for community-made AMIs or select from Amazon's premade AMIs.

### 2. Select the instance type
Instance types are optimized to fit different use cases. They provide varying combinations of CPU, memory, storage, and networking capacity.

### 3. Secure the instance with a key pair
A key pair comprises a public key (stored by AWS) and a private key (stored locally on your machine). Accessing an instance requires providing the private key associated with the public key that was chosen for the instance.

It is recommended to store key pairs in the .ssh sub-directory.

Windows location: `C:\user\{username}\.ssh\*.pem`

Mac/Linux location: `~/.ssh/*.pem`

## Connecting to an Instance
Connect using the `ssh` command. The default user is `ec2-user`, and the instance IP address can be found in the AWS console.
```
ssh -i {full path of your .pem file} ec2-user@{instance IP address}
```

## Pricing Models
* **On Demand Instances**: Fixed hourly rate
  * Suitable for apps with unpredictable workloads that cannot be interrupted
* **Reserved Instances**: 1 or 3 year term contract
  * Suitable for apps with steady/predictable usage (e.g. web servers)
  * Upfront payments can further reduce the cost
    * **Convertible RIs** allow changing its attributes during operation
    * **Scheduled RIs** can be set to launch within specific time windows
  * After purchasing a Reserved Instance, the following can still be modified:
    * Availability zone
    * Instance type, but only within the same instance type family.
* **Spot Instances**: Bid a price to pay for an instance
  * For apps with flexible start & end times, or apps that are only feasible at very low compute prices
  * These will get terminated (with warning) once the usage price goes above your bid price
* **Dedicated Hosts**: A physical dedicated server
  * Useful for using existing server-bound software licenses or regulations that don't allow multi-tenancy

## Monitoring
To collect logs from your Amazon EC2 instances and on-premises servers into CloudWatch Logs, AWS offers two options:
* The unified CloudWatch agent - Supported on all operating systems. Also supports collecting custom metrics using StatsD or collectd.
* CloudWatch Logs Agent - **DEPRECATING**; only supported on Linux servers.
