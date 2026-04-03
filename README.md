# Kiro Best Practices Boilerplate

A comprehensive collection of steering documents, agent hooks, MCP server configurations, and spec templates for Kiro IDE that enforces development best practices, automates quality checks, and streamlines workflows.

## 🚀 Quick Start

### Option 1: Clone as Template
```bash
# Clone this repository as a template for your new project
git clone https://github.com/mehtadman87/kiro-config.git your-project-name
cd your-project-name
rm -rf .git
git init
git add .
git commit -m "feat: initialize project with Kiro best practices"
```

### Option 2: Add to Existing Project (Recommended)
```bash
# Add only the .kiro directory to your existing project
cd your-existing-project
curl -L https://github.com/mehtadman87/kiro-config/archive/main.tar.gz | tar -xz
cp -r kiro-config-main/.kiro/. .kiro/
rm -rf kiro-config-main
```

### Option 3: Manual Download
```bash
# Download and extract only .kiro directory
cd your-project
git clone https://github.com/mehtadman87/kiro-config.git temp-kiro
cp -r temp-kiro/.kiro/. .kiro/
rm -rf temp-kiro
```

## ⚠️ Important: Activation Requirements

After installation:

- **🎯 Steering Documents**: Automatically refresh and become active immediately — no restart needed
- **🔄 Hooks**: Require **restarting Kiro** or **reopening the project** to become active

💡 **Tip**: After installation, restart Kiro to ensure all hooks are properly loaded and functional.

## 📋 What's Included

### 🎯 Steering Documents (31 files)

Steering files provide persistent context and guidelines to the Kiro agent across all interactions. They live in `.kiro/steering/` and use four inclusion modes:

- **Always** — Loaded into every agent interaction automatically
- **Auto** — Loaded when your request matches the file's description
- **File Match** — Loaded when a matching file pattern is read into context
- **Manual** — Loaded on-demand via `#` context key or `/` slash command in chat

#### Always Active (6 files)

Core standards loaded into every interaction:

- **[Development Standards](.kiro/steering/development-standards.md)** — Dependency management, code quality, file management, documentation, version control
- **[Git Best Practices](.kiro/steering/git-best-practices.md)** — Conventional commits, branching strategy, repository management, security
- **[Security Best Practices](.kiro/steering/security-best-practices.md)** — Code security, dependency management, data protection, LLM guardrails (3-layer model), prompt injection defense (5-layer), agentic AI security
- **[Technical Debt Management](.kiro/steering/technical-debt-management.md)** — Two-file tracking system (open/closed), severity guidelines, entry format, review cadence
- **[Testing Best Practices](.kiro/steering/testing-best-practices.md)** — Minimal verbosity execution, output management, filtering, CI/CD considerations
- **[TypeScript Best Practices](.kiro/steering/typescript-best-practices.md)** — Strict config, type safety, error handling, imports/exports, testing standards

#### Auto-Activated (4 files)

Loaded automatically when your request matches the description:

- **[AWS CLI Best Practices](.kiro/steering/aws-cli-best-practices.md)** — `--no-cli-pager` usage, output formatting, AWS integration patterns
- **[MCP Best Practices](.kiro/steering/mcp-best-practices.md)** — MCP server configuration, security, auto-approval, testing, agentic tool design
- **[PFR Tracking](.kiro/steering/pfr-tracking.md)** — AWS Product Feature Request identification, research validation, documentation format
- **[Research-Validated Development](.kiro/steering/research-validated-development.md)** — Mandatory web search validation for specs, architecture, and dependency decisions

#### Conditionally Loaded — File Match (4 files)

Activate when a matching file is read into context:

