---
inclusion: manual
---

# Cost Optimization Pillar - Scan Checks

The Cost Optimization pillar focuses on avoiding unnecessary costs, understanding where money is being spent, and selecting the most appropriate and right number of resource types.

## Data Dependencies

All data referenced below is collected during the Bulk Data Collection Phase (Phase 1) in POWER.md. Do NOT make additional AWS API calls for data listed here. Analyze the already-collected datasets.

**Collected datasets used by this pillar:**
- Cost Explorer: `get-cost-and-usage`, `get-anomaly-monitors`, `get-reservation-utilization`, `get-savings-plans-utilization`
- Budgets: `describe-budgets`
- Compute Optimizer: `get-enrollment-status`
- EC2: `describe-instances`, `describe-addresses`, `describe-volumes` (all + filtered unattached), `describe-snapshots --owner-ids self`, `describe-nat-gateways`, `describe-vpc-endpoints`, `describe-network-interfaces --filters Name=status,Values=available`, `describe-security-groups`, `describe-transit-gateways`
- ELB: `describe-load-balancers`, `describe-target-groups`
- RDS: `describe-db-instances`
- Lambda: `list-functions`, per-function `get-function-configuration`
- DynamoDB: `list-tables`, per-table `describe-table`
- S3: per-bucket `get-bucket-lifecycle-configuration`
- ECR: `describe-repositories`
- Autoscaling: `describe-auto-scaling-groups`
- Spot: `describe-spot-instance-requests` (add to Phase 1 compute batch if not present)
- Tags: `get-tag-keys`
- CloudWatch Logs: `describe-log-groups`

## Conditional Skip Rules

Skip the following checks when the corresponding resource type has zero results:

**Important**: Distinguish between "zero results" (the API call succeeded but returned no resources) and "data unavailable" (the API call failed due to AccessDeniedException, throttling, or other errors). When data is unavailable due to an API failure, do not skip the check silently. Instead, record an Informational finding stating: "Unable to assess [check name]: [resource type] data was not collected due to [error reason]. See the Error Handling rules in POWER.md." This ensures the report reflects gaps in coverage rather than falsely implying no issues exist.
- No EC2 instances -> skip right-sizing checks, previous-generation instance checks, stopped instance checks
- No EBS volumes -> skip unattached volume check (but the filtered query already handles this)
- No ELB load balancers -> skip idle load balancer check
- No RDS instances -> skip idle RDS check, RDS right-sizing
- No Lambda functions -> skip Lambda over-provisioned memory check, provisioned concurrency check
- No DynamoDB tables -> skip DynamoDB capacity review
- No ECR repositories -> skip unused ECR images check
- No NAT Gateways -> skip NAT Gateway vs VPC endpoint check
- No Autoscaling groups -> skip scale-down check

## Checks to Perform

### Expenditure Awareness (COST 1-3)

Analyze collected cost and tagging data:

1. **Cost allocation tags**: From collected `get-cost-and-usage` - Verify cost visibility
2. **Budgets**: From collected `describe-budgets` - Verify AWS Budgets are configured
3. **Cost anomaly detection**: From collected `get-anomaly-monitors` - Check if Cost Anomaly Detection is enabled
4. **Tagging strategy**: From collected `get-tag-keys` - Check if resources are consistently tagged for cost allocation

### Cost-Effective Resources (COST 4-7)

Analyze collected data with conditional skipping:

1. **Unused resources**:
   - From collected `describe-addresses` - Flag unattached Elastic IPs (each costs money)
   - From collected `describe-volumes --filters Name=status,Values=available` - Flag unattached EBS volumes (already server-side filtered)
   - From collected `describe-snapshots --owner-ids self` - Flag old snapshots (> 90 days)
   - From collected `describe-load-balancers` + `describe-target-groups` (skip if no ELB) - Cross-reference to find idle load balancers
   - From collected `describe-db-instances` (skip if no RDS) - Flag stopped instances (still incur storage costs)
2. **Reserved Instances / Savings Plans**: From collected `get-reservation-utilization` and `get-savings-plans-utilization` - Check RI/SP coverage and utilization
3. **Spot instances**: From collected `describe-spot-instance-requests` - Check if Spot is being used for fault-tolerant workloads
4. **Right-sizing** (skip if no EC2 instances):
   - Cross-reference collected EC2 instance sizes with CloudWatch CPU/memory metrics (live metric query via `awsapi`)
   - Flag instances with consistently low utilization (< 20% CPU average)
