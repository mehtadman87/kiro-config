---
inclusion: manual
---

# Reliability Pillar - Scan Checks

The Reliability pillar focuses on ensuring a workload performs its intended function correctly and consistently when expected. This includes the ability to operate and test the workload through its total lifecycle.

## Data Dependencies

All data referenced below is collected during the Bulk Data Collection Phase (Phase 1) in POWER.md. Do NOT make additional AWS API calls for data listed here. Analyze the already-collected datasets.

**Collected datasets used by this pillar:**
- EC2: `describe-instances`, `describe-vpcs`, `describe-subnets`, `describe-nat-gateways`, `describe-vpn-connections`
- RDS: `describe-db-instances`, `describe-db-clusters`, `describe-db-snapshots` (collect if not already listed - add to Phase 1 database batch)
- Lambda: `list-functions`, per-function `get-function-configuration`
- ECS: `list-clusters`, per-cluster `describe-services`
- EKS: `list-clusters`
- ELB: `describe-load-balancers`, `describe-target-groups`
- ElastiCache: `describe-replication-groups`
- DynamoDB: `list-tables`, per-table `describe-continuous-backups`
- SQS: `list-queues`, per-queue `get-queue-attributes` (RedrivePolicy)
- S3: per-bucket `get-bucket-versioning`, `get-bucket-replication`
- Route 53: `list-health-checks`, `list-hosted-zones`, per-zone `list-resource-record-sets`
- CloudWatch: `describe-alarms`
- Autoscaling: `describe-auto-scaling-groups`, `describe-policies`
- API Gateway: `get-rest-apis`, per-API `get-stages`
- Backup: `list-backup-plans` (add to Phase 1 operations batch if not present)
- CodeDeploy: `list-applications`, `list-deployment-configs`
- FIS: `list-experiment-templates` (add to Phase 1 operations batch if not present)

## Conditional Skip Rules

Skip the following checks when the corresponding resource type has zero results:

**Important**: Distinguish between "zero results" (the API call succeeded but returned no resources) and "data unavailable" (the API call failed due to AccessDeniedException, throttling, or other errors). When data is unavailable due to an API failure, do not skip the check silently. Instead, record an Informational finding stating: "Unable to assess [check name]: [resource type] data was not collected due to [error reason]. See the Error Handling rules in POWER.md." This ensures the report reflects gaps in coverage rather than falsely implying no issues exist.
- No RDS instances and no RDS clusters -> skip all RDS reliability checks (Multi-AZ, backups, snapshots, read replicas, Aurora cluster checks)
- No Lambda functions -> skip Lambda DLQ/destinations check, Lambda reserved concurrency check
- No ECS clusters -> skip ECS service desired count check
- No EKS clusters -> skip EKS task/replica count check
- No ELB load balancers -> skip health check and ALB multi-AZ checks
- No ElastiCache replication groups -> skip ElastiCache Multi-AZ check
- No DynamoDB tables -> skip DynamoDB PITR check
- No SQS queues -> skip SQS dead-letter queue check
- No API Gateways -> skip API Gateway throttling and logging checks
- No VPN connections -> skip VPN tunnel redundancy check
- No NAT Gateways -> skip single-AZ NAT Gateway check
- No Route 53 health checks and no hosted zones -> skip Route 53 failover checks
- No Autoscaling groups -> skip ASG and scaling policy checks
- No Backup plans and no RDS instances -> skip backup checks (but still flag as finding)

## Checks to Perform

### Foundations (REL 1-2)

1. **Service quotas**: Use `awsapi` to call `aws service-quotas list-service-quotas --service-code ec2` - Check if usage is approaching limits for key services (EC2, RDS, Lambda, ELB). This is a live check not covered by Phase 1 collection.
2. **VPC design**: From collected `describe-vpcs` and `describe-subnets` - Verify multi-AZ subnet deployment
3. **IP address capacity**: From collected `describe-subnets` - Check subnet CIDR ranges for sufficient IP space
4. **VPN tunnel redundancy** (skip if no VPN connections): From collected `describe-vpn-connections` - Verify both tunnels are UP for each VPN connection. A single tunnel is a single point of failure.
5. **EC2 instances in VPC** (skip if no EC2 instances): From collected `describe-instances` - Verify all instances are deployed within a VPC (not EC2-Classic)

### Workload Architecture (REL 3-5)

Analyze collected data:

1. **Multi-AZ deployments**:
   - From collected `describe-db-instances` (skip if no RDS) - Flag single-AZ RDS instances
   - From collected `describe-load-balancers` (skip if no ELB) - Verify ALBs span multiple AZs
   - From collected `describe-instances` (skip if no EC2) - Check instance distribution across AZs
2. **Auto Scaling** (skip if no ASGs): From collected `describe-auto-scaling-groups` - Verify ASGs exist for EC2 workloads
3. **Loose coupling**: From collected SQS, SNS, and EventBridge data - Check for queues, topics, and rules that indicate decoupled architectures
4. **API Gateway throttling** (skip if no API Gateways): From collected per-API `get-stages` - Verify throttle settings are configured to protect downstream services from overload
5. **SQS dead-letter queues** (skip if no SQS queues): From collected per-queue `get-queue-attributes` - Verify DLQs are configured for failure handling
6. **Lambda DLQ/destinations** (skip if no Lambda functions): From collected per-function `get-function-configuration` - Check if Lambda functions have dead-letter queues or on-failure destinations configured