- **[CDK Best Practices](.kiro/steering/cdk-best-practices.md)** — Project structure (projen), construct/stack/app patterns, testing, Lambda functions *(triggers on `*Stack.ts`, `*Construct.ts`, `cdk.json`)*
- **[Docker Best Practices](.kiro/steering/docker-best-practices.md)** — Multi-stage builds, security, optimization, health checks *(triggers on `Dockerfile*`, `docker-compose*`)*
- **[Python Best Practices](.kiro/steering/python-best-practices.md)** — PEP 8, virtual environments, type hints, testing *(triggers on `*.py`)*
- **[React Best Practices](.kiro/steering/react-best-practices.md)** — Functional components, hooks, accessibility, state management *(triggers on `*.tsx`, `*.jsx`)*

#### Manually Loaded — On-Demand (17 files)

Load via `#` context key or `/` slash command when working on specific topics:

**AWS Well-Architected Scanner** (8 files — loaded automatically by the Well-Architected Scanner power during scans):

- **[Security Pillar](.kiro/steering/security-pillar.md)** — Security scan checks: IAM, encryption, network security, data protection, incident response
- **[Reliability Pillar](.kiro/steering/reliability-pillar.md)** — Reliability scan checks: fault tolerance, recovery, change management, multi-AZ/region
- **[Performance Efficiency Pillar](.kiro/steering/performance-efficiency-pillar.md)** — Performance scan checks: compute, storage, database, networking optimization
- **[Cost Optimization Pillar](.kiro/steering/cost-optimization-pillar.md)** — Cost scan checks: right-sizing, reserved capacity, unused resources, budgets
- **[Operational Excellence Pillar](.kiro/steering/operational-excellence-pillar.md)** — Ops scan checks: monitoring, alerting, automation, CI/CD, runbooks
- **[Sustainability Pillar](.kiro/steering/sustainability-pillar.md)** — Sustainability scan checks: resource utilization, managed services, data lifecycle
- **[Data Analytics Lens](.kiro/steering/data-analytics-lens.md)** — Analytics lens checks: Glue, Redshift, Athena, EMR, Kinesis, MSK, Lake Formation, QuickSight, OpenSearch
- **[Generative AI Lens](.kiro/steering/generative-ai-lens.md)** — GenAI lens checks: Bedrock, SageMaker, guardrails, knowledge bases, agents, vector stores

**General** (9 files):

- **[Claude Model Best Practices](.kiro/steering/claude-model-best-practices.md)** — Claude Opus 4.6 / Sonnet 4.6 model selection, adaptive thinking config, Strands SDK extended thinking passthrough, prompting principles, agentic coding patterns
- **[Email Drafting Style](.kiro/steering/email-drafting-style.md)** — SA email formatting: subject lines, tone, structure, references, recommendations
- **[LLM Agent Architecture](.kiro/steering/llm-agent-architecture.md)** — Multi-agent architecture, MIT/Google scaling laws (~45% threshold, 17.2x error amplification), orchestration topologies, agent count guidelines
- **[LLM Accuracy & Confidence](.kiro/steering/llm-accuracy-and-confidence.md)** — Confidence estimation methods (verbalized, softmax, consistency, CoCoA, entropy), scoring framework, CoT verification, hallucination reduction
- **[LLM Cost Optimization](.kiro/steering/llm-cost-optimization.md)** — Multi-model routing (40-60% savings), prompt caching (45-90% savings), semantic caching, token optimization, batch processing
- **[LLM Memory & Context](.kiro/steering/llm-memory-and-context.md)** — Semantic/episodic/procedural/user-preference memory types, intelligent decay, context window management, NoLiMa benchmark findings
- **[LLM Observability & HITL](.kiro/steering/llm-observability-and-hitl.md)** — Outcome/trajectory/operational/reliability metrics, three-tier evaluation framework, LLM-as-judge, HITL escalation triggers
- **[RAG Best Practices](.kiro/steering/rag-best-practices.md)** — Semantic chunking, hybrid search (BM25 + vector), reranking, query transformation, adaptive RAG, context compression
- **[Vidya Spark Generator](.kiro/steering/vidya-spark-generator.md)** — Process guide for generating Vidya Spark inputs: customer context, problem statement, solution description, KPIs, key features

