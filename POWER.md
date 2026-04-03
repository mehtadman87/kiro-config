---
name: "aws-well-architected"
displayName: "AWS Well-Architected Scanner"
description: "Scan and validate AWS accounts against the AWS Well-Architected Framework best practices across all six pillars"
keywords: ["aws", "well-architected", "security", "reliability", "performance", "cost-optimization", "sustainability", "operational-excellence", "compliance", "best-practices", "scan", "audit", "review", "data-analytics", "analytics-lens", "generative-ai", "bedrock", "sagemaker", "genai-lens"]
author: "Agasthi Kothurkar"
---

# AWS Well-Architected Scanner Power

Scan and validate your AWS account against the AWS Well-Architected Framework best practices across all six pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, and Sustainability.

## Overview

This power provides AI-assisted scanning and validation of your AWS environment against the Well-Architected Framework. It combines the AWS Well-Architected Security Assessment MCP server with the AWS API and Knowledge MCP servers to deliver comprehensive account reviews covering all six pillars.

The agent uses live AWS API calls to inspect your actual resource configurations, compares them against Well-Architected best practices, and produces actionable findings with severity ratings and remediation guidance.

## Onboarding

### Step 1: Validate prerequisites

Before using this power, ensure the following are available:

- **AWS credentials**: Configured via AWS CLI profiles, environment variables, or IAM roles. The credentials need read-only access to the services being scanned.
  - Verify with: `aws sts get-caller-identity`
  - **Recommended IAM policy**: Attach the AWS-managed `ReadOnlyAccess` policy (`arn:aws:iam::aws:policy/ReadOnlyAccess`) to the IAM role or user running the scan. This covers all `describe-*`, `list-*`, and `get-*` calls used by the assessment. In addition, attach a small inline policy to allow credential report generation, which is not included in `ReadOnlyAccess`:
    ```json
    {
      "Version": "2012-10-17",
      "Statement": [
        {
          "Effect": "Allow",
          "Action": [
            "iam:GenerateCredentialReport",
            "iam:GetCredentialReport"
          ],
          "Resource": "*"
        }
      ]
    }
    ```
  - If you prefer a least-privilege approach instead of `ReadOnlyAccess`, the scan requires read permissions across: EC2, RDS, DynamoDB, Lambda, ECS, EKS, ECR, S3, IAM, CloudTrail, Config, CloudWatch, CloudWatch Logs, CloudFormation, SNS, SQS, SSM, KMS, Secrets Manager, WAFv2, Shield, Route 53, CloudFront, API Gateway, ELBv2, Auto Scaling, Application Auto Scaling, CodePipeline, CodeBuild, CodeDeploy, CodeGuru Reviewer, Backup, Cost Explorer, Budgets, Compute Optimizer, Health, FIS, X-Ray, Synthetics, Application Insights, EventBridge, Resource Explorer, Resource Groups Tagging, Organizations, SSO Admin, Step Functions, and DAX. When the Data Analytics Lens is in scope, add: Glue, Redshift, Athena, EMR, Kinesis, MSK (Kafka), Lake Formation, QuickSight, and OpenSearch. When the Generative AI Lens is in scope, add: Bedrock, Bedrock Agent, SageMaker, and OpenSearch Serverless.
- **uv/uvx**: Python package manager for running MCP servers
  - Verify with: `uvx --version`
- **Python 3.10+**: Required by the Well-Architected Security MCP server

If AWS credentials are not configured, DO NOT proceed with scanning. Guide the user to configure credentials first.

### Step 2: Validate IAM permissions

After confirming credentials are configured, run the following permission checks using `call_aws` in batch mode before proceeding. These calls verify that the IAM principal has the minimum permissions required for the scan.

**Core permission probe** (single batch call):
- `aws iam get-account-summary`
- `aws ec2 describe-instances --max-items 1`
- `aws s3api list-buckets --max-items 1`
- `aws lambda list-functions --max-items 1`
- `aws rds describe-db-instances --max-items 1`
- `aws cloudtrail describe-trails --max-items 1`
- `aws configservice describe-configuration-recorders`
- `aws cloudwatch describe-alarms --max-items 1`
- `aws iam generate-credential-report`

**Evaluation rules:**

1. If `aws iam get-account-summary` or `aws ec2 describe-instances` returns `AccessDeniedException` or `UnauthorizedAccess`, the principal is missing fundamental read permissions. STOP the scan and tell the user: "Your IAM principal lacks the required read permissions. Attach the AWS-managed `ReadOnlyAccess` policy (`arn:aws:iam::aws:policy/ReadOnlyAccess`) and the supplemental inline policy documented in Step 1, then try again."
2. If `aws iam generate-credential-report` returns `AccessDeniedException`, warn the user: "Your IAM principal is missing `iam:GenerateCredentialReport`. IAM credential report checks will be skipped. To include them, add the inline policy documented in Step 1." Record this as an Informational finding but allow the scan to continue.
3. If any other individual probe fails with an access error, record it as a warning but allow the scan to continue. The scan is designed to degrade gracefully for individual service permission gaps (see Error Handling in the Scan Workflow section).
4. If all probes succeed, proceed to Step 3.

