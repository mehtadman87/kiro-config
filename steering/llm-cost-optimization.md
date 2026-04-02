---
title: LLM Cost Optimization
inclusion: manual
---

# LLM Cost Optimization

Research-validated strategies for reducing LLM inference costs 40-80% while maintaining or improving output quality. Covers model routing, caching, token optimization, and batch processing.

---

## Multi-Model Routing (40-60% Savings)

Route queries to the cheapest model capable of handling them. Simple tasks go to smaller/cheaper models; complex tasks go to expensive frontier models.

**Routing strategies:**
- **Classifier-based routing:** Train a lightweight classifier to predict query complexity and route accordingly. Research shows this can outperform the best individual model by 2.6+ percentage points while being 4x cheaper.
- **Cascade routing:** Start with the cheapest model; if confidence is below threshold, escalate to the next tier. This consistently outperforms both pure routing and pure cascading.
- **Hybrid cascade-routing:** Combine both approaches for optimal quality-cost tradeoff.

Source: "Dynamic Model Routing and Cascading for Efficient LLM Inference" (2026) [arxiv.org/html/2603.04445v1]; "A Unified Approach to Routing and Cascading for LLMs" (2024) [arxiv.org/html/2410.10347v1]

**Practical model tiers for Claude:**

| Tier | Model | Use When |
|---|---|---|
| Tier 1 (cheapest) | Haiku 4.5 | Classification, extraction, simple Q&A, formatting |
| Tier 2 (balanced) | Sonnet 4.6 (low effort) | Standard coding, content generation, tool-heavy workflows |
| Tier 3 (capable) | Sonnet 4.6 (high effort) | Complex coding, multi-step reasoning, computer use |
| Tier 4 (frontier) | Opus 4.6 | Long-horizon autonomous work, deep research, large migrations |

---

## Prompt Caching (45-90% Savings)

**Provider-level prompt caching:** Anthropic prefix caching delivers up to 90% cost reduction and 85% latency reduction for long prompts. Structure prompts so that the static portion (system prompt, instructions, reference documents) comes first, and the variable portion (user query) comes last. This maximizes cache hit rates.

Source: "An Evaluation of Prompt Caching for Long-Horizon Agentic Tasks" (2026) [arxiv.org/html/2601.06007v1]

**Semantic caching:** Cache LLM responses and return cached results for semantically similar queries. 31% of LLM queries exhibit semantic similarity, representing massive inefficiency without caching.
- Exact-match caching handles ~30% of queries
- Semantic caching (embedding similarity) handles ~50%
- Combined: 40-60% cost reduction on typical production workloads
- Use similarity thresholds of 0.92-0.95 for semantic cache hits to avoid returning stale or incorrect results

---

## Token Optimization (30-50% Savings)

- Write concise prompts; shorter prompts reduce input tokens and often produce shorter, more focused outputs
- Strip unnecessary formatting, boilerplate, and whitespace from context
- Use structured formats (JSON) over verbose prose for data
- Compress retrieved documents before injection: remove headers, footers, navigation, and irrelevant sections
- For multi-turn conversations, summarize history rather than replaying full transcripts
- Set appropriate `max_tokens` limits; don't default to maximum

---

## Batch Processing (50% Savings)

- Use batch/async APIs for non-real-time workloads (Anthropic offers 50% discount for batch)
- Queue requests and process in bulk during off-peak hours
- Ideal for: report generation, data analysis, content creation, bulk classification

---

## Additional Cost Levers

- **Effort parameter:** For Claude 4.6, use `low` effort for simple tasks; this dramatically reduces thinking tokens
- **Adaptive thinking:** Let the model decide when to think deeply vs respond directly; avoids wasting tokens on easy queries
- **Output length control:** Explicitly request concise responses when verbosity isn't needed
- **Deduplication:** Before sending a query, check if an equivalent query was recently processed
- **Token budgets per agent:** In multi-agent systems, set per-agent token budgets to prevent runaway costs

---

## Cost Monitoring

Track these metrics per agent, per task type, and per user:
- Cost per successful task completion (not just cost per API call)
- Token efficiency ratio: useful output tokens / total tokens consumed
- Cache hit rate (target: above 30%)
- Routing accuracy: % of queries correctly routed to the optimal cost tier
- Wasted tokens: tokens spent on failed attempts, retries, and hallucinated outputs

---

## References

1. "Dynamic Model Routing and Cascading for Efficient LLM Inference" (2026). [arxiv.org/html/2603.04445v1]
2. "A Unified Approach to Routing and Cascading for LLMs" (2024). [arxiv.org/html/2410.10347v1]
3. "An Evaluation of Prompt Caching for Long-Horizon Agentic Tasks" (2026). [arxiv.org/html/2601.06007v1]
4. "Characterization and Analysis of LLM Routing and Hierarchical Techniques" (2025). [arxiv.org/html/2506.06579v1]