### 🔄 Agent Hooks (23 files)

Hooks are event-driven automations that trigger agent actions on IDE events. They live in `.kiro/hooks/`.

#### Automatic Hooks — File Events (10 hooks)

Quality checks that run automatically when you save, create, or delete files:

- **[Auto Test and Lint on Save](.kiro/hooks/auto-test-on-save.kiro.hook)** — Runs linter, formatter, and related tests with minimal verbosity on JS/TS save
- **[Lint and Format on Save](.kiro/hooks/lint-and-format-on-save.kiro.hook)** — Runs linter and formatter for Python and JSON files
- **[CDK Synth on Change](.kiro/hooks/cdk-synth-on-change.kiro.hook)** — Validates CDK code via `cdk synth`, checks security, runs diff
- **[Security Scan on Dependencies](.kiro/hooks/security-scan-on-dependency-change.kiro.hook)** — Runs `npm audit` / `yarn audit` / `pip-audit` on dependency file changes
- **[Validate Docker on Change](.kiro/hooks/validate-docker-on-change.kiro.hook)** — Checks Docker files for best practices and security
- **[MCP Config Validation](.kiro/hooks/mcp-config-validation.kiro.hook)** — Validates MCP server JSON structure and security of auto-approve settings
- **[Environment File Validation](.kiro/hooks/env-file-validation.kiro.hook)** — Checks `.env` files for secrets, validates format, verifies `.gitignore`
- **[API Schema Validation](.kiro/hooks/api-schema-validation.kiro.hook)** — Validates OpenAPI/GraphQL schemas and generates TypeScript types
- **[Spec Research Validation](.kiro/hooks/spec-research-validation.kiro.hook)** — Web search validation when spec files are modified
- **[Spec Creation Validation](.kiro/hooks/spec-creation-validation.kiro.hook)** — Web search validation when new spec files are created

#### Automatic Hooks — Agent Lifecycle (1 hook)

- **[Session Completion Review](.kiro/hooks/error-research-reminder.kiro.hook)** — Validates errors were researched via web search, checks for technical debt, validates architecture decisions, and identifies PFR opportunities

#### 🔘 Manual Hooks — Button Triggers (9 hooks)

On-demand tools available in the Kiro Agent Hooks panel:

**AWS Well-Architected Scanner** (3 hooks):

- **[Well-Architected Review](.kiro/hooks/well-architected-review.kiro.hook)** — Full six-pillar scan with optional Data Analytics and Generative AI lens assessments
- **[Data Analytics Lens Review](.kiro/hooks/data-analytics-lens-review.kiro.hook)** — Analytics-focused assessment for Glue, Redshift, Athena, EMR, Kinesis, MSK, Lake Formation, QuickSight, OpenSearch
- **[Generative AI Lens Review](.kiro/hooks/generative-ai-lens-review.kiro.hook)** — GenAI-focused assessment for Bedrock, SageMaker, OpenSearch Serverless

**General** (6 hooks):

- **[Commit Message Helper](.kiro/hooks/commit-message-helper.kiro.hook)** — Analyzes git diff and generates conventional commit messages
- **[README Spell Check](.kiro/hooks/readme-spell-check.kiro.hook)** — Fixes spelling, grammar, and formatting in README files
- **[Test MCP Servers](.kiro/hooks/mcp-server-test.kiro.hook)** — Tests connectivity and tool functionality for all configured MCP servers
- **[Dependency Update Check](.kiro/hooks/dependency-update-check.kiro.hook)** — Finds outdated packages, security vulnerabilities, suggests updates
- **[Code Coverage Check](.kiro/hooks/code-coverage-check.kiro.hook)** — Runs tests with coverage, identifies gaps below 80% threshold
- **[Performance Analysis](.kiro/hooks/performance-analysis.kiro.hook)** — Identifies performance anti-patterns, memory leaks, optimization opportunities

