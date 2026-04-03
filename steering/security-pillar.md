---
inclusion: manual
---

# Security Pillar - Scan Checks

The Security pillar focuses on protecting information, systems, and assets while delivering business value through risk assessments and mitigation strategies.

## Data Dependencies

All data referenced below is collected during the Bulk Data Collection Phase (Phase 1) in POWER.md. Do NOT make additional AWS API calls for data listed here. Analyze the already-collected datasets.

**Collected datasets used by this pillar:**
- IAM: `get-account-summary`, `list-users`, `get-account-password-policy`, `list-policies --scope Local`, `get-credential-report`, per-user `list-mfa-devices` and `list-access-keys`, per-policy `get-policy-version`, `sso-admin list-instances`
- S3: per-bucket `get-bucket-encryption`, `get-public-access-block`
- EC2: `describe-instances`, `describe-volumes`, `describe-security-groups`, `describe-flow-logs`, `describe-vpcs`, `describe-route-tables`, `describe-network-acls`, `get-ebs-encryption-by-default`
- RDS: `describe-db-instances`
- Lambda: `list-functions`, per-function `get-policy`, `list-code-signing-configs`
- ECS: `list-clusters`, per-cluster `describe-services`
- EKS: `list-clusters`, per-cluster `describe-cluster`
- ECR: `describe-repositories`
- KMS: `list-keys`, per-key `get-key-rotation-status`
- Secrets Manager: `list-secrets`
- SNS: `list-topics`, per-topic `get-topic-attributes`
- SQS: `list-queues`, per-queue `get-queue-attributes`
- ELB: `describe-load-balancers`
- WAF: `list-web-acls`
- CloudTrail: `describe-trails`
- Config: `describe-configuration-recorders`
- CloudWatch: `describe-alarms`
- EventBridge: `list-rules`
- SSM: `describe-instance-information`
- Shield: `describe-subscription`
- CodePipeline: `list-pipelines`, per-pipeline `get-pipeline`
- CodeBuild: `list-projects`, `batch-get-projects`
- CodeGuru: `list-repository-associations`
- Security MCP: `CheckSecurityServices` (cached), `GetSecurityFindings` (reads cache), `CheckStorageEncryption` (cached), `CheckNetworkSecurity` (cached)

## Conditional Skip Rules

Skip the following checks when the corresponding resource type has zero results:

**Important**: Distinguish between "zero results" (the API call succeeded but returned no resources) and "data unavailable" (the API call failed due to AccessDeniedException, throttling, or other errors). When data is unavailable due to an API failure, do not skip the check silently. Instead, record an Informational finding stating: "Unable to assess [check name]: [resource type] data was not collected due to [error reason]. See the Error Handling rules in POWER.md." This ensures the report reflects gaps in coverage rather than falsely implying no issues exist.
- No EC2 instances -> skip IMDSv2 check, SSM-managed check
- No RDS instances -> skip RDS encryption check, RDS public accessibility check
- No Lambda functions -> skip Lambda public access check, Lambda code signing check
- No ECS clusters -> skip ECS task definition secrets check
- No EKS clusters -> skip EKS cluster security check
- No ECR repositories -> skip ECR image scanning check
- No KMS keys -> skip KMS key rotation check
- No Secrets Manager secrets -> skip Secrets Manager rotation check
- No SNS topics -> skip SNS topic encryption check
- No SQS queues -> skip SQS queue encryption check
- No ELB load balancers -> skip TLS enforcement check on load balancers
- No WAF web ACLs -> skip WAF check (but still flag as finding if public endpoints exist without WAF)
- No CodePipeline pipelines -> skip CI/CD security stages check
- No CodeBuild projects -> skip CodeBuild credential exposure check
- No CodeGuru associations -> skip CodeGuru check (but still flag as informational)

## Checks to Perform

### Identity and Access Management (SEC 1-3)

Analyze the collected IAM data:

1. **Root account usage**: From `get-account-summary` - Check if root account has access keys or recent usage
2. **MFA enforcement**: From `list-users` + per-user `list-mfa-devices` - Verify MFA is enabled for all IAM users
3. **Password policy**: From `get-account-password-policy` - Validate password complexity requirements
4. **Access key rotation**: From per-user `list-access-keys` - Flag keys older than 90 days
5. **Overly permissive policies**: From `list-policies` + per-policy `get-policy-version` - Flag policies with `*` actions or resources
6. **IAM roles vs users**: From `get-account-summary` - Count IAM users vs roles. Prefer roles for workloads.
7. **Unused credentials**: From `get-credential-report` - Flag unused credentials
8. **IAM Identity Center**: From `sso-admin list-instances` - Check if IAM Identity Center is configured as a centralized identity provider. Workforce users should federate via Identity Center rather than using IAM users with long-term credentials.
9. **IAM user count**: From `list-users` - Flag accounts with a high number of IAM users (indicates lack of federation). Best practice is to use IAM Identity Center and eliminate IAM users for human access.

### Detection (SEC 4)

Use cached results from `well-architected-security` MCP server (stored in Phase 1 with `store_in_context=True`):

1. **GuardDuty**: From cached `CheckSecurityServices` - Enabled and actively monitoring
2. **Security Hub**: From cached `CheckSecurityServices` - Enabled with standards activated
3. **Inspector**: From cached `CheckSecurityServices` - Enabled for vulnerability scanning
4. **IAM Access Analyzer**: From cached `CheckSecurityServices` - Configured for external access detection
5. **CloudTrail**: From collected `describe-trails` - Verify multi-region trail with log file validation
6. **Config**: From collected `describe-configuration-recorders` - Verify AWS Config is recording

Use `GetSecurityFindings` (reads from cached context) to retrieve active findings and categorize by severity.

### Infrastructure Protection (SEC 5-6)

Analyze collected EC2/VPC/networking data:

1. **VPC Flow Logs**: From collected `describe-flow-logs` - Verify flow logs enabled on all VPCs
2. **Security Groups**: From collected `describe-security-groups` - Flag groups with 0.0.0.0/0 ingress on sensitive ports (22, 3389, 3306, 5432)
3. **NACLs**: From collected `describe-network-acls` - Review for overly permissive rules
4. **Public subnets**: From collected `describe-route-tables` - Identify subnets with internet gateway routes
5. **WAF**: From collected `list-web-acls` - Check if WAF is protecting public endpoints
6. **EC2 IMDSv2 enforcement** (skip if no EC2 instances): From collected `describe-instances` - Flag instances not requiring IMDSv2 (HttpTokens != "required"). IMDSv1 is vulnerable to SSRF attacks.
7. **SSM-managed compute** (skip if no EC2 instances): From collected `describe-instance-information` - Verify EC2 instances are managed via SSM rather than direct SSH/RDP access
8. **Shield Advanced**: From collected `describe-subscription` - Check if Shield Advanced is enabled for DDoS protection on critical workloads

### Data Protection (SEC 7-9)

Analyze collected storage and encryption data:

1. **S3 encryption**: From collected per-bucket `get-bucket-encryption` - Verify encryption at rest
2. **S3 public access**: From collected per-bucket `get-public-access-block` - Verify public access is blocked
3. **EBS encryption**: From collected `describe-volumes` - Flag unencrypted volumes
4. **RDS encryption** (skip if no RDS instances): From collected `describe-db-instances` - Flag unencrypted databases
5. **KMS key rotation** (skip if no KMS keys): From collected per-key `get-key-rotation-status` - Verify automatic rotation
6. **TLS enforcement**: From collected ELB data check listeners for HTTPS; from collected S3 bucket policies check for ssl-only

Also use cached `CheckStorageEncryption` and `CheckNetworkSecurity` results from Phase 1.

