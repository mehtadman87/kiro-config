---
inclusion: manual
---

# Sustainability Pillar - Scan Checks

The Sustainability pillar focuses on minimizing the environmental impacts of running cloud workloads by understanding impact, establishing sustainability goals, and maximizing utilization.

## Data Dependencies

All data referenced below is collected during the Bulk Data Collection Phase (Phase 1) in POWER.md. Do NOT make additional AWS API calls for data listed here. Analyze the already-collected datasets.

**Collected datasets used by this pillar:**
- EC2: `describe-instances`, `describe-volumes`, `describe-volumes --filters Name=status,Values=available`, `describe-addresses`, `describe-snapshots --owner-ids self`
- RDS: `describe-db-instances`
- Lambda: `list-functions`, per-function `get-function-configuration`
- ECS: `list-clusters`
- EKS: `list-clusters`
- ElastiCache: `describe-cache-clusters`
- DynamoDB: `list-tables`, per-table `describe-table`
- S3: per-bucket `get-bucket-lifecycle-configuration`
- SQS: `list-queues`
- Step Functions: `list-state-machines`
- CloudFront: `list-distributions`
- VPC Endpoints: `describe-vpc-endpoints`
- Autoscaling: `describe-auto-scaling-groups`
- Tags: `get-tag-keys`
- ELB: `describe-load-balancers`

## Conditional Skip Rules

Skip the following checks when the corresponding resource type has zero results:

**Important**: Distinguish between "zero results" (the API call succeeded but returned no resources) and "data unavailable" (the API call failed due to AccessDeniedException, throttling, or other errors). When data is unavailable due to an API failure, do not skip the check silently. Instead, record an Informational finding stating: "Unable to assess [check name]: [resource type] data was not collected due to [error reason]. See the Error Handling rules in POWER.md." This ensures the report reflects gaps in coverage rather than falsely implying no issues exist.
- No EC2 instances -> skip Graviton adoption check (EC2), right-sizing check, instance stop/start schedule check
- No RDS instances -> skip Graviton adoption check (RDS)
- No Lambda functions -> skip Lambda runtime check, Lambda architecture check, Lambda ephemeral storage check
- No ECS/EKS clusters -> skip containerized workloads check, ECS/Fargate right-sizing check
- No ElastiCache clusters -> skip ElastiCache Graviton check
- No DynamoDB tables -> skip DynamoDB on-demand mode check
- No EBS volumes -> skip EBS volume optimization check, unused storage check (volumes)
- No SQS queues and no Step Functions -> skip async processing patterns check
- No CloudFront distributions -> skip CloudFront check
- No VPC Endpoints -> skip VPC endpoints check (but still flag as finding if NAT Gateways exist)

## Checks to Perform

### Region Selection (SUS 1)

1. **Region awareness**: From collected `describe-instances` - Document which regions are in use. Note that some regions have lower carbon intensity than others. Reference AWS's published carbon footprint data.

### Compute Efficiency (SUS 2-3)

Analyze collected compute data:

1. **Graviton adoption** (skip per resource type if absent): From collected `describe-instances`, `describe-db-instances`, `describe-cache-clusters`, and `list-functions` - Check instance types across EC2, RDS, ElastiCache, and Lambda. Flag x86 instances where Graviton alternatives exist (Graviton processors are more energy-efficient).
2. **Right-sizing** (skip if no EC2 instances): Cross-reference collected instance data with CloudWatch metrics (live metric query via `awsapi`). Over-provisioned instances waste energy.
3. **Spot and serverless**: From collected Lambda, ECS, and Spot data - Check for Lambda, Fargate, and Spot usage which improve overall fleet utilization.
4. **Auto Scaling** (skip if no ASGs): From collected `describe-auto-scaling-groups` - Verify workloads scale down during low demand to reduce idle compute.
5. **Lambda architecture** (skip if no Lambda functions): From collected `list-functions` - Check runtime. Newer runtimes (e.g., Python 3.12, Node.js 20) are generally more efficient.
6. **Async processing patterns** (skip if no SQS and no Step Functions): From collected `list-queues` and `list-state-machines` - Check if asynchronous processing (SQS, Step Functions, EventBridge) is used instead of synchronous polling. Async patterns reduce idle compute and improve resource utilization.
7. **Containerized workloads** (skip if no ECS/EKS): From collected `list-clusters` (ECS and EKS) - Note container adoption. Containers improve compute density and utilization compared to dedicated EC2 instances per workload.