#### 🎛️ Optional Hooks — Disabled by Default (3 hooks)

Performance-sensitive hooks you can enable as needed:

- **[Accessibility Audit](.kiro/hooks/accessibility-audit.kiro.hook)** — ARIA labels, keyboard accessibility, heading hierarchy, color contrast
- **[Update Documentation](.kiro/hooks/update-documentation.kiro.hook)** — Updates associated docs, generates JSDoc/docstrings when code changes
- **[Translation Update](.kiro/hooks/translation-update.kiro.hook)** — Syncs translation files when base language changes

### 📡 MCP Server Configuration

Configured in `.kiro/settings/mcp.json`:

| Server | Package | Purpose |
|---|---|---|
| context7 | `context7-mcp-server@latest` | Dependency compatibility verification, library documentation lookup |
| aws-docs | `awslabs.aws-documentation-mcp-server@latest` | AWS documentation search, section reading, recommendations |
| well-architected-security | `awslabs.well-architected-security-mcp-server` | Security posture assessment, GuardDuty/Security Hub/Inspector status, storage encryption, network security checks |
| awsknowledge | `knowledge-mcp.global.api.aws` (HTTP) | AWS best practices documentation, Well-Architected Framework guidance, service guides |
| awsapi | `awslabs.aws-api-mcp-server@latest` | Execute read-only AWS CLI commands to inspect resource configurations across all pillars |

Both servers use `FASTMCP_LOG_LEVEL: "ERROR"` to reduce log noise. No tools are auto-approved by default.

### 🏗️ AWS Well-Architected Scanner Power

A Kiro Power that scans and validates AWS accounts against the [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) best practices across all six pillars, plus optional lens assessments.

**What it scans:**
- All six Well-Architected pillars: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability
- Optional Data Analytics Lens (Glue, Redshift, Athena, EMR, Kinesis, MSK, Lake Formation, QuickSight, OpenSearch)
- Optional Generative AI Lens (Bedrock, SageMaker, OpenSearch Serverless)

**Prerequisites:**
- AWS credentials with `ReadOnlyAccess` policy (verify with `aws sts get-caller-identity`)
- `uv`/`uvx` installed (verify with `uvx --version`)
- Python 3.10+

**Usage:**
- Click a hook in the Agent Hooks panel (Well-Architected Review, Data Analytics Lens Review, or Generative AI Lens Review)
- Or ask in chat: "Run a well-architected scan", "Scan only the security pillar in us-east-1", "Run a generative AI lens assessment"

**Output:** A self-contained HTML report with Cloudscape Design System styling, severity badges, region/severity filters, and remediation guidance. Read-only — no AWS resources are modified.

### 📑 Specs & Tracking

Spec templates and tracking files in `.kiro/specs/`:

| File | Description |
|---|---|
| `PFRs.md` | AWS Product Feature Request register — tracks identified service gaps with full PFR documentation format |
| `open-tech-debt.md` | Open technical debt register — tracks unresolved cross-cutting debt items with severity, impact, and remediation |
| `closed-tech-debt.md` | Closed technical debt register — resolved items moved here with date and resolution details |

## ⚙️ Configuration

### Enabling/Disabling Hooks
Edit any `.kiro.hook` file and change the `enabled` field:
```json
{
  "enabled": true,  // Change to false to disable
  "name": "Hook Name",
  // ... rest of configuration
}
```

### Customizing File Patterns
Modify the `patterns` array in hook files to match your project structure:
```json
{
  "when": {
    "type": "fileEdited",
    "patterns": [
      "src/**/*.ts",     // Your source files
      "lib/**/*.js",     // Your library files
      "**/*.custom"      // Your custom extensions
    ]
  }
}
```

### Adding Project-Specific Steering
Create additional steering documents in `.kiro/steering/`:
```markdown
---
title: Your Team Standards
inclusion: always
---

# Your Team-Specific Guidelines
- Team-specific coding standards
- Project-specific patterns
- Custom workflows
```

