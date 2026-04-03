---
inclusion: manual
---

# Generative AI Lens - Scan Checks

The Generative AI Lens focuses on best practices for designing, deploying, and operating generative AI applications on AWS. It covers the generative AI lifecycle (scoping, model selection, customization, development, deployment, continuous improvement) across services like Amazon Bedrock, Amazon SageMaker AI, Amazon Q, Amazon OpenSearch Service (for vector stores), and supporting infrastructure.

Reference: https://docs.aws.amazon.com/wellarchitected/latest/generative-ai-lens/generative-ai-lens.html

## Data Dependencies

All data referenced below is collected during the Bulk Data Collection Phase (Phase 1) in POWER.md plus the additional generative AI-specific data collected in the supplemental collection step. Do NOT make additional AWS API calls for data listed here. Analyze the already-collected datasets.

**Collected datasets used by this lens (from base Phase 1):**
- IAM: `list-policies --scope Local`, per-policy `get-policy-version`, `list-users`
- EC2: `describe-instances`, `describe-security-groups`, `describe-vpcs`, `describe-vpc-endpoints`
- Lambda: `list-functions`, per-function `get-function-configuration`, `get-policy`
- CloudTrail: `describe-trails`
- CloudWatch: `describe-alarms`, `list-dashboards`
- Logs: `describe-log-groups`
- KMS: `list-keys`, per-key `get-key-rotation-status`
- SNS: `list-topics`, `list-subscriptions`
- Step Functions: `list-state-machines`
- CloudFormation: `list-stacks`
- Tags: `get-tag-keys`
- Cost Explorer: `get-cost-and-usage`
- S3: `list-buckets`, per-bucket `get-bucket-encryption`, `get-bucket-versioning`
- ECR: `describe-repositories`
- ECS: `list-clusters`
- EKS: `list-clusters`
- API Gateway: `get-rest-apis`

**Additional generative AI-specific datasets (collected in supplemental step):**
- Bedrock: `list-foundation-models` (global, us-east-1 only), `list-custom-models`, `list-model-customization-jobs`, `list-provisioned-model-throughputs`, `list-guardrails`, `list-knowledge-bases`, `list-agents`, `list-inference-profiles`, `list-evaluation-jobs`
- SageMaker: `list-endpoints`, `list-models`, `list-training-jobs --sort-by CreationTime --sort-order Descending --max-items 20`, `list-notebook-instances`
- OpenSearch Serverless: `list-collections`

Note: The base Phase 1 datasets (IAM policies, CloudTrail, CloudWatch, VPC endpoints, S3, Lambda, SNS, Step Functions, CloudFormation, Tags, Cost Explorer, CodePipeline, CodeBuild) are already collected before this lens runs. Do not re-collect them.

## Conditional Skip Rules

Skip the following checks when the corresponding resource type has zero results:

**Important**: Distinguish between "zero results" (the API call succeeded but returned no resources) and "data unavailable" (the API call failed due to AccessDeniedException, throttling, or other errors). When data is unavailable due to an API failure, do not skip the check silently. Instead, record an Informational finding stating: "Unable to assess [check name]: [resource type] data was not collected due to [error reason]. See the Error Handling rules in POWER.md." This ensures the report reflects gaps in coverage rather than falsely implying no issues exist.
- No Bedrock custom models -> skip custom model checks
- No Bedrock provisioned throughputs -> skip provisioned throughput checks
- No Bedrock guardrails -> skip guardrail checks (but flag as finding if Bedrock agents or knowledge bases exist)
- No Bedrock knowledge bases -> skip RAG-specific checks
- No Bedrock agents -> skip agent workflow and excessive agency checks
- No Bedrock inference profiles -> skip cross-region inference checks
- No SageMaker endpoints -> skip SageMaker endpoint checks
- No SageMaker models -> skip SageMaker model checks
- No SageMaker training jobs -> skip training job checks
- No SageMaker notebook instances -> skip notebook security checks
- No OpenSearch Serverless collections -> skip vector store checks
- No Bedrock resources AND no SageMaker endpoints -> skip entire lens (no generative AI workload detected)

### Early Termination

Before running the full supplemental data collection, execute a lightweight probe to determine if generative AI resources exist. Run these three calls first (single batch):
- `aws bedrock list-custom-models --max-items 1 --region *`
- `aws bedrock-agent list-agents --max-items 1 --region *`
- `aws sagemaker list-endpoints --max-items 1 --region *`

