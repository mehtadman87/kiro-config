---
inclusion: manual
---

# Data Analytics Lens - Scan Checks

The Data Analytics Lens focuses on best practices for designing well-architected analytics workloads on AWS. It covers data ingestion, storage, processing, and consumption patterns across services like AWS Glue, Amazon Redshift, Amazon Athena, Amazon EMR, Amazon Kinesis, Amazon MSK, AWS Lake Formation, and Amazon QuickSight.

Reference: https://docs.aws.amazon.com/wellarchitected/latest/analytics-lens/analytics-lens.html

## Data Dependencies

All data referenced below is collected during the Bulk Data Collection Phase (Phase 1) in POWER.md plus the additional analytics-specific data collected in the Data Analytics supplemental collection step. Do NOT make additional AWS API calls for data listed here. Analyze the already-collected datasets.

**Collected datasets used by this lens (from base Phase 1):**
- S3: `list-buckets`, per-bucket `get-bucket-encryption`, `get-bucket-versioning`, `get-bucket-lifecycle-configuration`, `get-bucket-replication`
- IAM: `list-policies --scope Local`, per-policy `get-policy-version`, `list-users`, `get-credential-report`
- EC2: `describe-instances`, `describe-security-groups`, `describe-flow-logs`, `describe-vpcs`
- Lambda: `list-functions`, per-function `get-function-configuration`
- CloudTrail: `describe-trails`
- Config: `describe-configuration-recorders`, `describe-config-rules`
- CloudWatch: `describe-alarms`, `list-dashboards`
- SNS: `list-topics`, `list-subscriptions`
- KMS: `list-keys`, per-key `get-key-rotation-status`
- Step Functions: `list-state-machines`
- EventBridge: `list-rules`
- Tags: `get-tag-keys`
- Cost Explorer: `get-cost-and-usage`, `get-reservation-utilization`, `get-savings-plans-utilization`
- Compute Optimizer: `get-enrollment-status`
- CloudFormation: `list-stacks`
- Logs: `describe-log-groups`

**Additional analytics-specific datasets (collected in supplemental step):**
- Glue: `get-databases`, `get-crawlers`, `get-jobs`, `list-data-quality-rulesets`, `get-security-configurations`, `list-workflows`
- Redshift: `describe-clusters`, `describe-cluster-snapshots --snapshot-type automated`
- Athena: `list-work-groups`, `list-named-queries`
- EMR: `list-clusters --active`
- Kinesis: `list-streams`
- MSK: `list-clusters-v2`
- Lake Formation: `get-data-lake-settings`, `list-permissions`
- QuickSight: `list-dashboards --aws-account-id <account>`, `list-data-sources --aws-account-id <account>`
- Macie: (from cached `CheckSecurityServices`)
- OpenSearch: `list-domain-names`

## Conditional Skip Rules

Skip the following checks when the corresponding resource type has zero results:

**Important**: Distinguish between "zero results" (the API call succeeded but returned no resources) and "data unavailable" (the API call failed due to AccessDeniedException, throttling, or other errors). When data is unavailable due to an API failure, do not skip the check silently. Instead, record an Informational finding stating: "Unable to assess [check name]: [resource type] data was not collected due to [error reason]. See the Error Handling rules in POWER.md." This ensures the report reflects gaps in coverage rather than falsely implying no issues exist.
- No Glue databases -> skip Data Catalog checks
- No Glue crawlers -> skip crawler configuration checks
- No Glue jobs -> skip Glue job monitoring, Glue job bookmark, and Glue security configuration checks
- No Glue data quality rulesets -> skip data quality rule checks (but flag as finding if Glue jobs exist)
- No Glue workflows -> skip workflow orchestration checks
- No Redshift clusters -> skip all Redshift checks
- No Athena work groups -> skip Athena checks
- No EMR clusters -> skip EMR checks
- No Kinesis streams -> skip Kinesis checks
- No MSK clusters -> skip MSK checks
- No Lake Formation settings -> skip Lake Formation checks
- No QuickSight dashboards and data sources -> skip QuickSight checks
- No OpenSearch domains -> skip OpenSearch checks
- No Step Functions state machines -> skip orchestration checks via Step Functions

