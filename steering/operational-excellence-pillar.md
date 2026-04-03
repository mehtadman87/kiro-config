---
inclusion: manual
---

# Operational Excellence Pillar - Scan Checks

The Operational Excellence pillar focuses on running and monitoring systems to deliver business value and continually improving supporting processes and procedures.

## Data Dependencies

All data referenced below is collected during the Bulk Data Collection Phase (Phase 1) in POWER.md. Do NOT make additional AWS API calls for data listed here. Analyze the already-collected datasets.

**Collected datasets used by this pillar:**
- Organizations: `describe-organization`
- Tags: `get-tag-keys`
- SSM: `describe-parameters`, `describe-instance-information`, `describe-patch-baselines`, `describe-maintenance-windows`
- SSM Automation: `list-documents --filters Key=DocumentType,Values=Automation` (add to Phase 1 operations batch if not present)
- CloudFormation: `list-stacks` (add to Phase 1 operations batch if not present, use `--stack-status-filter CREATE_COMPLETE UPDATE_COMPLETE`)
- Config: `describe-config-rules`, `describe-conformance-packs`
- CloudWatch: `describe-alarms`, `describe-insight-rules`
- CloudWatch Logs: `describe-log-groups`
- CloudTrail: `describe-trails`
- SNS: `list-subscriptions`
- Health: `describe-events` (add to Phase 1 operations batch if not present, use `--filter eventStatusCodes=open`)
- CodePipeline: `list-pipelines`
- CodeDeploy: `list-applications`, `list-deployment-configs`
- Synthetics: `describe-canaries`
- Application Insights: `list-applications`
- X-Ray: `get-groups`
- Resource Explorer: `list-indexes`
- EC2: `describe-instances` (for SSM cross-reference)

## Conditional Skip Rules

Skip the following checks when the corresponding resource type has zero results:

**Important**: Distinguish between "zero results" (the API call succeeded but returned no resources) and "data unavailable" (the API call failed due to AccessDeniedException, throttling, or other errors). When data is unavailable due to an API failure, do not skip the check silently. Instead, record an Informational finding stating: "Unable to assess [check name]: [resource type] data was not collected due to [error reason]. See the Error Handling rules in POWER.md." This ensures the report reflects gaps in coverage rather than falsely implying no issues exist.
- No EC2 instances -> skip SSM-managed instances check (but still check if SSM is configured generally)
- No CodePipeline pipelines -> skip CI/CD pipeline check (but still flag as finding)
- No CodeDeploy applications -> skip deployment strategy check (but still flag as finding)
- No CloudWatch alarms -> flag as finding, do not skip
- No Config rules -> flag as finding, do not skip
- No Synthetics canaries -> skip canary check (flag as informational)
- No Application Insights applications -> skip Application Insights check (flag as informational)
- No SSM patch baselines -> flag as finding, do not skip
- No SSM maintenance windows -> flag as finding, do not skip

## Checks to Perform

### Organization (OPS 1-3)

Analyze collected data:

1. **AWS Organizations**: From collected `describe-organization` - Check if multi-account strategy is in place
2. **Tagging**: From collected `get-tag-keys` - Verify consistent tagging for ownership, environment, and cost center
3. **SSM Parameter Store**: From collected `describe-parameters` - Check if configuration is externalized

### Prepare (OPS 4-7)

Analyze collected data:

1. **CloudFormation/CDK usage**: From collected `list-stacks` - Verify infrastructure is managed as code
2. **Systems Manager**: From collected `describe-instance-information` - Check if EC2 instances are managed by SSM
3. **SSM Patch Manager**: From collected `describe-patch-baselines` - Verify patch baselines exist
4. **Config rules**: From collected `describe-config-rules` - Verify compliance rules are defined
5. **CloudWatch Application Insights**: From collected `list-applications` (Application Insights) - Check if Application Insights is configured for automatic anomaly detection
6. **CloudWatch ServiceLens**: From collected `get-groups` (X-Ray) - Check if X-Ray tracing is integrated with CloudWatch for end-to-end service observability
7. **Deployment safeguards**: From collected `list-deployment-configs` (CodeDeploy) - Check if deployment configurations include automatic rollback and minimum healthy percentage settings

### Operate (OPS 7-9)

Analyze collected data:

1. **CloudWatch alarms**: From collected `describe-alarms` - Check for active alarms (filter state=ALARM in analysis)
2. **CloudWatch Logs**: From collected `describe-log-groups` - Verify application logging is centralized
3. **Log retention**: From collected `describe-log-groups` - Flag log groups with no retention policy (infinite retention = cost waste)
4. **CloudTrail**: From collected `describe-trails` - Verify API activity logging
5. **Health Dashboard**: From collected `describe-events` - Check for active AWS health events
6. **SNS notifications**: From collected `list-subscriptions` - Verify alarm notifications are configured

### Evolve (OPS 10-11)

1. **Trusted Advisor**: Use `awsapi` to call `aws support describe-trusted-advisor-checks --language en --region us-east-1` - Check if Trusted Advisor recommendations are being addressed. This is a live check. Note: Requires Business, Enterprise On-Ramp, or Enterprise Support plan.
2. **AWS Config conformance packs**: From collected `describe-conformance-packs` - Check for governance frameworks

### Additional Operational Excellence Checks

Analyze collected data with conditional skipping:

1. **CI/CD pipelines**: From collected `list-pipelines` (CodePipeline) - Check if CodePipeline is used for automated deployments
2. **CodeDeploy**: From collected `list-applications` (CodeDeploy) - Check if CodeDeploy is configured with deployment strategies (rolling, blue/green)
3. **SSM Automation runbooks**: From collected `list-documents` (SSM Automation) - Check if operational runbooks exist for common tasks
4. **CloudWatch Synthetics**: From collected `describe-canaries` - Check if canaries are monitoring critical endpoints
5. **CloudWatch Contributor Insights**: From collected `describe-insight-rules` - Check if Contributor Insights rules exist for identifying top contributors to operational issues
6. **Resource Explorer**: From collected `list-indexes` - Check if Resource Explorer is enabled for cross-region resource visibility
7. **SSM maintenance windows**: From collected `describe-maintenance-windows` - Verify maintenance windows are defined for patching and updates

## Severity Mapping

- No CloudTrail enabled: Critical
- No infrastructure as code: High
- EC2 instances not managed by SSM: High
- No CloudWatch alarms configured: High
- Log groups with no retention policy: Medium
- No tagging strategy: Medium
- No AWS Config rules: Medium
- No patch baselines: Medium
- No conformance packs: Low
- No centralized logging: Medium
- No CI/CD pipeline: High
- No deployment strategy (CodeDeploy): Medium
- No operational runbooks in SSM: Medium
- No CloudWatch Synthetics canaries: Low
- No SSM maintenance windows: Medium
- No Application Insights configured: Low
- No deployment rollback safeguards: Medium
