---
title: LLM Memory Systems & Context Engineering
inclusion: manual
---

# LLM Memory Systems & Context Engineering

Research-validated strategies for agent memory architecture, context window management, and token optimization. Grounded in cognitive science research and production patterns (2025-2026).

---

## Memory Systems

### Memory Architecture Overview

Effective agent memory draws from cognitive science, implementing multiple complementary memory types. Research shows memory-augmented approaches reduce token usage by over 90% while maintaining competitive accuracy.

Source: "Evaluating Long-Term Memory for Long-Context Question Answering" (2025) [arxiv.org/html/2510.23730v2]

### Memory Types

**Semantic Memory (Knowledge)**
- Stores factual knowledge, domain rules, and learned patterns
- Implementation: RAG with vector databases, knowledge graphs, structured fact stores
- Best for: domain-specific knowledge that the base model lacks
- Optimization: memory architecture complexity should scale with model capability; foundation models benefit most from RAG, while stronger instruction-tuned models gain more from episodic learning
- Use grow-and-refine patterns to prevent brevity bias (trajectory-based memory gradually loses essential domain knowledge)

Source: "Agentic Learner with Grow-and-Refine Multimodal Semantic Memory" (2025) [arxiv.org/html/2511.21678]

**Episodic Memory (Experiences)**
- Stores past interaction trajectories, successes, failures, and their contexts
- Implementation: indexed logs of past agent runs with outcome labels, reflection summaries
- Best for: helping agents recognize the limits of their own knowledge and avoid repeating mistakes
- Key insight: episodic memory helps LLMs recognize when they don't know something, improving calibration
- Use MemRL-style separation: keep the stable reasoning of a frozen LLM separate from the plastic, evolving memory

Source: "Self-Evolving Agents via Runtime Reinforcement Learning on Episodic Memory" (2026) [arxiv.org/html/2601.03192v1]

**Procedural Memory (How-To)**
- Stores optimized prompt templates, tool usage patterns, and workflow recipes
- Implementation: prompt optimization pipelines, skill libraries, cached tool chains
- Best for: recurring tasks where the optimal approach has been discovered through iteration

**User Preference Memory**
- Stores individual user patterns, communication style, domain context, and past decisions
- Implementation: structured user profiles with key-value pairs, updated after each interaction
- Best for: personalized agent behavior across sessions

### Memory Optimization Strategies

**Intelligent Decay (Proactive Pruning)**
Assign each memory a composite score factoring in:
- Recency: when was this memory last accessed?
- Relevance: how often does this memory contribute to successful outcomes?
- User-specified utility: has the user explicitly marked this as important?

Prune or consolidate memories that fall below a threshold. This prevents unbounded memory growth while retaining high-value information.

Source: "Memory Management and Contextual Consistency for Long-Running Low-Code Agents" (2026) [arxiv.org/html/2509.25250v1]

**Hierarchical Memory Allocation**
- Hot tier: recent context, current task state (in-context window)
- Warm tier: session-level summaries, active project context (fast retrieval store)
- Cold tier: historical interactions, archived knowledge (vector DB with semantic search)

**Summarization Best Practices**
- Summarize completed conversation segments before they leave the context window
- Use structured summaries (JSON or markdown with consistent fields) rather than free-form prose
- Preserve: key decisions made, tools used, errors encountered, user preferences expressed
- Discard: verbose reasoning chains, failed attempts (unless the failure pattern is novel), redundant confirmations
- Periodically re-summarize warm-tier memories to compress further without losing critical information

**Memory Retrieval Optimization**
- Use hybrid retrieval: combine semantic similarity search with keyword/metadata filtering
- Limit retrieved memories to 3-5 most relevant items per query to avoid context pollution
- Score retrieved memories for relevance before injecting into context; discard below-threshold results
- For multi-turn conversations, maintain a rolling summary rather than retrieving full history

---

## Context Engineering

### The Shift from Prompt Engineering to Context Engineering

Research from Anthropic, JetBrains, and OpenAI confirms that "context stuffing" leads to context bloat, where performance degrades, costs increase, and latency becomes unacceptable. The discipline has evolved from crafting individual prompts to engineering the entire context pipeline.

### Context Window Management Strategies

**1. Selective Context (Highest Signal-to-Noise)**
- Only inject information directly relevant to the current step
- Use relevance scoring to filter candidate context before injection
- Preferred for most agentic workloads where precision matters

**2. Rolling Summarization (Infinite but Lossy)**
- Summarize older context as the window fills, keeping recent context verbatim
- Good for long conversations where general continuity matters more than exact recall
- Risk: critical details can be lost in summarization

**3. Sliding Window (Simple but Amnesiac)**
- Keep only the most recent N tokens; drop everything older
- Suitable for stateless or near-stateless interactions
- Not recommended for multi-step agentic tasks

**4. Entity Extraction (Dense but Complex)**
- Extract and maintain structured entities (people, decisions, code references) from context
- High information density but requires robust extraction pipeline
- Best combined with selective context for complex domains

### Context Optimization Techniques

**Performance degrades with context length.** The NoLiMa benchmark found that at 32k tokens, 11 of 12 tested models dropped below 50% of their short-context performance. Implications:
- Keep active context as lean as possible
- Prefer retrieving specific information over dumping entire documents
- Use structured token retention: dynamically adjust which tokens persist based on contextual significance

Source: "Structured Token Retention and Computational Memory Paths in Large Language Models" (2025) [arxiv.org/html/2502.03102v1]

**Context placement matters.** For Claude models:
- Place reference documents at the top of the prompt
- Place the query/instruction at the bottom
- This ordering alone can improve response quality by up to 30%

**Compress aggressively for multi-turn agents:**
- After each tool call, summarize the result rather than keeping raw output
- Strip boilerplate, headers, and formatting from retrieved documents
- Use structured formats (JSON, markdown tables) over prose for data-heavy context
- Remove successfully completed task descriptions from context; keep only the remaining todo list

### Context Budget Allocation

For a typical agentic turn with a 200k token window:
- System prompt + instructions: 5-10% (10k-20k tokens)
- Retrieved context / documents: 20-30% (40k-60k tokens)
- Conversation history (summarized): 10-15% (20k-30k tokens)
- Current task context + tool results: 20-30% (40k-60k tokens)
- Reserved for model output + thinking: 25-35% (50k-70k tokens)

Never fill more than 75% of the context window. Leave headroom for the model's output and internal reasoning.

---

## References

1. "Evaluating Long-Term Memory for Long-Context Question Answering" (2025). [arxiv.org/html/2510.23730v2]
2. "Agentic Learner with Grow-and-Refine Multimodal Semantic Memory" (2025). [arxiv.org/html/2511.21678]
3. "Self-Evolving Agents via Runtime Reinforcement Learning on Episodic Memory" (2026). [arxiv.org/html/2601.03192v1]
4. "Memory Management and Contextual Consistency for Long-Running Low-Code Agents" (2026). [arxiv.org/html/2509.25250v1]
5. "Efficient Lifelong Memory for LLM Agents" (2026). [arxiv.org/html/2601.02553]
6. "Structured Token Retention and Computational Memory Paths in Large Language Models" (2025). [arxiv.org/html/2502.03102v1]