If all three return empty results, skip the entire Generative AI Lens (no supplemental collection, no checks, no report tab). This avoids 20+ unnecessary API calls.

## Assessment Execution Order

To minimize redundant analysis passes over the same collected data, group checks by the primary dataset they analyze rather than by lens focus area. The agent should process checks in this order:

1. **CloudTrail analysis pass** (single pass over `describe-trails`): Evaluate GENOPS01 check 1, GENOPS03 check 2, GENSEC03 check 1 together.
2. **CloudWatch analysis pass** (single pass over `describe-alarms` + `list-dashboards`): Evaluate GENOPS01 checks 2-3, GENOPS02 check 1, GENSEC03 check 2, GENPERF01 check 2 together.
3. **Guardrail analysis pass** (single pass over guardrail data): Evaluate GENSEC02 check 1, GENSEC04 check 1, GENCOST03 check 1 together.
4. **Agent analysis pass** (single pass over agent data): Evaluate GENSEC05 checks 1-2, GENREL03 check 3, GENCOST05 check 1, GENREL05 check 3 together.
5. **IAM policy analysis pass** (single pass over IAM policies): Evaluate GENSEC01 check 1, GENSEC01 check 5, GENSEC04 check 2 together.
6. **Remaining checks**: Process all other checks that reference unique datasets.

## Checks to Perform

### Operational Excellence: Model Performance Evaluation (GENOPS01)

Analyze collected Bedrock and SageMaker data:

1. **Model invocation logging** (skip if no Bedrock resources): From collected Bedrock data and `describe-trails` - Verify Bedrock model invocation logging is enabled via CloudTrail data events or Bedrock logging configuration to support performance evaluation and auditing. (GENOPS01-BP01)
2. **CloudWatch monitoring for generative AI**: From collected `describe-alarms` and `list-dashboards` - Check for CloudWatch alarms and dashboards monitoring generative AI metrics (Bedrock invocation latency, SageMaker endpoint invocation errors, token counts). (GENOPS01-BP01)
3. **SageMaker endpoint monitoring** (skip if no SageMaker endpoints): From collected `list-endpoints` - Verify SageMaker endpoints have CloudWatch alarms configured for InvocationErrors, ModelLatency, and OverheadLatency. (GENOPS01-BP01)

### Operational Excellence: Monitor and Manage Operational Health (GENOPS02)

1. **Log group retention for AI services**: From collected `describe-log-groups` - Check that log groups for generative AI services (`/aws/bedrock/`, `/aws/sagemaker/`) have appropriate retention policies set. (GENOPS02-BP01)
2. **SNS alerting for AI workloads**: From collected `list-topics` and `list-subscriptions` - Verify SNS topics with active subscriptions exist for alerting on generative AI operational issues. (GENOPS02-BP01)
3. **Bedrock provisioned throughput as rate management** (skip if no Bedrock provisioned throughputs): From collected `list-provisioned-model-throughputs` - Verify provisioned throughput is configured for production workloads to manage inference rate limits. Workloads relying solely on on-demand throughput are subject to default service quotas and throttling. (GENOPS02-BP03)

### Operational Excellence: Observability and Traceability (GENOPS03)

1. **Bedrock agent tracing** (skip if no Bedrock agents): From collected agent data - Check if agents have tracing enabled for debugging and monitoring agent workflow execution.
2. **CloudTrail logging**: From collected `describe-trails` - Verify CloudTrail is logging Bedrock and SageMaker API calls for traceability and audit.

### Operational Excellence: Lifecycle Management (GENOPS04)

1. **Infrastructure as code**: From collected `list-stacks` - Check if generative AI resources (Bedrock agents, knowledge bases, SageMaker endpoints) are deployed via CloudFormation or CDK stacks rather than manually provisioned. (GENOPS04-BP01)
2. **SageMaker model registry** (skip if no SageMaker models): From collected SageMaker data - Check if models are registered in SageMaker Model Registry for version control and lifecycle management. (GENOPS04-BP02)

### Operational Excellence: Model Customization (GENOPS05)

1. **Customization job usage** (skip if no Bedrock custom models and no SageMaker training jobs): From collected `list-model-customization-jobs` and `list-training-jobs` - Verify model customization jobs use managed services (Bedrock fine-tuning, SageMaker training) rather than self-managed infrastructure. (GENOPS05-BP01)

### Security: Endpoint Security (GENSEC01)

Analyze collected IAM, network, and endpoint data:

1. **Bedrock endpoint access control**: From collected IAM policies - Check for overly permissive IAM policies granting `bedrock:InvokeModel` or `bedrock:InvokeModelWithResponseStream` with `Resource: "*"`. Policies should scope access to specific model ARNs.
2. **VPC endpoints for Bedrock**: From collected `describe-vpc-endpoints` - Check if VPC endpoints exist for Bedrock (`com.amazonaws.<region>.bedrock-runtime`) to enable private network communication and avoid traversing the public internet.
3. **SageMaker endpoint network isolation** (skip if no SageMaker endpoints): From collected endpoint data - Verify SageMaker endpoints are deployed in VPCs with appropriate security group configurations.
4. **SageMaker notebook security** (skip if no SageMaker notebook instances): From collected `list-notebook-instances` - Flag notebook instances with direct internet access enabled. Notebooks should use VPC-only mode.
5. **Bedrock data store access** (skip if no Bedrock knowledge bases): From collected knowledge base data and IAM policies - Verify knowledge base IAM roles follow least privilege for accessing S3 data sources and OpenSearch vector stores.

### Security: Response Validation (GENSEC02)

1. **Bedrock Guardrails** (skip if no Bedrock agents and no Bedrock knowledge bases): From collected `list-guardrails` - Verify Bedrock Guardrails are configured to filter harmful, biased, or factually incorrect model responses. If agents or knowledge bases exist but no guardrails are defined, flag as finding.

### Security: Event Monitoring (GENSEC03)

1. **CloudTrail data plane logging for Bedrock**: From collected `describe-trails` - Verify CloudTrail is configured to log Bedrock data events (model invocations) for security monitoring and audit, not just management events.
2. **CloudWatch alarms for security events**: From collected `describe-alarms` - Check for alarms monitoring unauthorized access attempts or throttling on Bedrock and SageMaker APIs.

### Security: Prompt Security (GENSEC04)

1. **Guardrails for prompt injection** (skip if no Bedrock guardrails): From collected `list-guardrails` - Verify guardrails include content filters and denied topic policies that help mitigate prompt injection attacks. (GENSEC04-BP02)
2. **Secure prompt catalog**: From collected Bedrock data and IAM policies - If Bedrock Prompt Management is used, verify that prompt templates are access-controlled via IAM policies with least privilege. Prompt catalogs should restrict who can create, modify, or delete prompt templates. (GENSEC04-BP01)

### Security: Excessive Agency (GENSEC05)

1. **Agent action group permissions** (skip if no Bedrock agents): From collected agent data and IAM policies - Verify Bedrock agent execution roles follow least privilege. Flag agents with overly broad IAM permissions (e.g., `s3:*`, `dynamodb:*`) that could enable excessive agency.
2. **Agent guardrail association** (skip if no Bedrock agents): From collected agent and guardrail data - Verify Bedrock agents have guardrails associated to constrain their behavior and prevent unintended actions.

### Security: Data Poisoning (GENSEC06)

1. **Training data encryption** (skip if no SageMaker training jobs and no Bedrock custom models): From collected S3 bucket encryption data and KMS keys - Verify S3 buckets used for model training or fine-tuning data have encryption at rest enabled with KMS.
2. **S3 versioning for training data** (skip if no SageMaker training jobs and no Bedrock custom models): From collected per-bucket `get-bucket-versioning` - Verify versioning is enabled on S3 buckets storing training data to detect unauthorized modifications.

### Reliability: Throughput Management (GENREL01)

1. **Bedrock provisioned throughput** (skip if no Bedrock provisioned throughputs): From collected `list-provisioned-model-throughputs` - Review provisioned throughput configurations for production workloads. Flag production workloads relying solely on on-demand throughput without provisioned capacity for predictable latency.
2. **Bedrock inference profiles** (skip if no Bedrock inference profiles): From collected `list-inference-profiles` - Check if cross-region inference profiles are configured for distributing inference load and improving availability.

### Reliability: Network and Communication (GENREL02)

1. **VPC endpoint resilience**: From collected `describe-vpc-endpoints` - Verify VPC endpoints for Bedrock and SageMaker are configured across multiple availability zones for high availability.

### Reliability: Error Handling and Recovery (GENREL03)

1. **Step Functions for orchestration**: From collected `list-state-machines` - Check if Step Functions state machines are used to orchestrate generative AI workflows with built-in retry logic and error handling. (GENREL03-BP01)
2. **SNS dead-letter alerting**: From collected `list-topics` - Verify alerting mechanisms exist for failed model invocations or agent workflow failures. (GENREL03-BP01)
3. **Agent timeout mechanisms** (skip if no Bedrock agents): From collected agent data - Verify Bedrock agents have timeout configurations to prevent indefinite execution loops. Check if agent idle session timeout and maximum iteration limits are configured. (GENREL03-BP02)