### Step 3: Confirm target account and regions

Before running any scan, always confirm with the user:
1. Which AWS account to scan (verify with `aws sts get-caller-identity`)
2. Which regions to scan. The user can specify:
   - A list of specific regions (e.g., `us-east-1, us-west-2, eu-west-1`)
   - `all` to auto-discover all enabled regions via `aws ec2 describe-regions --query "Regions[].RegionName" --output text`
   - If not specified, default to the region configured in their AWS CLI profile
3. Which pillars to focus on (or all six)
4. Whether to include the Data Analytics Lens assessment. This adds analytics-specific checks for services like AWS Glue, Amazon Redshift, Amazon Athena, Amazon EMR, Amazon Kinesis, Amazon MSK, AWS Lake Formation, and Amazon QuickSight. Include it when the account runs analytics workloads.
5. Whether to include the Generative AI Lens assessment. This adds checks for Amazon Bedrock, Amazon SageMaker AI, guardrails, knowledge bases, agents, vector stores, and model lifecycle management. Include it when the account runs generative AI workloads.

Note: Some checks are global (IAM, S3 bucket policies, Route 53, CloudFront) and only need to run once regardless of region selection. The agent should run global checks from `us-east-1` and regional checks in each specified region.


### Step 4: Run a scan

Once prerequisites are met, simply ask:

- "Run a well-architected scan"
- "Scan my AWS account for best practices"
- "Audit my account against the Well-Architected Framework"
- "Run a data analytics lens assessment"
- "Scan my analytics workloads against the Data Analytics Lens"
- "Run a generative AI lens assessment"
- "Scan my Bedrock and SageMaker workloads against the Generative AI Lens"

## MCP Servers

This power includes the following MCP servers:

### well-architected-security

- **Purpose**: Assess AWS environments against the Well-Architected Security Pillar. Monitors GuardDuty, Security Hub, Inspector, and IAM Access Analyzer status. Retrieves security findings and analyzes security posture.
- **Command**: `uvx --from awslabs.well-architected-security-mcp-server awslabs.well-architected-security-mcp-server`
- **Key tools**: CheckSecurityServices, GetSecurityFindings, AnalyzeSecurityPosture, ExploreAwsResources, GetResourceComplianceStatus
- **Note**: Tool names listed above are representative. Always confirm exact tool names and parameters by activating the power first.
- **Profile support**: Most tools accept an optional `aws_profile` parameter. When the user is using a named AWS CLI profile, pass it to every tool call. If omitted, the server defaults to the `AWS_PROFILE` environment variable or `default`.

### awsknowledge

- **Purpose**: Access AWS best practices documentation, service guides, and Well-Architected Framework pillar whitepapers
- **Type**: HTTP server
- **URL**: https://knowledge-mcp.global.api.aws

### awsapi

- **Purpose**: Execute read-only AWS CLI commands to inspect resource configurations across all pillars
- **Command**: `uvx awslabs.aws-api-mcp-server@latest`

## When to Load Steering Files

- Scanning or reviewing security configurations -> `security-pillar.md`
- Scanning or reviewing reliability configurations -> `reliability-pillar.md`
- Scanning or reviewing performance configurations -> `performance-efficiency-pillar.md`
- Scanning or reviewing cost configurations -> `cost-optimization-pillar.md`
- Scanning or reviewing operational excellence -> `operational-excellence-pillar.md`
- Scanning or reviewing sustainability configurations -> `sustainability-pillar.md`
- Scanning or reviewing data analytics workloads -> `data-analytics-lens.md`
- Scanning or reviewing generative AI workloads -> `generative-ai-lens.md`

## Scan Workflow

When the user asks to scan their AWS account, follow this workflow. The workflow is optimized to minimize API calls through batching, cross-pillar data reuse, region-level parallelism, conditional skipping, and server-side filtering.

### Data Collection Principles

These principles apply to every phase of the scan:

1. **Collect once, analyze many times**: Each AWS API call should be made exactly once. The collected data is reused across all six pillar assessments. Never re-query a resource type that has already been collected.
2. **Batch API calls**: Use `call_aws` batch mode (up to 20 commands per call) to execute independent API calls concurrently. Group calls by dependency level, not by pillar.
3. **Region-level parallelism**: Use `--region *` on regional `describe-*` and `list-*` calls to query all target regions in a single invocation instead of iterating region by region.
4. **Server-side filtering**: Always use `--filters`, `--query`, or `--scope` parameters when the API supports them to reduce response payload size. For example, use `--filters Name=status,Values=available` for unattached volumes rather than fetching all volumes and filtering client-side.
5. **Conditional skipping**: If a resource type has zero results from the collection phase, skip all pillar checks that depend on that resource type. The resource-to-check mapping is defined in each steering file.
6. **Store and reuse context**: Use `store_in_context=True` on all `well-architected-security` MCP calls so subsequent tools can read cached results without re-querying AWS.
7. **Respect the 20-command batch limit**: The `call_aws` batch mode accepts at most 20 commands per call. If a batch group lists more than 20 commands, split it into multiple batch calls. Never send more than 20 commands in a single `call_aws` invocation.