### Incident Response (SEC 10)

Analyze collected monitoring data:

1. **SNS topics for alerts**: From collected `list-topics` - Verify alerting topics exist
2. **CloudWatch alarms**: From collected `describe-alarms` - Check for security-related alarms
3. **EventBridge rules**: From collected `list-rules` - Check for automated incident response rules

### Additional Security Checks

Analyze collected data with conditional skipping:

1. **Account-level EBS encryption default**: From collected `get-ebs-encryption-by-default` - Verify EBS encryption is enabled by default for the account
2. **Secrets Manager rotation** (skip if no secrets): From collected `list-secrets` - Flag secrets without automatic rotation configured
3. **SNS topic encryption** (skip if no SNS topics): From collected per-topic `get-topic-attributes` - Flag topics without KmsMasterKeyId set
4. **SQS queue encryption** (skip if no SQS queues): From collected per-queue `get-queue-attributes` - Flag queues without encryption
5. **Lambda public access** (skip if no Lambda functions): From collected per-function `get-policy` - Flag functions with resource-based policies allowing public invocation (`Principal: "*"`)
6. **ECR image scanning** (skip if no ECR repos): From collected `describe-repositories` - Verify image scan on push is enabled
7. **ECS task definition secrets** (skip if no ECS clusters): From collected ECS task definitions - Review for hardcoded credentials in environment variables instead of Secrets Manager references
8. **EKS cluster security** (skip if no EKS clusters): From collected per-cluster `describe-cluster` - Verify endpoint private access is enabled and public access is restricted
9. **RDS public accessibility** (skip if no RDS instances): From collected `describe-db-instances` - Flag instances with PubliclyAccessible set to true

### Application Security (SEC 11)

Analyze collected CI/CD data with conditional skipping:

1. **CodeGuru Reviewer** (skip if no CodeGuru associations): From collected `list-repository-associations` - Check if CodeGuru Reviewer is configured for automated code reviews
2. **Inspector code scanning**: From cached `CheckSecurityServices` - Check if Inspector is scanning Lambda functions and ECR images for code vulnerabilities
3. **CodePipeline security stages** (skip if no pipelines): From collected per-pipeline `get-pipeline` - Check if CI/CD pipelines include security testing stages (SAST/DAST)
4. **Lambda code signing** (skip if no Lambda functions): From collected `list-code-signing-configs` - Check if code signing is configured to validate software integrity before deployment
5. **CodeBuild credential exposure** (skip if no CodeBuild projects): From collected `batch-get-projects` - Flag CodeBuild projects with AWS_ACCESS_KEY_ID or AWS_SECRET_ACCESS_KEY in environment variables (should use IAM roles instead)

## Severity Mapping

- Root account with access keys: Critical
- MFA not enabled: Critical
- Public S3 buckets: Critical
- Security groups open to 0.0.0.0/0 on SSH/RDP: Critical
- GuardDuty/Security Hub not enabled: High
- CloudTrail not enabled: High
- Unencrypted EBS/RDS: High
- Access keys older than 90 days: Medium
- Missing VPC flow logs: Medium
- No WAF on public endpoints: Medium
- Password policy not meeting complexity requirements: Low
- Secrets without automatic rotation: Medium
- SNS topics without encryption: Medium
- SQS queues without encryption: Medium
- Lambda functions with public access: Critical
- ECR scan on push not enabled: Medium
- ECS task definitions with hardcoded credentials: High
- EKS public endpoint unrestricted: High
- RDS publicly accessible: Critical
- EBS encryption not enabled by default: Medium
- EC2 instances not requiring IMDSv2: High
- No Shield Advanced on critical workloads: Low
- No CodeGuru or Inspector code scanning: Low
- CI/CD pipelines without security testing stages: Medium
- IAM Identity Center not configured (using IAM users for workforce): High
- No Lambda code signing: Low
- CodeBuild projects with hardcoded credentials: High
