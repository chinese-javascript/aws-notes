# Elastic Block Storage
EBS provides a virtual disk in the cloud that provides storage for EC2 instances. An EC2 instance may have multiple EBS volumes attached, with one of them being the boot volume.

EBS volumes are tied to a specific AZ in which they are auto-replicated

## EBS Volume Types
| Type | Speed | Notes |
| --- | --- | --- |
| General Purpose SSD (GP2) | 3 IOPS/GB, up to 10,000 IOPS | General purpose storage. |
| Provisioned IOPS SSD (IO1) | up to 20,000 IOPS | For I/O intensive apps such as large relational or NoSQL databases. Useful if you need more than 10,000 IOPS. |
| Throughout Optimized HDD (ST1) | Slow | Cheap HDD-based alternative that can support high-throughput workloads. For big data, data warehousing, log processing, etc. Cannot be a boot volume. |
| Cold HDD (SC1) | Slower | Has the lowest cost for storage. Good for infrequently accessed workloads (e.g. for a file server). Cannot be a boot volume. |
| Magnetic (Standard) | Slowest | Lowest cost of all EBS volume types that is bootable. Useful where lowest storage cost is important. |

## Encryption
To enable encryption at rest using EC2 and EBS, it must be configured during the creation of the volume.

To encrypt already existing instances, either:
* Enable OS-level encryption (e.g. Windows BitLocker).
* Create a snapshot of the instance. Create a copy with encryption enabled, then use it to create an AMI. Redeploy the EC2 instance and delete the old instance.

Additional attached volumes can be encrypted via the console, CLI or API.