## Checks to Perform

### Operational Excellence: Monitor Health of Analytics Workload (BP 1)

Analyze collected Glue, CloudWatch, and monitoring data:

1. **Data quality validation** (skip if no Glue data quality rulesets): From collected `list-data-quality-rulesets` - Verify data quality rules are defined for source data validation before processing. If Glue jobs exist but no quality rulesets are defined, flag as finding.
2. **Pipeline monitoring**: From collected `describe-alarms` and `list-dashboards` - Check for CloudWatch alarms and dashboards monitoring analytics job metrics (Glue job runs, EMR step failures, Redshift query performance, Kinesis iterator age).
3. **Glue job metrics monitoring** (skip if no Glue jobs): From collected `get-jobs` - Verify Glue jobs have CloudWatch metrics enabled (DefaultArguments should include `--enable-metrics`).
4. **Log group retention for analytics**: From collected `describe-log-groups` - Check that log groups for analytics services (`/aws-glue/`, `/aws/emr/`, `/aws/redshift/`) have appropriate retention policies set (not indefinite).

### Operational Excellence: Modernize Deployment (BP 2)

Analyze collected CI/CD and infrastructure-as-code data:

1. **Infrastructure as code**: From collected `list-stacks` - Check if analytics resources (Glue, Redshift, EMR) are deployed via CloudFormation or CDK stacks rather than manually provisioned. (BP 2.1)
2. **Glue workflow orchestration** (skip if no Glue workflows and no Step Functions state machines): From collected `list-workflows` and `list-state-machines` - Verify ETL jobs are orchestrated through Glue Workflows or Step Functions rather than ad-hoc scheduling. (BP 2.4)
3. **EventBridge scheduling**: From collected `list-rules` - Check for EventBridge rules that schedule analytics jobs, indicating automated pipeline execution. (BP 2.4)

### Security: Data Platform Governance and Compliance (BP 3)

Analyze collected encryption, classification, and lifecycle data:

1. **S3 data lake encryption**: From collected per-bucket `get-bucket-encryption` - Verify all S3 buckets used for analytics data have encryption at rest enabled (SSE-S3 or SSE-KMS). (BP 3.6)
2. **S3 lifecycle policies**: From collected per-bucket `get-bucket-lifecycle-configuration` - Verify data retention policies are implemented via S3 lifecycle rules for analytics data buckets. (BP 3.7)
3. **Glue Data Catalog encryption** (skip if no Glue databases): From collected `get-security-configurations` - Check if Glue security configurations enforce encryption for the Data Catalog, job bookmarks, and S3 targets. (BP 3.6)
4. **Macie for sensitive data discovery**: From cached `CheckSecurityServices` - Check if Amazon Macie is enabled for discovering and protecting sensitive data in S3 data lakes. (BP 3.1, BP 3.2)
5. **KMS key management**: From collected `list-keys` and per-key `get-key-rotation-status` - Verify KMS keys used by analytics services have automatic rotation enabled. (BP 3.6)
6. **S3 versioning for data lakes**: From collected per-bucket `get-bucket-versioning` - Verify versioning is enabled on S3 buckets storing analytics data to support data recovery and audit trails. (BP 3.6)

### Security: Data Access Control (BP 4)

Analyze collected Lake Formation and IAM data:

1. **Lake Formation access control** (skip if no Lake Formation settings): From collected `get-data-lake-settings` - Verify Lake Formation is configured as the central access control mechanism for the data lake rather than relying solely on IAM policies and S3 bucket policies. (BP 4.1)
2. **Lake Formation permissions** (skip if no Lake Formation settings): From collected `list-permissions` - Review Lake Formation permissions for overly broad grants (e.g., `ALL` permissions on databases or tables to wide principal groups). (BP 4.3)
3. **Redshift database security** (skip if no Redshift clusters): From collected `describe-clusters` - Verify Redshift clusters enforce SSL connections (RequireSsl parameter) and use encryption at rest. (BP 4.3)
4. **Athena workgroup isolation** (skip if no Athena work groups): From collected `list-work-groups` - Verify Athena work groups are configured to enforce query result encryption and workgroup-level access control. (BP 4.3)
5. **CloudTrail data event logging**: From collected `describe-trails` - Verify CloudTrail is logging data events for S3 and Lambda to provide an audit trail of data access in analytics workloads. (BP 4.5)
6. **Redshift audit logging** (skip if no Redshift clusters): From collected Redshift logging status data - Verify Redshift audit logging is enabled to track database changes and user activity. (BP 4.5)

### Security: Infrastructure Access Control (BP 5)

Analyze collected network and access data:

1. **Redshift cluster network isolation** (skip if no Redshift clusters): From collected `describe-clusters` - Verify Redshift clusters are deployed in VPCs and not publicly accessible (PubliclyAccessible should be false). (BP 5.1)
2. **EMR cluster security** (skip if no EMR clusters): From collected EMR cluster data - Verify EMR clusters are launched in VPCs with appropriate security group configurations. (BP 5.1)
3. **Glue connection security** (skip if no Glue jobs): From collected Glue job data - Check if Glue jobs that connect to data stores use VPC connections for network isolation. (BP 5.1)
4. **OpenSearch domain security** (skip if no OpenSearch domains): From collected `list-domain-names` - Verify OpenSearch domains are deployed within VPCs and have encryption at rest and in transit enabled. (BP 5.1)
5. **MSK cluster encryption** (skip if no MSK clusters): From collected `list-clusters-v2` - Verify MSK clusters have encryption in transit (TLS) and at rest enabled. (BP 5.1)
6. **Least privilege IAM policies for analytics roles**: From collected IAM policies - Check for overly permissive IAM policies on analytics service roles (e.g., Glue execution roles, EMR service roles, Redshift IAM roles) that grant `*` actions or resources. (BP 5.2)
7. **Infrastructure change monitoring**: From collected `describe-trails` and `describe-config-rules` - Verify CloudTrail is logging management events for analytics services and AWS Config rules are monitoring analytics resource configuration changes. (BP 5.3)
8. **Audit log protection**: From collected CloudTrail data - Verify CloudTrail logs are stored in S3 buckets with log file validation enabled and bucket policies preventing deletion. (BP 5.4)

### Reliability: Design Resilience for Analytics Workload (BP 6)

Analyze collected monitoring and alerting data:

1. **Glue job failure monitoring** (skip if no Glue jobs): From collected `get-jobs` and `describe-alarms` - Verify CloudWatch alarms exist for Glue job failure metrics (`glue.driver.aggregate.numFailedTasks`).
2. **Glue job retry configuration** (skip if no Glue jobs): From collected `get-jobs` - Check if Glue jobs have retry configurations set (MaxRetries > 0) for automatic recovery from transient failures.
3. **SNS alerting for pipeline failures**: From collected `list-topics` and `list-subscriptions` - Verify SNS topics exist with active subscriptions for notifying stakeholders about analytics pipeline failures.
4. **Redshift automated snapshots** (skip if no Redshift clusters): From collected `describe-clusters` and `describe-cluster-snapshots` - Verify Redshift clusters have automated snapshots enabled with appropriate retention periods.
5. **S3 cross-region replication for critical data**: From collected per-bucket `get-bucket-replication` - Check if critical analytics data buckets have cross-region replication configured for disaster recovery.
6. **Kinesis stream resilience** (skip if no Kinesis streams): From collected `list-streams` - Verify Kinesis streams have appropriate shard counts and retention periods for the workload requirements.

### Reliability: Govern Data and Metadata Changes (BP 7)

Analyze collected Data Catalog and data governance data:

1. **Central Data Catalog** (skip if no Glue databases): From collected `get-databases` - Verify a Glue Data Catalog is configured with databases and tables to serve as the central metadata repository. (BP 7.1)
2. **Glue crawlers for schema management** (skip if no Glue crawlers): From collected `get-crawlers` - Verify Glue crawlers are configured to keep the Data Catalog in sync with source data schema changes. (BP 7.1)
3. **Glue job bookmarks** (skip if no Glue jobs): From collected `get-jobs` - Check if Glue jobs have job bookmarks enabled (DefaultArguments should include `--job-bookmark-option job-bookmark-enable`) to track processed data and support incremental processing. (BP 7.1)
4. **Lake Formation data lineage** (skip if no Lake Formation settings): From collected Lake Formation data - Check if data lineage tracking is available and configured. (BP 7.3)
5. **Data quality anomaly monitoring** (skip if no Glue data quality rulesets): From collected `list-data-quality-rulesets` and `describe-alarms` - Verify CloudWatch alarms are configured to alert on data quality rule failures, enabling proactive detection of data quality anomalies in the pipeline. (BP 7.2)

### Performance Efficiency: Compute, Storage, and File Format (BP 8-10)

Analyze collected compute and storage configuration data:

1. **Redshift cluster sizing** (skip if no Redshift clusters): From collected `describe-clusters` - Review cluster node types and counts. Flag single-node clusters in production as they lack redundancy and limit performance. (BP 8.1)
2. **EMR instance fleet diversity** (skip if no EMR clusters): From collected EMR cluster data - Check if EMR clusters use instance fleets with multiple instance types for better availability and cost optimization. (BP 8.1)
3. **Glue worker type selection** (skip if no Glue jobs): From collected `get-jobs` - Review Glue job worker types (Standard, G.1X, G.2X, G.025X) and flag jobs using Standard workers for memory-intensive workloads. (BP 8.1)
4. **Athena workgroup query limits** (skip if no Athena work groups): From collected `list-work-groups` - Check if Athena work groups have query data scan limits configured to prevent runaway queries. (BP 8.3)
5. **Redshift Spectrum usage** (skip if no Redshift clusters): From collected Redshift cluster data - Check if Redshift Spectrum is available for querying data directly in S3, enabling storage-compute decoupling. (BP 9.2)
6. **Kinesis shard configuration** (skip if no Kinesis streams): From collected Kinesis stream data - Review shard counts relative to throughput requirements. Flag streams that may be under-provisioned or over-provisioned. (BP 8.4)
7. **EMR managed scaling** (skip if no EMR clusters): From collected EMR cluster data - Check if EMR clusters have managed scaling enabled to automatically adjust capacity based on workload demand. (BP 8.4)

### Cost Optimization: Compute and Storage Efficiency (BP 11-14)

Analyze collected cost and resource utilization data:

1. **Redshift Reserved Nodes** (skip if no Redshift clusters): From collected `get-reservation-utilization` and `describe-clusters` - Check if long-running Redshift clusters are covered by Reserved Node purchases. (BP 14.1)
2. **EMR Spot Instance usage** (skip if no EMR clusters): From collected EMR cluster data - Check if EMR task nodes use Spot Instances for cost savings on non-critical processing. (BP 14.1)
3. **S3 storage class optimization**: From collected per-bucket `get-bucket-lifecycle-configuration` - Verify S3 lifecycle policies transition infrequently accessed analytics data to cost-effective storage classes (S3 IA, S3 Glacier). (BP 11.1)
4. **Glue job DPU optimization** (skip if no Glue jobs): From collected `get-jobs` - Flag Glue jobs with high DPU allocations (NumberOfWorkers) that may be over-provisioned. Check if auto scaling is enabled. (BP 11.4)
5. **Cost allocation tagging**: From collected `get-tag-keys` - Verify analytics resources are tagged with cost allocation tags (e.g., `Project`, `Team`, `Environment`) for financial accountability. (BP 12.1)
6. **Compute Optimizer enrollment**: From collected `get-enrollment-status` - Verify AWS Compute Optimizer is enabled to receive right-sizing recommendations for analytics compute resources. (BP 13.2)
7. **Redshift pause/resume** (skip if no Redshift clusters): From collected `describe-clusters` - Flag provisioned Redshift clusters that may benefit from pause/resume scheduling for non-production workloads. Check if Redshift Serverless would be more cost-effective. (BP 13.3)
8. **Unused analytics resources**: From collected resource data - Flag unused or idle analytics resources: stopped Redshift clusters without pause/resume schedules, idle EMR clusters in WAITING state, unattached EBS volumes from terminated EMR clusters. (BP 13.1)