### Reliability: Prompt Management (GENREL04)

1. **Bedrock prompt management** (skip if no Bedrock resources): From collected Bedrock data - Check if Bedrock Prompt Management is used for versioning and managing prompt templates rather than hardcoding prompts in application code. (GENREL04-BP01)
2. **Model catalog**: From collected Bedrock and SageMaker data - Check if a model catalog or registry is maintained to track which foundation models are approved, their versions, and their intended use cases. For SageMaker, check SageMaker Model Registry usage. For Bedrock, check if model access permissions are scoped to approved models only. (GENREL04-BP02)

### Reliability: Distributed Availability (GENREL05)

1. **Multi-region model availability**: From collected Bedrock and SageMaker data - Check if generative AI workloads are deployed across multiple regions for disaster recovery. Flag single-region deployments for production workloads. (GENREL05-BP01)
2. **Knowledge base replication** (skip if no Bedrock knowledge bases): From collected knowledge base data - Check if knowledge base data sources (S3, OpenSearch) have cross-region replication configured. (GENREL05-BP02)
3. **Agent multi-region availability** (skip if no Bedrock agents): From collected agent data - Check if Bedrock agents and their dependent resources (action groups, knowledge bases, Lambda functions) are available across all required regions of availability. (GENREL05-BP03)

### Reliability: Distributed Compute Tasks (GENREL06)

1. **Training job fault tolerance** (skip if no SageMaker training jobs): From collected `list-training-jobs` - Verify SageMaker training jobs use managed spot training with checkpointing enabled for cost-effective fault tolerance. Check if training jobs have appropriate retry configurations and checkpoint storage in S3. (GENREL06-BP01)

### Performance Efficiency: Establish Performance Evaluation (GENPERF01)

1. **Model evaluation jobs** (skip if no Bedrock resources): From collected `list-evaluation-jobs` - Check if Bedrock model evaluation jobs exist, indicating ground truth datasets are defined and periodic model performance evaluation is being conducted. (GENPERF01-BP01)
2. **Performance metrics collection**: From collected CloudWatch alarms, dashboards, and log groups - Verify performance metrics specific to generative AI (invocation latency, token throughput, error rates) are being collected and tracked over time. (GENPERF01-BP02)

### Performance Efficiency: Maintaining Model Performance (GENPERF02)

1. **SageMaker endpoint instance types** (skip if no SageMaker endpoints): From collected endpoint data - Review endpoint instance types. Flag endpoints using non-accelerated instances (no GPU/Inferentia) for large model inference workloads. (GENPERF02-BP03)
2. **Bedrock provisioned throughput for latency** (skip if no Bedrock provisioned throughputs): From collected provisioned throughput data - Verify latency-sensitive production workloads use provisioned throughput rather than on-demand for consistent performance. (GENPERF02-BP01)
3. **Model customization for performance** (skip if no Bedrock custom models): From collected `list-custom-models` - Check if fine-tuned or distilled models exist for specific tasks rather than relying solely on general-purpose foundation models. The existence of custom models indicates the team is optimizing model selection for their use case. (GENPERF02-BP03)

### Performance Efficiency: Compute Optimization (GENPERF03)

1. **SageMaker inference optimization** (skip if no SageMaker endpoints): From collected endpoint data - Check if SageMaker endpoints use inference-optimized instance types (ml.inf, ml.g5, ml.p4) for model hosting.
2. **Auto scaling for SageMaker endpoints** (skip if no SageMaker endpoints): From collected endpoint data - Verify SageMaker endpoints have auto scaling policies configured to handle variable inference traffic.

### Performance Efficiency: Vector Store Optimization (GENPERF04)

1. **OpenSearch Serverless for vector search** (skip if no OpenSearch Serverless collections): From collected `list-collections` - Verify OpenSearch Serverless collections used as vector stores are configured with appropriate capacity settings.

### Cost Optimization: Model Selection and Pricing (GENCOST01-02)

1. **Provisioned vs on-demand throughput** (skip if no Bedrock provisioned throughputs): From collected `list-provisioned-model-throughputs` and cost data - Review provisioned throughput commitments. Flag over-provisioned throughput that may be wasting spend.
2. **SageMaker endpoint right-sizing** (skip if no SageMaker endpoints): From collected endpoint data - Flag SageMaker endpoints that may be over-provisioned based on instance type relative to model size.
3. **Cost allocation tagging**: From collected `get-tag-keys` - Verify generative AI resources are tagged with cost allocation tags for financial accountability and chargeback.

