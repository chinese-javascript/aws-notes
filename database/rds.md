# Relational Database Service
Amazon RDS provides a managed service for setting up, operating and scaling an RDB in the cloud.

## Supported RDBs
* Amazon Aurora
* PostgreSQL
* MySQL
* MariaDB
* Oracle Database
* SQL Server

# Backups
RDS offers *Automated Backups* and *Snapshots* for backing up instances. Restored versions of a DB will be a new RDS instance (with a new DNS endpoint).

## Automated Backups (default enabled)
This backup method takes full daily snapshots along with transaction logs throughout the day, and this will allow you to recover your DB to any point in time within a *retention period* (1-35 days). Backup data is stored on S3 (same amount of storage as the DB size). When a recovery is done the most recent daily backup is chosen and the captured transaction logs are re-applied. These backups get deleted upon deletion of the RDS instance.

## Snapshots (manual)
Takes a point-in-time snapshot of the DB. Unlike auto-backups, these are stored even after the RDS instance is deleted.

# Redundancy
## Read Replicas (Performance/Scaling)
Provides up to 5 read-only copies of your prod DB (which get updated via asynchronous replication) to cater for very read-heavy workloads. Each read replica will get its own DNS endpoint which can be connected to (e.g. by an EC2 instance in an Auto Scaling group).

Read replicas can be promoted to become their own DBs and stop replication.

## Multi-AZ (Disaster Recovery)
Creates exact copies of your prod DB in another AZ via synchronous replication. All writes to the primary DB get automatically replicated in the standby DBs. If the main prod DB goes down, the DNS endpoint will point to a functioning multi-AZ copy.

*Each DB copy gets a different IP, however the DNS endpoint will only point to one functioning copy. Any connections made to a DB with multi-AZ must be via the DNS endpoint (not a DB instance IP).*

# Encryption
Encryption at rest is supported for all RDB types and if enabled on the RDS instance, will apply to all stored data and backups. 

*Encryption can't be toggled on an already existing DB instance. Workaround is to take a snapshot, make a copy of the snapshot (creates new RDS instance) and from there encryption can be enabled.*

# Monitoring
## Database Log Files
### MySQL Log Files
You can monitor the *MySQL error log*, *slow query log*, and the *general log*. The MySQL error log is by default written to the `mysql-error.log` file. The other logs are configured via the DB parameter group.