### Storage Efficiency (SUS 4)

Analyze collected storage data:

1. **S3 lifecycle policies**: From collected per-bucket `get-bucket-lifecycle-configuration` - Verify data is transitioned to lower-cost, lower-energy storage tiers (Glacier, Intelligent-Tiering)
2. **S3 Intelligent-Tiering**: From collected S3 data - Check if Intelligent-Tiering is used for unpredictable access patterns
3. **EBS volume optimization** (skip if no EBS volumes): From collected `describe-volumes` - Flag over-provisioned IOPS volumes and gp2 volumes (gp3 is more efficient)
4. **Unused storage**: From collected `describe-volumes --filters Name=status,Values=available` (already server-side filtered) and `describe-snapshots` - Flag unattached EBS volumes and old snapshots

### Data Transfer Efficiency (SUS 5)

Analyze collected networking data:

1. **CloudFront** (skip if no distributions): From collected `list-distributions` - Verify CDN is used to reduce data transfer and origin load
2. **VPC endpoints**: From collected `describe-vpc-endpoints` - Check if VPC endpoints are used to keep traffic within the AWS network
3. **S3 Transfer Acceleration**: Only flag if enabled unnecessarily (it adds overhead)

### Resource Management (SUS 6)

Analyze collected data:

1. **Tagging for sustainability**: From collected `get-tag-keys` - Check if resources are tagged with environment, team, or project to enable tracking and cleanup
2. **Idle resources**: Reuse findings from Cost Optimization pillar analysis for unused EIPs (from `describe-addresses`), unattached volumes (from filtered `describe-volumes`), and idle load balancers (from `describe-load-balancers` + `describe-target-groups`). Do NOT re-query these resources. Idle resources consume energy.
3. **Managed services**: From collected data across all resource types - Note usage of managed services (RDS, DynamoDB, Lambda, Fargate) vs self-managed EC2. Managed services generally have better utilization rates.

### Additional Sustainability Checks

Analyze collected data with conditional skipping:

1. **Customer Carbon Footprint Tool**: The Customer Carbon Footprint Tool data is only available via the AWS Console (Billing > Cost Management > Customer Carbon Footprint Tool). This cannot be retrieved via CLI. Recommend users check the Console manually to establish a sustainability baseline.
2. **ECS/Fargate right-sizing** (skip if no ECS clusters): From collected ECS service data - Cross-reference Fargate task CPU/memory with CloudWatch metrics (live metric query via `awsapi`) to identify over-provisioned tasks
3. **DynamoDB on-demand mode** (skip if no DynamoDB tables): From collected per-table `describe-table` - On-demand mode avoids over-provisioned capacity that wastes energy during low-traffic periods
4. **S3 storage class distribution**: From collected S3 bucket data - Flag buckets storing large amounts of data in S3 Standard that could use Glacier or Intelligent-Tiering
5. **EC2 instance stop/start schedules** (skip if no EC2 instances): From collected `describe-instances` - Check for Instance Scheduler or tagged schedules on non-production instances. Dev/test instances running 24/7 waste energy.
6. **Lambda ephemeral storage** (skip if no Lambda functions): From collected per-function `get-function-configuration` - Flag functions with over-provisioned ephemeral storage (default 512MB is sufficient for most workloads)

## Severity Mapping

- No Auto Scaling (always-on over-provisioned compute): Medium
- No S3 lifecycle policies on large buckets: Medium
- gp2 volumes instead of gp3: Low
- No Graviton adoption where available: Low
- Unattached EBS volumes and idle resources: Medium
- No CloudFront for static content: Low
- No VPC endpoints for AWS service traffic: Low
- Outdated Lambda runtimes: Low
- No tagging for resource lifecycle management: Low
- Non-production instances running 24/7 without schedules: Medium
- Over-provisioned Fargate tasks: Low
- Large S3 buckets entirely in Standard storage class: Low
- Lambda functions with over-provisioned ephemeral storage: Low