### Error Handling

These rules govern how the agent should handle failures during data collection:

1. **AccessDeniedException or UnauthorizedAccess**: If an API call fails because the user's IAM role lacks permissions, skip that resource type and record an Informational finding in the relevant pillar(s) stating: "Unable to assess [resource type]: insufficient IAM permissions. Grant [specific permission] to include this check in future scans." Do not retry the call or halt the scan.
2. **Throttling (ThrottlingException, TooManyRequestsException, Rate exceeded)**: Wait 5 seconds and retry the call once. If it fails again, skip the resource type and record an Informational finding noting the throttling. Do not retry more than once.
3. **Service not available in region**: Some services are not available in all regions. If a call returns an endpoint error or `InvalidClientTokenId` for a specific region, skip that service for that region silently. This is expected behavior, not a finding.
4. **Other errors**: For any other unexpected error, skip the resource type, record an Informational finding with the error message, and continue the scan. Never halt the entire scan because of a single failed API call.
5. **MCP tool failures**: If a `well-architected-security` MCP tool call fails, record an Informational finding noting the tool and error, and continue. The scan should degrade gracefully, not abort.
6. **Pass the correct `aws_profile`**: When the user is using a named AWS CLI profile (not the default), pass the `aws_profile` parameter to all `well-architected-security` MCP tool calls (`CheckSecurityServices`, `CheckStorageEncryption`, `CheckNetworkSecurity`, `ListServicesInRegion`, `GetSecurityFindings`). Determine the active profile from the user's environment or by asking during Step 2.

### 1. Bulk Data Collection Phase

This phase replaces the previous Discovery Phase. Instead of a lightweight inventory followed by per-pillar re-queries, collect all resource data upfront in batched, parallelized calls. The collected data serves every pillar assessment.

#### Step 1a: Global resource collection (run once from us-east-1)

Batch the following global service calls into `call_aws` batch calls (up to 20 per batch):

**IAM batch** (single batch call):
- `aws iam get-account-summary`
- `aws iam list-users`
- `aws iam get-account-password-policy`
- `aws iam list-policies --scope Local`
- `aws iam generate-credential-report`
- `aws sso-admin list-instances`

Note: Per-user calls (`list-mfa-devices`, `list-access-keys`) depend on the output of `list-users` and must run in Step 1d (Dependent detail calls), not here.

**S3 batch** (single batch call):
- `aws s3api list-buckets`
- Then for each bucket (batched, up to 20 per call):
  - `aws s3api get-bucket-encryption --bucket <bucket>`
  - `aws s3api get-public-access-block --bucket <bucket>`
  - `aws s3api get-bucket-versioning --bucket <bucket>`
  - `aws s3api get-bucket-lifecycle-configuration --bucket <bucket>`
  - `aws s3api get-bucket-replication --bucket <bucket>`

**Other global services batch**:
- `aws organizations describe-organization`
- `aws cloudfront list-distributions`
- `aws route53 list-health-checks`
- `aws route53 list-hosted-zones`
- `aws budgets describe-budgets --account-id <account>`
- `aws ce get-anomaly-monitors`
- `aws ce get-cost-and-usage --time-period Start=<30-days-ago>,End=<today> --granularity MONTHLY --metrics BlendedCost`
- `aws ce get-reservation-utilization --time-period Start=<30-days-ago>,End=<today>`
- `aws ce get-savings-plans-utilization --time-period Start=<30-days-ago>,End=<today>`
- `aws compute-optimizer get-enrollment-status`
- `aws shield describe-subscription`
- `aws resourcegroupstaggingapi get-tag-keys`

#### Step 1b: Regional resource collection (use --region * for all target regions)

Use `--region *` to collect data across all target regions in a single call per resource type. Batch independent calls together.

**Compute batch** (single batch call with `--region *`):
- `aws ec2 describe-instances --region *`
- `aws ec2 describe-volumes --region *`
- `aws ec2 describe-volumes --filters Name=status,Values=available --region *` (unattached volumes)
- `aws ec2 describe-addresses --region *`
- `aws ec2 describe-security-groups --region *`
- `aws ec2 describe-flow-logs --region *`
- `aws ec2 describe-vpcs --region *`
- `aws ec2 describe-subnets --region *`
- `aws ec2 describe-route-tables --region *`
- `aws ec2 describe-network-acls --region *`
- `aws ec2 describe-nat-gateways --region *`
- `aws ec2 describe-vpc-endpoints --region *`
- `aws ec2 describe-transit-gateways --region *`
- `aws ec2 describe-vpn-connections --region *`
- `aws ec2 describe-network-interfaces --filters Name=status,Values=available --region *`
- `aws ec2 describe-placement-groups --region *`
- `aws ec2 get-ebs-encryption-by-default --region *`
- `aws ec2 describe-snapshots --owner-ids self --region *`
- `aws ec2 describe-spot-instance-requests --region *`

