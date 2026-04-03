---
inclusion: manual
---

# Performance Efficiency Pillar - Scan Checks

The Performance Efficiency pillar focuses on using computing resources efficiently to meet system requirements and maintaining that efficiency as demand changes and technologies evolve.

## Data Dependencies

All data referenced below is collected during the Bulk Data Collection Phase (Phase 1) in POWER.md. Do NOT make additional AWS API calls for data listed here. Analyze the already-collected datasets.

**Collected datasets used by this pillar:**
- EC2: `describe-instances`, `describe-volumes`, `describe-placement-groups`
- RDS: `describe-db-instances`, `describe-db-clusters`, `describe-db-proxies`
- Lambda: `list-functions`
- ECS: `list-clusters`, per-cluster `describe-services`
- EKS: `list-clusters`
- ELB: `describe-load-balancers`
- ElastiCache: `describe-cache-clusters`
- DynamoDB: `list-tables`, per-table `describe-table`
- DAX: `describe-clusters`
- CloudFront: `list-distributions`
- API Gateway: `get-rest-apis`
- CloudWatch: `list-dashboards`, `describe-alarms`
- X-Ray: `get-groups`
- Global Accelerator: `list-accelerators`
- Transit Gateway: `describe-transit-gateways`
- Autoscaling: `describe-auto-scaling-groups`
- Application Autoscaling: `describe-scalable-targets --service-namespace dynamodb`

## Conditional Skip Rules

Skip the following checks when the corresponding resource type has zero results:

**Important**: Distinguish between "zero results" (the API call succeeded but returned no resources) and "data unavailable" (the API call failed due to AccessDeniedException, throttling, or other errors). When data is unavailable due to an API failure, do not skip the check silently. Instead, record an Informational finding stating: "Unable to assess [check name]: [resource type] data was not collected due to [error reason]. See the Error Handling rules in POWER.md." This ensures the report reflects gaps in coverage rather than falsely implying no issues exist.
- No EC2 instances -> skip instance type review, Graviton adoption check, enhanced networking check, placement groups check
- No EBS volumes -> skip EBS volume type check
- No RDS instances and no RDS clusters -> skip RDS instance sizing, RDS Enhanced Monitoring, RDS Performance Insights, RDS storage type, RDS Proxy, Aurora Serverless checks
- No Lambda functions -> skip Lambda memory configuration, Lambda architecture (ARM) check
- No ECS/EKS clusters -> skip Container Insights, ECS/EKS resource limits check
- No ElastiCache clusters -> skip caching layer check
- No DynamoDB tables -> skip DynamoDB capacity mode, DynamoDB auto scaling, DAX checks
- No API Gateways -> skip API Gateway caching check
- No CloudFront distributions -> skip CloudFront check
- No Global Accelerator -> skip Global Accelerator check
- No Transit Gateways -> skip Transit Gateway check
- No RDS Proxies -> skip RDS Proxy check

## Checks to Perform

### Selection (PERF 1-4)

Analyze collected compute data:

1. **Instance type review** (skip if no EC2 instances):
   - From collected `describe-instances` - Identify instance types in use
   - Flag previous-generation instance types (e.g., m4, c4, t2) that should be upgraded to current generation (m7, c7, t3/t4)
   - Flag instances that may be over-provisioned (consistently low CPU/memory utilization)
2. **Graviton adoption** (skip if no EC2 instances): From collected `describe-instances` - Check if ARM-based (Graviton) instance types are being used where applicable. Flag x86 instances that could benefit from Graviton migration for cost and performance gains.
3. **EBS volume types** (skip if no EBS volumes): From collected `describe-volumes` - Flag gp2 volumes that should be migrated to gp3 for better price-performance
4. **RDS instance sizing** (skip if no RDS instances): From collected `describe-db-instances` - Identify previous-generation DB instance classes
5. **Lambda memory configuration** (skip if no Lambda functions): From collected `list-functions` - Flag functions with default 128MB memory (often suboptimal)

### Review (PERF 5)

1. **CloudWatch metrics**: Use `awsapi` to check CloudWatch metrics for (these are live metric queries, not covered by Phase 1 collection):
   - EC2 CPU utilization (flag consistently < 20% or > 80%)
   - RDS CPU and connection counts
   - Lambda duration and throttles