### Sustainability (BP 15)

Analyze collected resource efficiency data:

1. **Data retention policies**: From collected per-bucket `get-bucket-lifecycle-configuration` - Verify lifecycle policies exist to remove or archive unnecessary data, reducing storage footprint. (BP 15.3, BP 15.4)
2. **Efficient resource utilization** (skip if no Redshift clusters and no EMR clusters): From collected cluster data - Flag over-provisioned analytics clusters that waste compute resources. Check for auto scaling configurations. (BP 15.7)
3. **Serverless analytics adoption**: From collected Glue, Athena, and Redshift data - Check if the workload uses serverless options (Glue, Athena, Redshift Serverless) where appropriate to minimize idle resource consumption. (BP 15.7)
4. **Columnar file format adoption** (skip if no Glue databases): From collected Glue Data Catalog table metadata - Check the `Classification` field on tables. Flag tables using row-oriented formats (csv, json) for large analytical datasets where columnar formats (parquet, orc) would reduce storage and improve query performance. (BP 15.5)

## Severity Mapping

- S3 data lake buckets without encryption: High
- Glue Data Catalog without encryption: High
- Redshift cluster publicly accessible: Critical
- Redshift cluster without encryption at rest: High
- Redshift cluster without SSL enforcement: High
- Lake Formation not configured (relying on IAM-only access control): Medium
- Lake Formation overly broad permissions: High
- No CloudTrail data events for S3: Medium
- EMR clusters not in VPC: High
- OpenSearch domains not in VPC: High
- MSK clusters without encryption in transit: High
- No data quality rules defined for Glue jobs: Medium
- No CloudWatch alarms for analytics pipeline failures: Medium
- No CloudWatch dashboards for analytics monitoring: Low
- Glue jobs without metrics enabled: Low
- Analytics log groups without retention policies: Low
- Analytics resources not deployed via IaC: Low
- No ETL orchestration (no Glue Workflows or Step Functions): Medium
- Glue jobs without retry configuration: Medium
- No SNS alerting for pipeline failures: Medium
- Redshift without automated snapshots: High
- No cross-region replication for critical data: Low
- No Glue Data Catalog configured: Medium
- Glue crawlers not configured for schema management: Low
- Glue jobs without job bookmarks: Low
- Single-node Redshift cluster in production: Medium
- Athena work groups without query scan limits: Low
- Redshift clusters without Reserved Nodes (long-running): Medium
- EMR task nodes not using Spot Instances: Low
- No S3 lifecycle policies for analytics data: Medium
- Glue jobs potentially over-provisioned: Low
- Analytics resources missing cost allocation tags: Medium
- Compute Optimizer not enabled: Low
- Macie not enabled for data lake sensitive data discovery: Medium
- S3 data lake buckets without versioning: Medium
- Athena work groups without query result encryption: Medium
- Glue security configurations not defined: Medium
- Analytics roles with overly permissive IAM policies: High
- No infrastructure change monitoring for analytics services: Medium
- CloudTrail log file validation not enabled: Medium
- Redshift audit logging not enabled: Medium
- No data quality anomaly alerting: Low
- Kinesis streams under/over-provisioned: Low
- EMR clusters without managed scaling: Low
- Unused or idle analytics resources: Medium
- Large analytical tables using row-oriented formats (csv/json) instead of columnar (parquet/orc): Low