**Database batch** (single batch call with `--region *`):
- `aws rds describe-db-instances --region *`
- `aws rds describe-db-clusters --region *`
- `aws rds describe-db-proxies --region *`
- `aws rds describe-db-snapshots --snapshot-type manual --region *`
- `aws dynamodb list-tables --region *`
- `aws elasticache describe-cache-clusters --region *`
- `aws elasticache describe-replication-groups --region *`
- `aws dax describe-clusters --region *`
- `aws backup list-backup-plans --region *`

**Serverless and containers batch** (single batch call with `--region *`):
- `aws lambda list-functions --region *`
- `aws ecs list-clusters --region *`
- `aws eks list-clusters --region *`
- `aws ecr describe-repositories --region *`
- `aws stepfunctions list-state-machines --region *`

**Networking and delivery batch** (single batch call with `--region *`):
- `aws elasticloadbalancingv2 describe-load-balancers --region *`
- `aws elasticloadbalancingv2 describe-target-groups --region *`
- `aws apigateway get-rest-apis --region *`
- `aws wafv2 list-web-acls --scope REGIONAL --region *`
- `aws globalaccelerator list-accelerators --region us-west-2`

**Operations and monitoring batch 1** (first batch call with `--region *`, 20 commands):
- `aws cloudtrail describe-trails --region *`
- `aws configservice describe-configuration-recorders --region *`
- `aws configservice describe-config-rules --region *`
- `aws configservice describe-conformance-packs --region *`
- `aws cloudwatch describe-alarms --region *`
- `aws cloudwatch list-dashboards --region *`
- `aws cloudwatch describe-insight-rules --region *`
- `aws logs describe-log-groups --region *`
- `aws sns list-topics --region *`
- `aws sns list-subscriptions --region *`
- `aws sqs list-queues --region *`
- `aws ssm describe-instance-information --region *`
- `aws ssm describe-parameters --region *`
- `aws ssm describe-patch-baselines --region *`
- `aws ssm describe-maintenance-windows --region *`
- `aws ssm list-documents --filters Key=DocumentType,Values=Automation --region *`
- `aws events list-rules --region *`
- `aws synthetics describe-canaries --region *`
- `aws application-insights list-applications --region *`
- `aws xray get-groups --region *`

**Operations and monitoring batch 2** (second batch call with `--region *`, 4 commands):
- `aws resource-explorer-2 list-indexes --region *`
- `aws cloudformation list-stacks --stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE --region *`
- `aws health describe-events --filter eventStatusCodes=open --region *`
- `aws fis list-experiment-templates --region *`

**Security services batch** (single batch call with `--region *`):
- `aws kms list-keys --region *`
- `aws secretsmanager list-secrets --region *`

**CI/CD batch** (single batch call with `--region *`):
- `aws codepipeline list-pipelines --region *`
- `aws codedeploy list-applications --region *`
- `aws codedeploy list-deployment-configs --region *`
- `aws codebuild list-projects --region *`
- `aws codeguru-reviewer list-repository-associations --region *`
- `aws lambda list-code-signing-configs --region *`

**Autoscaling batch** (single batch call with `--region *`):
- `aws autoscaling describe-auto-scaling-groups --region *`
- `aws autoscaling describe-policies --region *`
- `aws application-autoscaling describe-scalable-targets --service-namespace dynamodb --region *`

#### Step 1c: Security service status collection

Use the `well-architected-security` MCP server with `store_in_context=True`:
- Call `CheckSecurityServices` with `store_in_context=True` for each target region (or batch if the tool supports it). This caches the status of GuardDuty, Security Hub, Inspector, IAM Access Analyzer, Trusted Advisor, and Macie.
- Call `CheckStorageEncryption` with `store_in_context=True` for each target region.
- Call `CheckNetworkSecurity` with `store_in_context=True` for each target region.
- Call `ListServicesInRegion` with `store_in_context=True` for each target region.

All subsequent calls to `GetSecurityFindings` and `GetStoredSecurityContext` will read from the cached context without making additional AWS API calls.

#### Step 1d: Dependent detail calls

Some checks require a second-level lookup based on results from Step 1a/1b. Batch these as well:

- **Per IAM user** (after `list-users`): batch `list-mfa-devices` and `list-access-keys` calls (up to 20 per batch)
- **Per IAM policy** (after `list-policies`): batch `get-policy-version` calls
- **Per KMS key** (after `list-keys`): batch `get-key-rotation-status` calls
- **Per Lambda function** (after `list-functions`): batch `get-function-configuration` and `get-policy` calls
- **Per DynamoDB table** (after `list-tables`): batch `describe-table` and `describe-continuous-backups` calls
- **Per SQS queue** (after `list-queues`): batch `get-queue-attributes --attribute-names All` calls
- **Per SNS topic** (after `list-topics`): batch `get-topic-attributes` calls
- **Per ECS cluster** (after `list-clusters`): batch `list-services` then `describe-services` calls
- **Per EKS cluster** (after `list-clusters`): batch `describe-cluster` calls
- **Per CodeBuild project** (after `list-projects`): batch `batch-get-projects` calls
- **Per CodePipeline** (after `list-pipelines`): batch `get-pipeline` calls
- **Per API Gateway** (after `get-rest-apis`): batch `get-stages` calls
- **Per Route 53 hosted zone** (after `list-hosted-zones`): batch `list-resource-record-sets` calls
- **IAM credential report**: `aws iam get-credential-report` (after `generate-credential-report`)

#### Step 1e: Analytics services supplemental collection (only when Data Analytics Lens is in scope)

When the user requests a Data Analytics Lens assessment (or a full scan that includes it), collect additional analytics-specific data. Skip this step entirely if the Data Analytics Lens is not in scope.

**Glue batch** (single batch call with `--region *`):
- `aws glue get-databases --region *`
- `aws glue get-crawlers --region *`
- `aws glue get-jobs --region *`
- `aws glue list-data-quality-rulesets --region *`
- `aws glue get-security-configurations --region *`
- `aws glue list-workflows --region *`

**Redshift batch** (single batch call with `--region *`):
- `aws redshift describe-clusters --region *`
- `aws redshift describe-cluster-snapshots --snapshot-type automated --region *`

**Athena batch** (single batch call with `--region *`):
- `aws athena list-work-groups --region *`
- `aws athena list-named-queries --region *`

**EMR batch** (single batch call with `--region *`):
- `aws emr list-clusters --active --region *`

**Streaming batch** (single batch call with `--region *`):
- `aws kinesis list-streams --region *`
- `aws kafka list-clusters-v2 --region *`

**Lake Formation and governance batch** (single batch call with `--region *`):
- `aws lakeformation get-data-lake-settings --region *`
- `aws lakeformation list-permissions --region *`

**QuickSight batch** (run from us-east-1):
- `aws quicksight list-dashboards --aws-account-id <account>`
- `aws quicksight list-data-sources --aws-account-id <account>`

**OpenSearch batch** (single batch call with `--region *`):
- `aws opensearch list-domain-names --region *`

**Dependent detail calls for analytics services:**
- **Per OpenSearch domain** (after `list-domain-names`): batch `describe-domain` calls to check VPC, encryption, and access policy configurations
- **Per Kinesis stream** (after `list-streams`): batch `describe-stream-summary` calls to check shard count and retention
- **Per MSK cluster** (after `list-clusters-v2`): batch `describe-cluster-v2` calls to check encryption and configuration
- **Per Athena work group** (after `list-work-groups`): batch `get-work-group` calls to check encryption and query limit settings
- **Per Redshift cluster** (after `describe-clusters`): batch `describe-logging-status` calls to check audit logging

#### Step 1e-genai: Generative AI services supplemental collection (only when Generative AI Lens is in scope)

When the user requests a Generative AI Lens assessment (or a full scan that includes it), collect additional generative AI-specific data. Skip this step entirely if the Generative AI Lens is not in scope.

**Early termination probe** (single batch call with `--region *`):
Before full collection, run a lightweight probe to detect if any generative AI resources exist:
- `aws bedrock list-custom-models --max-items 1 --region *`
- `aws bedrock-agent list-agents --max-items 1 --region *`
- `aws sagemaker list-endpoints --max-items 1 --region *`

If all three return empty results, skip the entire Generative AI Lens. No further supplemental collection is needed.

**Bedrock batch** (single batch call with `--region *`):
- `aws bedrock list-foundation-models` (run once from us-east-1 only -- this is a global catalog, same result in every region)
- `aws bedrock list-custom-models --region *`
- `aws bedrock list-model-customization-jobs --region *`
- `aws bedrock list-provisioned-model-throughputs --region *`
- `aws bedrock list-guardrails --region *`
- `aws bedrock list-inference-profiles --region *`
- `aws bedrock list-evaluation-jobs --region *`

**Bedrock Agent batch** (single batch call with `--region *`):
- `aws bedrock-agent list-knowledge-bases --region *`
- `aws bedrock-agent list-agents --region *`

**SageMaker batch** (single batch call with `--region *`):
- `aws sagemaker list-endpoints --region *`
- `aws sagemaker list-models --region *`
- `aws sagemaker list-training-jobs --sort-by CreationTime --sort-order Descending --max-items 20 --region *`
- `aws sagemaker list-notebook-instances --region *`

