# Virtual Private Cloud (VPC) Overview
Create a virtual network in the cloud dedicated to your AWS account where you can launch AWS resources. A VPC spans all the AZs in a region, and each AZ can hold multiple subnets.

# Key Concepts
* A VPC lets you specify the VPC's IP address range, add subnets, associate security groups, and configure route tables.
* A *subnet* is a range of IP addresses within the VPC. Use a *public subnet* for resources that must be connected to the internet, and a *private subnet* for resources that don't.

# Security
## Security Groups
A *security group* is a virtual firewall to control inbound and outbound traffic to an instance. Each security group rule consists of a source/destination for the traffic, and the port/port range.

## VPC Flow Logs
*VPC Flow Logs* is a feature that enables capturing information about the IP traffic to and from network interfaces in your VPC. Flow log data can be published to CloudWatch and S3.
