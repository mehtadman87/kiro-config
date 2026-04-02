# AWS Product Feature Requests (PFRs)

| ID | Title | Service | Status |
|---|---|---|---|
| PFR-001 | First-class `thinking` parameter in Strands Agents SDK BedrockConfig | Strands Agents SDK / Amazon Bedrock | Open |

---

### PFR-001: First-class `thinking` parameter in Strands Agents SDK BedrockConfig

**Date Identified:** 2026-04-02
**AWS Service:** Strands Agents SDK (open-source) / Amazon Bedrock
**Category:** API Gap
**Priority:** Medium
**Status:** Open

**Describe the Problem or Context.**
The Strands Agents SDK `BedrockConfig` TypedDict does not include a first-class `thinking` parameter for configuring Claude's extended thinking or adaptive thinking modes. Developers must use the generic `additional_request_fields` passthrough to pass the `thinking` configuration object to the Bedrock Converse API. This is undiscoverable without reading the AWS blog post or Bedrock API docs separately, and the passthrough pattern is not validated at the SDK level (no type checking, no defaults, no documentation in the config class itself).

**Describe the target user(s).**
Developers building agentic AI systems with Strands Agents SDK on Amazon Bedrock, particularly those using Claude Opus 4.6 or Sonnet 4.6 with extended/adaptive thinking for complex reasoning tasks.

**Describe the desired future state.**
`BedrockConfig` should include typed, documented parameters for thinking configuration:
- `thinking_type`: `"adaptive"` | `"enabled"` | `"disabled"` (default: `None` / not set)
- `thinking_budget_tokens`: `int` (optional, required when `thinking_type` is `"enabled"`)
- `thinking_effort`: `"max"` | `"high"` | `"medium"` | `"low"` (optional, for adaptive mode)

This would provide IDE autocompletion, type validation, and inline documentation, making the feature discoverable without external research.

**Known Workarounds.**
Use `additional_request_fields={"thinking": {"type": "adaptive", "budget_tokens": 10000}}` as a passthrough. This works but provides no type safety, no autocompletion, no SDK-level validation, and is not documented in the `BedrockConfig` attributes list. Developers must discover this pattern from the AWS blog post or by reading the Bedrock Converse API docs independently.

**Customer Specific Use Case.**
Building multi-agent orchestration systems where different agents require different thinking configurations (e.g., Opus 4.6 orchestrator with `effort: "max"` for deep reasoning, Sonnet 4.6 sub-agents with thinking disabled for fast extraction). The lack of a typed parameter makes it harder to manage these configurations consistently across agent definitions and increases the risk of misconfiguration.

**Research Validation.**
- Strands Agents SDK API docs confirm no `thinking` parameter in `BedrockConfig`: [strandsagents.com/docs/api/python/strands.models.bedrock/]
- AWS blog documents the `additional_request_fields` workaround: [aws.amazon.com/blogs/opensource/using-strands-agents-with-claude-4-interleaved-thinking/]
- Amazon Bedrock docs confirm the thinking API exists at the Converse API level: [docs.aws.amazon.com/bedrock/latest/userguide/claude-messages-adaptive-thinking.html]
- The Strands SDK GitHub repo (github.com/strands-agents/sdk-python) does not show any open PRs or issues for this feature as of April 2026.