**OpenSearch Serverless batch** (single batch call with `--region *`):
- `aws opensearchserverless list-collections --region *`

**Conditional dependent detail calls** (skip each group if the parent list returned zero results):
- **Per Bedrock agent** (only if agents exist): batch `get-agent` calls to check guardrail associations, action groups, and execution role permissions
- **Per Bedrock knowledge base** (only if knowledge bases exist): batch `get-knowledge-base` calls to check data source configurations and IAM roles
- **Per Bedrock guardrail** (only if guardrails exist): batch `get-guardrail` calls to check content filter and denied topic configurations
- **Per SageMaker endpoint** (only if endpoints exist): batch `describe-endpoint` calls to check instance types, auto scaling, and VPC configuration
- **Per SageMaker notebook instance** (only if notebooks exist): batch `describe-notebook-instance` calls to check direct internet access and VPC settings
- **Per OpenSearch Serverless collection** (only if collections exist): batch `batch-get-collection` calls to check type (VECTORSEARCH) and encryption

#### Step 1f: Build the resource inventory

After all collection calls complete, build an in-memory resource inventory keyed by resource type and region. This inventory is the single source of truth for all pillar assessments. Record which resource types returned zero results so those checks can be skipped.

For each resource type, track one of three states:
- **present**: The API call succeeded and returned one or more resources.
- **empty**: The API call succeeded but returned zero resources. Checks depending on this resource type should be skipped per the conditional skip rules in each steering file.
- **failed**: The API call failed (AccessDeniedException, throttling, service unavailable, or other error). Checks depending on this resource type must NOT be silently skipped. Instead, record an Informational finding for each affected check stating the data was unavailable and why. This ensures the report reflects gaps in assessment coverage.

**Resource presence flags** (used for conditional skipping):
- `has_ec2_instances`, `has_rds_instances`, `has_rds_clusters`, `has_lambda_functions`, `has_ecs_clusters`, `has_eks_clusters`, `has_dynamodb_tables`, `has_elasticache`, `has_elb`, `has_api_gateways`, `has_vpn_connections`, `has_nat_gateways`, `has_sqs_queues`, `has_sns_topics`, `has_codepipeline`, `has_codebuild`, `has_ecr_repos`, `has_waf`, `has_secrets`, `has_kms_keys`, `has_step_functions`, `has_cloudfront`, `has_route53`, `has_dax`, `has_global_accelerator`, `has_transit_gateways`, `has_placement_groups`, `has_rds_proxies`
- Analytics-specific (only when Data Analytics Lens is in scope): `has_glue_databases`, `has_glue_crawlers`, `has_glue_jobs`, `has_glue_data_quality_rulesets`, `has_glue_workflows`, `has_redshift_clusters`, `has_athena_workgroups`, `has_emr_clusters`, `has_kinesis_streams`, `has_msk_clusters`, `has_lakeformation`, `has_quicksight`, `has_opensearch_domains`
- Generative AI-specific (only when Generative AI Lens is in scope): `has_bedrock_custom_models`, `has_bedrock_provisioned_throughputs`, `has_bedrock_guardrails`, `has_bedrock_knowledge_bases`, `has_bedrock_agents`, `has_bedrock_inference_profiles`, `has_sagemaker_endpoints`, `has_sagemaker_models`, `has_sagemaker_training_jobs`, `has_sagemaker_notebooks`, `has_opensearch_serverless_collections`

### 2. Pillar Assessment Phase

Analyze the collected data against each pillar's checks. Do NOT make additional AWS API calls for data that was already collected in Phase 1. Each steering file specifies which collected datasets it uses and which checks to skip when a resource type is absent.

When the Data Analytics Lens is in scope, also assess the collected analytics data against the checks defined in `data-analytics-lens.md`. The Data Analytics Lens assessment runs as a separate lens alongside the six pillar assessments and produces its own tab in the report.

When the Generative AI Lens is in scope, also assess the collected generative AI data against the checks defined in `generative-ai-lens.md`. The Generative AI Lens assessment runs as a separate lens alongside the six pillar assessments and produces its own tab in the report.

Use `awsknowledge` to reference best practices documentation as needed.

Use `GetSecurityFindings` (which reads from the cached context stored in Phase 1) to retrieve active findings for the Security pillar.

Tag each finding with the region where the resource was found. Use `global` for IAM, S3, Route 53, and CloudFront findings.

### 3. Findings Report

After all pillar assessments are complete, generate a single consolidated HTML report. The report must be a self-contained HTML file (all CSS and JS inline, no external dependencies) that can be opened directly in a browser.

#### Report File Naming

Name the report file based on the assessment the user requested. Use the following convention:

| Assessment requested | File name |
|---|---|
| Full Well-Architected scan (all six pillars, no lenses) | `well-architected-report.html` |
| Full Well-Architected scan with Data Analytics Lens | `well-architected-with-analytics-report.html` |
| Full Well-Architected scan with Generative AI Lens | `well-architected-with-genai-report.html` |
| Full Well-Architected scan with both lenses | `well-architected-full-report.html` |
| Data Analytics Lens only | `data-analytics-lens-report.html` |
| Generative AI Lens only | `generative-ai-lens-report.html` |
| Single pillar scan (e.g., Security only) | `well-architected-security-report.html` (use the pillar name in lowercase, hyphenated) |

If the user explicitly provides a custom file name, use that instead.

#### Multi-Step Report Generation

Because the HTML report can be very large (especially for multi-region scans with many findings), generate the report incrementally across multiple file-write operations. This prevents any single write from exceeding context or output limits. Follow these steps in order:

**Step 3a: Write the HTML shell and CSS**

Create the report file with the opening `<!DOCTYPE html>`, `<html>`, `<head>` (containing all `<style>` blocks and Cloudscape design tokens), and close the `</head>`. Then open the `<body>` and write the top navigation bar, the scope caveat banner, and the tab navigation bar. End this write after the tab bar markup and the opening `<div>` for the tab content area. Do not include any findings data in this step.

**Step 3b: Write the Executive Summary tab**

Append the Executive Summary tab content: overall posture rating banner, pillar score cards grid, region breakdown, and Quick Wins section. Close the Executive Summary tab `<div>`.

**Step 3c: Write pillar tabs (one append per pillar)**

For each pillar that is in scope, append one tab `<div>` containing:
- The pillar rating
- The region and severity filter dropdowns
- The findings table for that pillar
- The summary of checks that passed

Write each pillar as a separate append operation. The order is: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability.

If the scan covers only a single pillar, this step produces one append. If all six pillars are in scope, this step produces six appends.

**Step 3d: Write lens tabs (if applicable)**

If the Data Analytics Lens is in scope, append the Data Analytics tab `<div>` with its findings, filters, and the documentation link note.

If the Generative AI Lens is in scope, append the Generative AI tab `<div>` with its findings, filters, and the documentation link note.

Each lens tab is a separate append operation.

**Step 3e: Write the closing JavaScript and HTML**

Append the `<script>` block containing all tab-switching logic, filter interactivity, and severity sorting. Then close the `</body>` and `</html>` tags.

**Important**: Each step must produce valid, well-formed HTML when combined with the previous steps. Do not leave unclosed tags between steps except for the intentional container `<div>` and `<body>` that will be closed in a later step.

#### HTML Report Content

The HTML report must include:

- **Header**: Report title, AWS account ID, regions scanned, scan timestamp, and overall posture rating (Critical / Needs Improvement / Good / Excellent)
- **Scope Caveat Banner**: A visible notice below the header stating: "This automated assessment covers only best practices that can be reliably verified through AWS API inspection. Certain best practices that require manual review, architectural judgment, or application-level inspection (such as prompt engineering quality, data classification accuracy, CI/CD pipeline purpose, file format selection, storage-compute decoupling patterns, user feedback mechanisms, and load testing practices) are excluded from this scan. Consult the full Well-Architected Framework and lens documentation for a comprehensive review." Style this as a Cloudscape alert/flashbar component with an `info` type: light blue background (`#f0f4ff`), left border `4px solid #0972d3`, `14px` font size, `12px 16px` padding.
- **Executive Summary Tab**: Overall posture rating with a summary paragraph, pillar score cards showing each pillar's rating and finding counts by severity, a region breakdown showing finding counts per region, and a Quick Wins section listing the top 5 items that can be fixed immediately for the biggest impact
- **One tab per pillar** (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability): Each tab contains a pillar rating, a findings table with columns for Severity, Region, Resource, Description, Remediation, and Well-Architected Reference, and a summary of checks that passed. The findings table should be filterable by both region and severity using dropdown filters above the table.
- **Data Analytics Lens tab** (only when the lens is in scope): A dedicated tab for Data Analytics Lens findings, organized by the lens design principles (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability as they apply to analytics workloads). Uses the same findings table format, filters, and rating logic as the pillar tabs. Include a note at the top of the tab linking to the AWS Data Analytics Lens documentation: https://docs.aws.amazon.com/wellarchitected/latest/analytics-lens/analytics-lens.html
- **Generative AI Lens tab** (only when the lens is in scope): A dedicated tab for Generative AI Lens findings, organized by the lens focus areas (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability as they apply to generative AI workloads). Uses the same findings table format, filters, and rating logic as the pillar tabs. Include a note at the top of the tab linking to the AWS Generative AI Lens documentation: https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/generative-ai-lens.html

#### HTML Report Structure and Visual Design

The report must follow the Cloudscape Design System visual language (the design system used by the AWS Management Console). Since this is a self-contained HTML file with no external dependencies, replicate the Cloudscape look and feel using inline CSS. Do not import Cloudscape React components or any external library.

**Cloudscape design tokens to replicate:**

