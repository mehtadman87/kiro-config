---
title: Claude Opus 4.6 & Sonnet 4.6 Best Practices
inclusion: manual
---

# Claude Opus 4.6 & Sonnet 4.6 Best Practices

Anthropic-specific prompting, configuration, and agentic coding patterns for Claude 4.6 models. Sourced from official Anthropic documentation and production-validated patterns.

Source: Anthropic official documentation [docs.anthropic.com/en/docs/build-with-claude/prompt-engineering/claude-4-best-practices]

---

## Model Selection

| Use Case | Recommended Model | Effort Setting |
|---|---|---|
| Long-horizon autonomous work, large code migrations, deep research | Opus 4.6 | high or max |
| Fast turnaround, cost-efficient coding, tool-heavy workflows | Sonnet 4.6 | medium |
| High-volume, latency-sensitive workloads | Sonnet 4.6 | low |
| Chat, content generation, classification | Sonnet 4.6 | low |
| Computer use agents | Sonnet 4.6 | high (adaptive) |

---

## Adaptive Thinking Configuration

Claude 4.6 models use adaptive thinking (`thinking: {type: "adaptive"}`) where Claude dynamically decides when and how much to think. This replaces the older `budget_tokens` approach.

- Adaptive thinking reliably outperforms extended thinking in internal evaluations
- Claude calibrates thinking based on the `effort` parameter and query complexity
- On easier queries, the model responds directly without thinking overhead
- Set `max_tokens` to 64k at medium or high effort to give the model room to think and act

**When thinking is excessive:**
Add: "When deciding how to approach a problem, choose an approach and commit to it. Avoid revisiting decisions unless you encounter new information that directly contradicts your reasoning."

**When thinking is insufficient:**
Add: "After receiving tool results, carefully reflect on their quality and determine optimal next steps before proceeding."

### Strands Agents Extended Thinking Passthrough (Bedrock)