5. **Storage tiering**: From collected per-bucket `get-bucket-lifecycle-configuration` - Check if lifecycle policies exist for S3 data tiering

### Manage Demand and Supply (COST 8-9)

Analyze collected data:

1. **Auto Scaling** (skip if no ASGs): From collected `describe-auto-scaling-groups` - Verify ASGs scale down during low demand
2. **Lambda provisioned concurrency** (skip if no Lambda functions): Use `awsapi` to call `aws lambda list-provisioned-concurrency-configs --function-name <fn>` for functions found in collected data - Flag unnecessary provisioned concurrency
3. **NAT Gateway usage** (skip if no NAT Gateways): From collected `describe-nat-gateways` and `describe-vpc-endpoints` - Flag multiple NAT Gateways where VPC endpoints could reduce costs
4. **Data transfer patterns**: From collected `describe-transit-gateways` and VPC peering data - Check for cross-region data replication that may be unnecessary. Review Transit Gateway attachments for cross-AZ/cross-region traffic that could be optimized.
5. **S3 same-region access points**: Use `awsapi` to call `aws s3control list-access-points --account-id <account>` - Check if access points are used to keep S3 traffic within the same region/VPC. This is a live check.

### Optimize Over Time (COST 10-11)

Analyze collected data:

1. **Previous-generation resources**: From collected EC2, EBS, and RDS data - Flag old instance types, gp2 volumes, and other resources that have newer, cheaper alternatives
2. **Compute Optimizer**: From collected `get-enrollment-status` - Check if Compute Optimizer is enabled for right-sizing recommendations
3. **Resource decommissioning**: From collected data - Cross-reference all resource types for decommissioning candidates: stopped EC2 instances older than 30 days (from `describe-instances`), unused security groups (from `describe-security-groups`), empty target groups (from `describe-target-groups`), and orphaned ENIs (from `describe-network-interfaces --filters Name=status,Values=available`, already server-side filtered)

### Additional Cost Checks

Analyze collected data with conditional skipping:

1. **DynamoDB capacity review** (skip if no DynamoDB tables): From collected per-table `describe-table` - Flag provisioned tables with consistently low consumed capacity that should switch to on-demand, and on-demand tables with steady high throughput that could save with provisioned mode
2. **CloudWatch Logs ingestion**: From collected `describe-log-groups` - Flag log groups with high ingestion volume but no retention policy (infinite retention accumulates cost)
3. **Lambda over-provisioned memory** (skip if no Lambda functions): Cross-reference collected Lambda function memory configuration with CloudWatch `MaxMemoryUsed` metric (live metric query via `awsapi`). Flag functions where allocated memory is more than double the peak usage.
4. **Unused ECR images** (skip if no ECR repos): From collected `describe-repositories` - Flag repositories without lifecycle policies that may accumulate old images
5. **Idle RDS instances** (skip if no RDS instances): Cross-reference collected RDS instances with CloudWatch `DatabaseConnections` metric (live metric query via `awsapi`). Flag instances with zero connections over the past 14 days.
6. **S3 request costs**: Use `awsapi` to call `aws s3api list-bucket-analytics-configurations --bucket <bucket>` for each bucket - Check if S3 analytics is enabled on high-traffic buckets. This is a live check for per-bucket analytics config.
7. **Data transfer costs**: From collected `describe-nat-gateways` and `describe-vpc-endpoints` - Estimate potential savings from replacing NAT Gateway traffic with VPC endpoints for AWS service calls

## Severity Mapping

- Unattached Elastic IPs: Medium
- Unattached EBS volumes: Medium
- No AWS Budgets configured: High
- No cost allocation tags: High
- Idle load balancers: Medium
- No lifecycle policies on large S3 buckets: Medium
- No Savings Plans or Reserved Instances for steady-state workloads: High
- Compute Optimizer not enabled: Low
- Old snapshots accumulating: Low
- Multiple NAT Gateways without VPC endpoints: Medium
- Lambda functions with over-provisioned memory: Low
- ECR repositories without lifecycle policies: Low
- Idle RDS instances (zero connections): High
- DynamoDB tables on wrong capacity mode: Medium
- No S3 analytics on high-traffic buckets: Low
- Stopped EC2 instances older than 30 days: Medium
- Orphaned ENIs and unused security groups: Low
- Cross-region data transfer without justification: Medium