- Font family: `"Amazon Ember", "Helvetica Neue", Roboto, Arial, sans-serif`
- Base background: `#f2f3f3` (page background), `#ffffff` (container/card background)
- Text colors: `#000716` (primary text), `#5f6b7a` (secondary text)
- Border color: `#e9ebed`
- Border radius: `8px` for containers, `4px` for buttons and inputs
- Primary accent: `#0972d3` (links, active tab indicator)
- Focus ring: `0 0 0 2px #0972d3` outline on interactive elements

**Layout:**

- Top navigation bar: dark background (`#0f1b2a`), white text, report title on the left, account ID and timestamp on the right. This mimics the Cloudscape top navigation component.
- Tab navigation: horizontal tabs below the header, styled as Cloudscape Tabs. Active tab has a `2px` bottom border in `#0972d3` and bold text. Inactive tabs use `#5f6b7a` text. Tab bar has a bottom border of `1px solid #e9ebed`.
- Content area: white container cards with `8px` border-radius, `1px solid #e9ebed` border, and `20px` padding. Use `box-shadow: 0 1px 2px rgba(0, 7, 22, 0.12)` for subtle elevation.
- Spacing: `20px` gap between cards, `20px` page margin.

**Components to replicate:**

- **Container/Card**: White background, rounded corners, subtle shadow, optional header with `font-size: 18px; font-weight: 700; color: #000716`.
- **Table**: Cloudscape-style table with `#fafafa` header row background, `#e9ebed` row borders, `14px` font size, `12px 16px` cell padding. Hover row highlight: `#f4f4f4`.
- **Badge/Status indicator**: Severity badges as inline pill-shaped spans with `border-radius: 4px`, `padding: 2px 8px`, `font-size: 12px`, `font-weight: 700`. Colors:
  - Critical: background `#d91515`, text `#ffffff`
  - High: background `#f89256`, text `#000716`
  - Medium: background `#f2cd54`, text `#000716`
  - Low: background `#0972d3`, text `#ffffff`
  - Informational: background `#e9ebed`, text `#5f6b7a`
- **Pillar score cards** (Executive Summary): Grid of cards, each showing pillar name, rating badge, and finding counts. Use CSS Grid with `repeat(auto-fill, minmax(280px, 1fr))`.
- **Filter dropdowns**: Styled as Cloudscape Select components. `border: 1px solid #7d8998`, `border-radius: 4px`, `padding: 6px 12px`, `font-size: 14px`. Place region and severity filters side by side above each findings table.
- **Posture rating banner**: Large banner in the Executive Summary showing the overall rating. Use the severity badge colors for the rating background.

**Tabs:**

- Use a tabbed navigation layout. Clicking a tab shows that pillar's content and hides the others. The first tab visible on load should be the Executive Summary.
- Tab labels: Executive Summary, Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability. When the Data Analytics Lens is in scope, add a "Data Analytics" tab after Sustainability. When the Generative AI Lens is in scope, add a "Generative AI" tab after Data Analytics (or after Sustainability if Data Analytics is not in scope).

**Sorting and filtering:**

- Sort findings within each tab by severity (Critical first, Informational last).
- Each pillar tab has region and severity dropdown filters above the findings table.

**Responsive and print:**

- Include a print-friendly CSS `@media print` query that hides the tab bar, shows all pillar sections (and the Data Analytics Lens and Generative AI Lens sections when present) sequentially, and removes shadows and background colors for clean printing.
- Use responsive CSS so the report is readable on both desktop and tablet screens. Score card grid and filter row should wrap on narrow viewports.

#### Pillar Rating Logic

Assign each pillar a rating based on its worst finding:
- Any Critical finding -> Critical
- No Critical but any High -> Needs Improvement
- No Critical or High but any Medium -> Good
- Only Low or Informational findings -> Excellent
- Zero findings (no checks produced results, e.g., no resources of that type exist) -> Not Applicable. Display "N/A" instead of a severity-colored badge. In the Executive Summary, show the pillar card with a gray badge and a note: "No applicable resources found for this pillar."

The overall posture rating uses the same logic across all findings from all pillars. Pillars rated "Not Applicable" are excluded from the overall rating calculation. If all pillars are "Not Applicable," the overall rating is "Not Applicable."

### 4. Severity Definitions

- **Critical**: Active security vulnerability, data exposure risk, or service outage risk requiring immediate action
- **High**: Significant deviation from best practices that could lead to incidents
- **Medium**: Best practice not followed but no immediate risk
- **Low**: Minor improvement opportunity
- **Informational**: Observation or recommendation for future consideration

## Important Notes

- This power performs READ-ONLY operations. It does not modify any AWS resources.
- Scans may take several minutes depending on the number of resources and regions.
- Some checks require specific AWS services to be enabled (e.g., Security Hub, GuardDuty). The scan will note when a service is not enabled rather than failing.
- Findings are point-in-time assessments. Run scans regularly to track improvement.
- Always present findings with remediation guidance, not just problems.