Sources: [Strands Agents SDK — BedrockModel API](https://strandsagents.com/docs/api/python/strands.models.bedrock/); [Using Strands Agents with Claude 4 Interleaved Thinking (AWS Blog)](https://aws.amazon.com/blogs/opensource/using-strands-agents-with-claude-4-interleaved-thinking/); [Amazon Bedrock — Adaptive Thinking](https://docs.aws.amazon.com/bedrock/latest/userguide/claude-messages-adaptive-thinking.html); [Amazon Bedrock — Extended Thinking](https://docs.aws.amazon.com/bedrock/latest/userguide/claude-messages-extended-thinking.html)

Strands Agents SDK does not expose a first-class `thinking` parameter in its `BedrockConfig`. However, the SDK's `additional_request_fields` parameter maps directly to the Bedrock Converse API's `additionalModelRequestFields`, which accepts arbitrary model-specific fields — including the `thinking` configuration object. This is the documented passthrough mechanism for enabling extended thinking on Claude Opus 4.6 agents deployed via Strands.

**Adaptive thinking (recommended for agentic workflows):**

Claude dynamically decides how much to think based on task complexity. Automatically enables interleaved thinking between tool calls.

```python
from strands import Agent
from strands.models.bedrock import BedrockModel

model = BedrockModel(
    model_id="us.anthropic.claude-opus-4-6-v1:0",
    max_tokens=16000,
    additional_request_fields={
        "thinking": {
            "type": "adaptive",
            "budget_tokens": 10000  # optional with adaptive
        }
    }
)

agent = Agent(
    model=model,
    system_prompt="You are a federal proposal drafting expert...",
    tools=[...]
)
```

**Fixed-budget interleaved thinking (matching the official AWS blog pattern):**

```python
from strands import Agent
from strands.models import BedrockModel

# Pattern from the official AWS blog post:
# https://aws.amazon.com/blogs/opensource/using-strands-agents-with-claude-4-interleaved-thinking/
model = BedrockModel(
    model_id="us.anthropic.claude-sonnet-4-20250514-v1:0",
    additional_request_fields={
        "anthropic_beta": ["interleaved-thinking-2025-05-14"],
        "thinking": {
            "type": "enabled",
            "budget_tokens": 8000
        },
    },
)

agent = Agent(model=model, tools=[...])
```

Note: Adaptive thinking (`"type": "adaptive"`) automatically enables interleaved thinking without the beta header. The explicit beta header is only needed when using `"type": "enabled"` with a fixed budget and wanting interleaved thinking between tool calls.

**Effort levels for adaptive thinking:**

| Effort | Behavior | Use For |
|---|---|---|
| `max` | Always thinks, no constraints on depth. Opus 4.6 only. | Proposal Drafter, Executive Summary Writer |
| `high` (default) | Always thinks, deep reasoning on complex tasks | Quality Reviewer, Win Theme Weaver, RFP Advisor |
| `medium` | Moderate thinking, may skip for simple queries | Sub-Orchestrators (if upgraded to Opus) |
| `low` | Minimal thinking, skips for simple tasks | Not recommended for this solution |

To set effort level:

```python
additional_request_fields={
    "thinking": {
        "type": "adaptive",
        "effort": "max"  # or "high", "medium", "low"
    }
}
```

**Fixed-budget extended thinking (non-adaptive):**

```python
additional_request_fields={
    "thinking": {
        "type": "enabled",
        "budget_tokens": 8000  # required — max tokens for thinking
    }
}
```

**Key constraints validated against AWS docs:**

- `budget_tokens` must be less than `max_tokens` for standard extended thinking. However, with interleaved thinking (automatically enabled by adaptive mode), the thinking budget can exceed `max_tokens` because the limit becomes the full context window (200K tokens).
- Claude 4 models return summarized thinking (not full thinking output) by default. You are billed for full thinking tokens generated, not the summary tokens returned.
- When using extended thinking with tool use, `tool_choice` must be set to `any`. Strands handles this automatically in its agentic loop.
- Thinking blocks must be preserved and passed back unmodified in multi-turn conversations. Strands' conversation management handles this automatically.
- Do NOT enable extended thinking on Sonnet 4.6 agents used for extraction and coordination tasks — it adds latency without proportional quality improvement for structured, well-defined tasks.

---

## Prompting Principles for Claude 4.6

**Be explicit, not aggressive.** Claude 4.6 is significantly more responsive to system prompts than previous models. Where you previously needed "CRITICAL: You MUST use this tool when...", now use "Use this tool when...". Over-prompting causes overtriggering.

**Use XML tags for structure.** Wrap distinct content types in descriptive tags (`<instructions>`, `<context>`, `<input>`, `<examples>`). This reduces misinterpretation, especially in complex prompts mixing instructions, context, and variable inputs.

**Provide 3-5 diverse examples.** Few-shot prompting remains one of the most reliable ways to steer output format, tone, and structure. Make examples relevant, diverse (cover edge cases), and wrapped in `<example>` tags.

**Give Claude a role.** Even a single sentence in the system prompt focusing Claude's behavior makes a measurable difference.

**Put long documents at the top.** Place long documents and inputs above your query, instructions, and examples. Queries at the end improve response quality by up to 30% in tests.

---

## Agentic Coding with Claude 4.6

**Subagent orchestration:** Claude 4.6 has a strong predilection for spawning subagents. Steer this:
"Use subagents when tasks can run in parallel, require isolated context, or involve independent workstreams. For simple tasks, sequential operations, or single-file edits, work directly rather than delegating."

**Parallel tool calling:** Claude 4.6 excels at parallel tool execution. Boost to ~100% reliability with:
"If you intend to call multiple tools and there are no dependencies between the calls, make all independent calls in parallel."

**Minimize overengineering:** Claude 4.6 tends to create extra files, add unnecessary abstractions, or build in flexibility that wasn't requested. Add explicit guidance to keep solutions minimal and scoped to what was asked.

**Reduce hallucinations in code:** "Never speculate about code you have not opened. If the user references a specific file, read the file before answering. Investigate and read relevant files BEFORE answering questions about the codebase."

**State tracking across context windows:**
- Use git for state tracking across sessions
- Write tests in structured format (e.g., `tests.json`) before starting work
- Create setup scripts (`init.sh`) to gracefully restart from a fresh context window
- Save progress to files before context window refreshes
- Consider starting with a fresh context window rather than compaction; Claude 4.6 is effective at discovering state from the filesystem

---

## Context Awareness

Claude 4.6 can track its remaining context window throughout a conversation. Inform it about your context management strategy:
"Your context window will be automatically compacted as it approaches its limit. Do not stop tasks early due to token budget concerns. Save current progress and state to memory before the context window refreshes."