### Cost Optimization: Cost-Aware Prompting (GENCOST03)

1. **Bedrock Guardrails for cost filtering** (skip if no Bedrock guardrails): From collected guardrail data - Check if guardrails include content filters that can reject irrelevant or abusive prompts before they consume model tokens. (GENCOST03-BP04)
2. **Response length controls** (skip if no Bedrock guardrails): From collected per-guardrail `get-guardrail` data - Check if guardrails include output token limits or word count limits to prevent unnecessarily long model responses that increase costs. (GENCOST03-BP02)

### Cost Optimization: Cost-Informed Vector Stores (GENCOST04)

1. **Vector dimension optimization** (skip if no Bedrock knowledge bases): From collected per-knowledge-base `get-knowledge-base` data - Check the embedding model configuration. Flag knowledge bases using high-dimensional embeddings (e.g., 1024-dim) when lower dimensions (256 or 512) would suffice for the use case, reducing storage and compute costs in the vector store. (GENCOST04-BP01)

### Cost Optimization: Cost-Informed Agents (GENCOST05)

1. **Agent workflow complexity** (skip if no Bedrock agents): From collected agent data - Review agent configurations for potential runaway costs. Flag agents without maximum iteration limits or timeout configurations.

### Sustainability (GENSUS01-03)

1. **Serverless and managed services**: From collected Bedrock and SageMaker data - Check if the workload uses managed serverless options (Bedrock on-demand, SageMaker Serverless Inference) where appropriate to minimize idle resource consumption. (GENSUS01-BP01)
2. **Auto scaling for SageMaker** (skip if no SageMaker endpoints): From collected endpoint data - Verify endpoints scale down during low-demand periods to reduce energy consumption. (GENSUS01-BP01)
3. **Efficient model customization techniques** (skip if no Bedrock custom models): From collected `list-model-customization-jobs` - Check the `customizationType` field. Verify efficient techniques are used (FINE_TUNING, DISTILLATION) rather than CONTINUED_PRE_TRAINING for tasks that don't require it, as fine-tuning and distillation consume fewer compute resources. (GENSUS01-BP02)

## Severity Mapping

- Bedrock IAM policies with `InvokeModel` on `Resource: "*"`: High
- No VPC endpoints for Bedrock in VPC-based workloads: Medium
- SageMaker endpoints not in VPC: High
- SageMaker notebooks with direct internet access: High
- No Bedrock Guardrails configured (with agents or knowledge bases present): Critical
- No CloudTrail data event logging for Bedrock: Medium
- Guardrails without content filters for prompt injection: High
- Prompt catalog without access controls: Medium
- Bedrock agents with overly broad IAM permissions: Critical
- Bedrock agents without associated guardrails: Critical
- Training data S3 buckets without encryption: High
- Training data S3 buckets without versioning: Medium
- No provisioned throughput for production workloads: Medium
- No cross-region inference profiles for production: Low
- No Step Functions orchestration with retry logic: Low
- Agents without timeout mechanisms: High
- Single-region deployment for production generative AI: Medium
- Agent capabilities not available across required regions: Medium
- No model invocation logging enabled: High
- No CloudWatch alarms for generative AI metrics: Medium
- No CloudWatch dashboards for generative AI monitoring: Low
- AI service log groups without retention policies: Low
- Generative AI resources not deployed via IaC: Low
- No model catalog or registry: Low
- SageMaker endpoints without auto scaling: Medium
- SageMaker endpoints using non-accelerated instances for large models: Medium
- No performance metrics collection for generative AI: Medium
- No model evaluation jobs configured: Medium
- No custom models for task-specific optimization: Low
- No provisioned throughput for rate management: Medium
- Over-provisioned Bedrock throughput: Low
- Over-provisioned SageMaker endpoint instances: Low
- Generative AI resources missing cost allocation tags: Medium
- No response length controls in guardrails: Low
- Knowledge bases using unnecessarily high-dimensional embeddings: Low
- No prompt management (hardcoded prompts): Low
- Agents without iteration limits or timeouts: High
- Knowledge base IAM roles with overly broad permissions: High
- Training jobs without checkpointing or spot training: Low
- Continued pre-training used where fine-tuning or distillation would suffice: Low