Inclusion options:
- `always` — loaded into every interaction
- `auto` — loaded when your request matches the file's description
- `fileMatch` with `fileMatchPattern` — loaded when matching files are in context
- `manual` — loaded on-demand via `#` context key or `/` slash command

## 🛠️ Technology Support

This boilerplate includes best practices for:

- **Languages**: TypeScript, JavaScript, Python
- **Frameworks**: React, CDK, Docker
- **Tools**: Git, npm/yarn, pytest, ESLint, Prettier
- **Cloud**: AWS (CLI, CDK, Bedrock, Strands Agents SDK, Well-Architected Framework)
- **APIs**: OpenAPI/Swagger, GraphQL
- **AI/ML**: LLM agent architecture, RAG, multi-agent orchestration, cost optimization, observability
- **Testing**: Jest, pytest, coverage analysis
- **Protocols**: MCP (Model Context Protocol)
- **Documentation**: Markdown, JSDoc, docstrings

## � MCP Integration

Includes best practices for Model Context Protocol servers:

- **Context7** — For dependency compatibility checking and library documentation
- **AWS Docs** — For current AWS documentation search and best practices
- **Well-Architected Security** — For security posture assessment, GuardDuty/Security Hub/Inspector status, storage encryption, and network security checks
- **AWS Knowledge** — For AWS best practices documentation and Well-Architected Framework guidance
- **AWS API** — For executing read-only AWS CLI commands to inspect resource configurations
- Proper configuration patterns, security, and testing workflows
- Agentic tool design principles (single-purpose tools, input validation, parallel calling)

## 🔒 Security Features

Built-in security practices:

- Dependency vulnerability scanning (automatic on package file changes)
- Environment file validation (no secrets in code)
- Docker security best practices
- AWS security patterns
- MCP server security configurations
- LLM guardrails (3-layer input/runtime/output model)
- Prompt injection defense (5-layer defense-in-depth)
- Agentic AI security (least privilege, sandboxing, secret management)

## 🎨 Customization Guide

### For Your Team
1. **Review all steering documents** — Modify for your team's standards
2. **Adjust hook sensitivity** — Enable/disable based on your workflow
3. **Update file patterns** — Match your project structure
4. **Add team-specific hooks** — Create custom automation for your needs

### For Your Project Type
- **Web Applications**: Enable accessibility audit, focus on React patterns
- **Infrastructure**: Enable CDK hooks, focus on AWS patterns
- **Libraries**: Enable documentation updates, focus on API patterns
- **Microservices**: Enable Docker validation, focus on testing patterns
- **AI/ML Projects**: Load LLM steering files via `#` for agent architecture, RAG, cost optimization

## 📖 Documentation

- [Steering Documents](.kiro/steering/) — Individual best practice guides
- [Hook Configurations](.kiro/hooks/) — All available automation hooks
- [Specs & Tracking](.kiro/specs/) — PFR register and technical debt tracking

## 🤝 Contributing

### Adding New Best Practices
1. Create steering document in `.kiro/steering/`
2. Add corresponding hook in `.kiro/hooks/` if automation is needed
3. Update this README
4. Test with a sample project

### Sharing Improvements
1. Fork this repository
2. Add your improvements
3. Submit pull request with description
4. Include examples of usage

## 📄 License

MIT License — Feel free to use this in your projects and modify as needed.

## 🙋‍♂️ Support

- **Issues**: Report bugs or request features via GitHub issues
- **Discussions**: Share your customizations and ask questions

---

## 🎯 Quick Verification

After setup, verify everything works:

1. **Check Steering**: AI responses should reference best practices
2. **Test Hooks**: Save a TypeScript file to trigger auto-test hook
3. **Manual Hooks**: Look for buttons in Kiro Agent Hooks panel
4. **MCP Integration**: Click "Test MCP Servers" button to verify connectivity

**Happy coding with consistent, high-quality development practices!** 🚀