2. **Trusted Advisor**: Use `awsapi` to call `aws support describe-trusted-advisor-checks --language en --region us-east-1` - Check for performance-related recommendations. This is a live check. Note: Requires Business, Enterprise On-Ramp, or Enterprise Support plan.

### Monitoring (PERF 6)

Analyze collected monitoring data:

1. **CloudWatch dashboards**: From collected `list-dashboards` - Verify performance dashboards exist
2. **Enhanced monitoring** (skip if no RDS instances): From collected `describe-db-instances` - Check if Enhanced Monitoring is enabled for RDS
3. **X-Ray tracing**: From collected `get-groups` - Check if distributed tracing is configured
4. **Container Insights** (skip if no ECS/EKS): For ECS/EKS workloads, verify Container Insights is enabled

### Caching and Content Delivery (PERF 3-4)

Analyze collected data with conditional skipping:

1. **CloudFront** (skip if no distributions): From collected `list-distributions` - Check if CloudFront is used for static content delivery
2. **ElastiCache** (skip if no clusters): From collected `describe-cache-clusters` - Check if caching layer exists for database-heavy workloads
3. **API Gateway caching** (skip if no API Gateways): From collected `get-rest-apis` - Check if API caching is enabled
4. **S3 Transfer Acceleration**: Check if enabled for globally accessed buckets
5. **RDS Performance Insights** (skip if no RDS instances): From collected `describe-db-instances` - Check if Performance Insights is enabled for database performance analysis
6. **DynamoDB DAX** (skip if no DynamoDB tables): From collected `describe-clusters` (DAX) - Check if DAX is used for read-heavy DynamoDB workloads requiring microsecond latency
7. **Aurora Serverless** (skip if no RDS clusters): From collected `describe-db-clusters` - Note if Aurora Serverless v2 is used for variable workloads (auto-scales compute)
8. **Global Accelerator** (skip if no accelerators): From collected `list-accelerators` - Check if Global Accelerator is used for latency-sensitive global applications
9. **VPC networking** (skip if no Transit Gateways): From collected `describe-transit-gateways` - Check if Transit Gateway is used for complex multi-VPC architectures instead of multiple peering connections
10. **Placement groups** (skip if no placement groups): From collected `describe-placement-groups` - Check if placement groups are used for low-latency, high-throughput workloads

### Additional Performance Checks

Analyze collected data with conditional skipping:

1. **RDS Proxy** (skip if no RDS Proxies): From collected `describe-db-proxies` - Check if RDS Proxy is used for Lambda-to-RDS connections (reduces connection overhead)
2. **DynamoDB capacity mode** (skip if no DynamoDB tables): From collected per-table `describe-table` - Review tables using provisioned capacity with low utilization that could benefit from on-demand mode, and vice versa
3. **ECS/EKS resource limits** (skip if no ECS clusters): From collected ECS task definitions - Verify CPU and memory limits are set on container definitions to prevent resource contention
4. **Enhanced networking** (skip if no EC2 instances): From collected `describe-instances` - Check if ENA (Elastic Network Adapter) is enabled on instances that support it
5. **DynamoDB auto scaling** (skip if no DynamoDB tables): From collected `describe-scalable-targets --service-namespace dynamodb` - Verify auto scaling is configured for provisioned DynamoDB tables
6. **Lambda architecture (ARM)** (skip if no Lambda functions): From collected `list-functions` - Flag functions using x86_64 that could run on arm64 for better price-performance
7. **RDS storage type** (skip if no RDS instances): From collected `describe-db-instances` - Flag RDS instances using magnetic storage instead of gp3 or io2

## Severity Mapping

- Previous-generation instance types: Medium
- gp2 volumes (should be gp3): Medium
- Lambda functions at 128MB default: Low
- No CloudWatch dashboards: Low
- No caching layer for database-heavy workloads: Medium
- EC2 consistently over 80% CPU: High
- No Enhanced Monitoring on RDS: Low
- No distributed tracing: Low
- No CloudFront for static content: Low
- RDS Performance Insights not enabled: Low
- No DAX for read-heavy DynamoDB workloads: Low
- No Global Accelerator for latency-sensitive global apps: Low
- No RDS Proxy for Lambda-to-RDS workloads: Low
- DynamoDB tables without auto scaling: Medium
- ECS tasks without CPU/memory limits: Medium
- Lambda functions on x86_64 where arm64 is available: Low
- RDS using magnetic storage: Medium
- ENA not enabled on supported instances: Low