### Change Management (REL 6-8)

Analyze collected data:

1. **CloudWatch monitoring**: From collected `describe-alarms` - Verify alarms exist for key metrics (CPU, memory, disk, error rates)
2. **Auto Scaling policies** (skip if no ASGs): From collected `describe-policies` - Verify scaling policies are configured
3. **Health checks** (skip if no ELB): From collected `describe-target-groups` - Verify health checks are configured on target groups
4. **Deployment automation**: From collected `list-applications` (CodeDeploy) - Check if CodeDeploy is used with rolling or blue/green deployment strategies to manage change safely
5. **API Gateway execution logging** (skip if no API Gateways): From collected per-API `get-stages` - Verify execution logging is enabled on API Gateway stages for monitoring and troubleshooting
6. **ASG ELB health checks** (skip if no ASGs): From collected `describe-auto-scaling-groups` - Verify ASGs use ELB health checks (not just EC2 status checks) when behind a load balancer

### Failure Management (REL 9-13)

Analyze collected data:

1. **Backups**:
   - From collected `list-backup-plans` - Verify AWS Backup plans exist
   - From collected `describe-db-instances` (skip if no RDS) - Check automated backup retention period (flag if < 7 days)
   - From collected `describe-db-snapshots` (skip if no RDS) - Verify snapshots exist
2. **Multi-AZ RDS** (skip if no RDS): Flag any production RDS instance without Multi-AZ
3. **S3 versioning**: From collected per-bucket `get-bucket-versioning` - Verify versioning is enabled on important buckets
4. **DynamoDB backups** (skip if no DynamoDB tables): From collected per-table `describe-continuous-backups` - Check point-in-time recovery
5. **ECS/EKS task count** (skip if no ECS/EKS): From collected ECS service data - Verify services run more than one task/replica
6. **Fault isolation**: From collected `describe-subnets` and `describe-nat-gateways` - Verify workloads use multiple AZs. Check if NAT Gateways exist in each AZ with subnets. Flag single-AZ architectures.
7. **Fault Injection Service (FIS)**: From collected `list-experiment-templates` - Check if chaos engineering experiments are defined for reliability testing
8. **Disaster recovery**: From collected RDS data check for cross-region read replicas (ReadReplicaDBInstanceIdentifiers), from collected S3 data check cross-region replication, from collected Route 53 data check failover routing policies. Flag workloads with no DR strategy.

### Additional Reliability Checks

Analyze collected data with conditional skipping:

1. **Route 53 health checks** (skip if no Route 53): From collected `list-health-checks` - Verify DNS health checks exist for critical endpoints
2. **Route 53 failover routing** (skip if no hosted zones): From collected per-zone `list-resource-record-sets` - Check if failover routing policies are configured for critical domains
3. **S3 cross-region replication**: From collected per-bucket `get-bucket-replication` - Check if critical buckets have cross-region replication for disaster recovery
4. **RDS read replicas** (skip if no RDS): From collected `describe-db-instances` - Check if read replicas exist for high-availability database workloads
5. **Lambda reserved concurrency** (skip if no Lambda functions): From collected per-function `get-function-configuration` - Check if critical Lambda functions have reserved concurrency to prevent throttling
6. **ECS service desired count** (skip if no ECS clusters): From collected per-cluster `describe-services` - Flag services with desiredCount of 1 (no redundancy)
7. **ElastiCache Multi-AZ** (skip if no ElastiCache): From collected `describe-replication-groups` - Flag Redis replication groups without Multi-AZ enabled
8. **Aurora cluster instances** (skip if no RDS clusters): From collected `describe-db-clusters` - Flag Aurora clusters with only one instance (no failover target)

## Severity Mapping

- No backups configured: Critical
- Single-AZ RDS in production: High
- No Auto Scaling configured: High
- No CloudWatch alarms: High
- Missing health checks on load balancers: High
- S3 versioning not enabled: Medium
- Service quotas above 80% utilization: Medium
- Single instance without ASG: Medium
- No SQS/SNS for decoupling: Low
- DynamoDB PITR not enabled: Medium
- No Route 53 health checks for critical endpoints: High
- No cross-region replication for critical S3 buckets: Medium
- ECS services with desiredCount of 1: Medium
- ElastiCache without Multi-AZ: Medium
- Aurora cluster with single instance: High
- No Lambda reserved concurrency for critical functions: Low
- SQS queues without dead-letter queues: Medium
- Lambda functions without DLQ or on-failure destination: Medium
- No FIS experiment templates (no chaos testing): Low
- No disaster recovery strategy (no cross-region replicas or failover): High
- Single-AZ NAT Gateway architecture: Medium
- VPN connections with only one tunnel UP: High
- ASGs not using ELB health checks when behind load balancer: Medium
- API Gateway stages without execution logging: Low
