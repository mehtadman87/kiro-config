# Kiro Best Practices Boilerplate

A comprehensive collection of steering documents, agent hooks, MCP server configurations, and spec templates for Kiro IDE that enforces development best practices, automates quality checks, and streamlines workflows.

## Quick Start

### Option 1: Clone as Template

```bash
git clone https://github.com/mehtadman87/kiro-config.git your-project-name
cd your-project-name
rm -rf .git
git init
git add .
git commit -m "feat: initialize project with Kiro best practices"
```

### Option 2: Add to Existing Project (Recommended)

```bash
cd your-existing-project
mkdir -p .kiro
curl -L https://github.com/mehtadman87/kiro-config/archive/main.tar.gz | tar -xz
cp -r kiro-config-main/hooks kiro-config-main/settings kiro-config-main/specs kiro-config-main/steering .kiro/
rm -rf kiro-config-main
```

### Option 3: Manual Download

```bash
cd your-project
mkdir -p .kiro
git clone https://github.com/mehtadman87/kiro-config.git temp-kiro
cp -r temp-kiro/hooks temp-kiro/settings temp-kiro/specs temp-kiro/steering .kiro/
rm -rf temp-kiro
```

### Activation Requirements

After installation:

- Steering Documents: Automatically refresh and become active immediately — no restart needed.
- Hooks: Require restarting Kiro or reopening the project to become active.

---

## Steering Documents (23 files)

Steering files provide persistent context and guidelines to the Kiro agent across all interactions. They live in `.kiro/steering/` and use three inclusion modes:

- **Always**: Loaded into every agent interaction automatically.
- **File Match**: Loaded conditionally when a matching file pattern is read into context.
- **Manual**: Loaded on-demand via `#` context key in chat.

### Always Active (12 files)

These are loaded into every agent interaction:

| File | Description |
|---|---|
| `aws-cli-best-practices.md` | `--no-cli-pager` usage, output formatting, AWS integration patterns |
| `cdk-best-practices.md` | Project structure (projen), construct/stack/app patterns, testing, Lambda functions |
| `development-standards.md` | Dependency management, code quality, file management, documentation, version control |
| `email-drafting-style.md` | SA email formatting: subject lines, tone, structure, references, recommendations |
| `git-best-practices.md` | Conventional commits, branching strategy, repository management, security |
| `mcp-best-practices.md` | MCP server configuration, security, auto-approval, testing, agentic tool design |
| `pfr-tracking.md` | AWS Product Feature Request identification, research validation, documentation format |
| `research-validated-development.md` | Mandatory web search validation for specs, architecture, and dependency decisions |
| `security-best-practices.md` | Code security, dependency management, data protection, LLM guardrails (3-layer model), prompt injection defense (5-layer), agentic AI security |
| `technical-debt-management.md` | Debt categorization, severity guidelines, entry format, review cadence |
| `testing-best-practices.md` | Minimal verbosity execution, output management, filtering, CI/CD considerations |
| `typescript-best-practices.md` | Strict config, type safety, error handling, imports/exports, testing standards |

### Conditionally Loaded — File Match (3 files)

These activate when a matching file is read into context:

| File | Triggers On | Description |
|---|---|---|
| `docker-best-practices.md` | `Dockerfile*`, `docker-compose*`, `*.dockerfile` | Multi-stage builds, security, optimization, health checks |
| `python-best-practices.md` | `*.py` | PEP 8, virtual environments, type hints, testing |
| `react-best-practices.md` | `*.tsx`, `*.jsx`, `*react*` | Functional components, hooks, accessibility, state management |

### Manually Loaded — On-Demand (7 files)

Load these via `#` context key when working on specific topics:

| File | Description |
|---|---|
| `claude-model-best-practices.md` | Claude Opus 4.6 / Sonnet 4.6 model selection, adaptive thinking config, Strands SDK extended thinking passthrough, prompting principles, agentic coding patterns |
| `llm-agent-architecture.md` | Multi-agent architecture, MIT/Google scaling laws (~45% threshold, 17.2x error amplification), orchestration topologies (parallel, sequential, hierarchical, hybrid), agent count guidelines |
| `llm-accuracy-and-confidence.md` | Confidence estimation methods (verbalized, softmax, consistency, CoCoA, entropy), scoring framework with thresholds, CoT verification, hallucination reduction |
| `llm-cost-optimization.md` | Multi-model routing (40-60% savings), prompt caching (45-90% savings), semantic caching, token optimization, batch processing (50% savings), cost monitoring metrics |
| `llm-memory-and-context.md` | Semantic/episodic/procedural/user-preference memory types, intelligent decay, hierarchical memory allocation, context window management, NoLiMa benchmark findings |
| `llm-observability-and-hitl.md` | Outcome/trajectory/operational/reliability metrics, three-tier evaluation framework, LLM-as-judge, HITL escalation triggers, four HITL architecture patterns |
| `rag-best-practices.md` | Semantic chunking, hybrid search (BM25 + vector), reranking, query transformation, adaptive RAG, context compression, citation/grounding |
| `vidya-spark-generator.md` | Process guide for generating Vidya Spark inputs: customer context, problem statement, solution description, KPIs, and key features — grounded in deep research |

---

## Agent Hooks (21 files)

Hooks are event-driven automations that trigger agent actions on IDE events. They live in `.kiro/hooks/`.

### Automatic Hooks — File Events (12 hooks)

These trigger automatically when matching files are saved, created, or deleted:

| Hook | Trigger | File Patterns | Description |
|---|---|---|---|
| Auto Test on Save | `fileEdited` | `*.ts`, `*.js`, `*.tsx`, `*.jsx` | Runs related tests with minimal verbosity on code save |
| Lint and Format on Save | `fileEdited` | `*.ts`, `*.js`, `*.tsx`, `*.jsx`, `*.py`, `*.json` | Runs linter and formatter using project config |
| CDK Synth on Change | `fileEdited` | `src/**/*.ts`, `lib/**/*.ts`, `cdk.json`, `*Stack.ts`, `*Construct.ts` | Validates CDK code via `cdk synth`, checks security, runs diff |
| Security Scan on Dependency Change | `fileEdited` | `package.json`, `package-lock.json`, `yarn.lock`, `requirements.txt`, `poetry.lock`, `Pipfile.lock` | Runs security audit on dependency file changes |
| Validate Docker on Change | `fileEdited` | `Dockerfile*`, `docker-compose*.yml`, `*.dockerfile` | Validates syntax, security, best practices for Docker files |
| MCP Configuration Validation | `fileEdited` | `.kiro/settings/mcp.json` | Validates JSON structure, security of auto-approve settings |
| Environment File Validation | `fileEdited` | `.env*` | Checks for secrets, validates format, verifies `.gitignore` |
| API Schema Validation | `fileEdited` | `*.openapi.yml`, `*.swagger.yml`, `*.graphql`, `schema.json` | Validates schemas, generates TypeScript types, checks breaking changes |
| Spec Research Validation | `fileEdited` | `*requirements.md`, `*design.md`, `*tasks.md`, `*bugfix.md` | Web search validation when spec files are modified |
| Spec Creation Validation | `fileCreated` | `*requirements.md`, `*design.md`, `*tasks.md`, `*bugfix.md` | Web search validation when new spec files are created |

### Automatic Hooks — Agent Lifecycle (2 hooks)

These trigger on agent session events:

| Hook | Trigger | Description |
|---|---|---|
| Error Research Reminder | `agentStop` | Validates errors were researched via web search, checks for technical debt, validates architecture decisions |
| PFR Opportunity Identification | `agentStop` | Reviews session for AWS service gaps and documents validated PFRs in `PFRs.md` |

### Manual Hooks — Button Triggers (6 hooks)

On-demand tools available in the Kiro Agent Hooks panel:

| Hook | Button Text | Description |
|---|---|---|
| Commit Message Helper | Generate Commit Message | Analyzes git diff and generates conventional commit messages |
| README Spell Check | Spell Check README | Fixes spelling, grammar, and formatting in README files |
| Test MCP Servers | Test MCP Servers | Tests connectivity and tool functionality for all configured MCP servers |
| Dependency Update Check | Check Dependencies | Finds outdated packages, security vulnerabilities, suggests updates |
| Code Coverage Check | Check Coverage | Runs tests with coverage, identifies gaps below 80% threshold |
| Performance Analysis | Analyze Performance | Identifies performance anti-patterns, memory leaks, optimization opportunities |

### Optional Hooks — Disabled by Default (3 hooks)

Performance-sensitive hooks you can enable as needed by setting `"enabled": true`:

| Hook | Trigger | File Patterns | Description |
|---|---|---|---|
| Accessibility Audit | `fileEdited` | `*.tsx`, `*.jsx`, `*.html` | ARIA labels, keyboard accessibility, heading hierarchy, color contrast |
| Update Documentation | `fileEdited` | `src/**/*.ts`, `src/**/*.js`, `lib/**/*.py` | Updates associated docs, generates JSDoc/docstrings |
| Translation Update | `fileEdited` | `**/locales/en/**/*.json`, `**/i18n/en/**/*.json` | Syncs translation files when base language changes |

---

## MCP Server Configuration

Configured in `.kiro/settings/mcp.json`:

| Server | Package | Purpose |
|---|---|---|
| context7 | `context7-mcp-server@latest` | Dependency compatibility verification, library documentation lookup |
| aws-docs | `awslabs.aws-documentation-mcp-server@latest` | AWS documentation search, section reading, recommendations |

Both servers use `FASTMCP_LOG_LEVEL: "ERROR"` to reduce log noise. No tools are auto-approved by default.

---

## Specs

Spec templates and tracking files in `.kiro/specs/`:

| File | Description |
|---|---|
| `PFRs.md` | AWS Product Feature Request register — tracks identified service gaps with full PFR documentation format |
| `technical-debt.md` | Global technical debt register — tracks cross-cutting debt items with severity, impact, and remediation |

---

## Technology Support

This boilerplate includes best practices for:

- **Languages**: TypeScript, JavaScript, Python
- **Frameworks**: React, CDK, Docker
- **Tools**: Git, npm/yarn, pytest, ESLint, Prettier
- **Cloud**: AWS (CLI, CDK, Bedrock, Strands Agents SDK)
- **APIs**: OpenAPI/Swagger, GraphQL
- **AI/ML**: LLM agent architecture, RAG, multi-agent orchestration, cost optimization, observability
- **Testing**: Jest, pytest, coverage analysis
- **Protocols**: MCP (Model Context Protocol)

---

## Configuration

### Enabling/Disabling Hooks

Edit any `.kiro.hook` file and change the `enabled` field:

```json
{
  "enabled": true
}
```

### Customizing File Patterns

Modify the `patterns` array in hook files to match your project structure:

```json
{
  "when": {
    "type": "fileEdited",
    "patterns": [
      "src/**/*.ts",
      "lib/**/*.js"
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
```

Inclusion options:
- `always` — loaded into every interaction
- `fileMatch` with `fileMatchPattern` — loaded when matching files are in context
- `manual` — loaded on-demand via `#` context key

---

## License

MIT License — use and modify as needed.
