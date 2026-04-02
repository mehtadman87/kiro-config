---
title: Vidya Spark Generator
inclusion: manual
---

# Vidya Spark Generator

When a user asks to generate a Vidya Spark input, solution, or idea, follow this process:

## Step 1: Analyze the Solution Context

Thoroughly review all planning files in the workspace:
- All steering files in `.kiro/steering/`
- All spec files in `.kiro/specs/`
- Any design documents, requirements, or architecture files
- README.md and any other project documentation
- Any code files that reveal the solution's implementation

Synthesize a comprehensive understanding of what the solution does, who it serves, and what problems it solves.

## Step 2: Conduct Deep Research

Use Kiro's web search tools, Kiro Powers, and available MCP servers to:

1. Take a step back from the solution and identify the core problems being addressed
2. Research the customer's industry, market position, and competitive landscape
3. Investigate the problem domain at a PhD level:
   - Academic research (arxiv, IEEE, ACM) on the problem space
   - Industry analyst reports (Gartner, Forrester, IDC, McKinsey) on market trends
   - Current state-of-the-art solutions and their limitations
   - Quantitative data on the problem's business impact (cost, time, failure rates)
4. Analyze competing solutions and identify differentiation opportunities
5. Research best practices and proven patterns for the solution domain
6. Identify metrics and KPIs that matter most in this problem space

## Step 3: Enhance the Solution Value

Using the research findings:
- Identify gaps in the current solution description that could be strengthened
- Find quantitative evidence that supports the solution's value proposition
- Discover additional customer pain points the solution could address
- Identify industry-specific language and framing that resonates with the target audience
- Ensure the solution description connects technical capabilities to business outcomes

## Step 4: Generate the Vidya Spark Output

Save the output as `vidya-spark-generator.md` in the workspace root (or a location specified by the user). Use the following format:

```markdown
# Vidya Spark: [Solution Name]

## Customer Business Overview / Specific Background Context
[Enter context about the customer's business, industry position, and relevant background. Include industry size, growth trends, and competitive dynamics. Ground this in research findings.]

## Problem Statement
[Describe the problem being solved. Include quantitative evidence of the problem's impact — cost, time, failure rates, industry statistics. Connect the problem to business outcomes the customer cares about.]

## Idea Name
[Solution idea title — concise, descriptive, and compelling.]

## Solution Description
[Describe the solution envisioned. Connect technical capabilities to business value. Explain how the solution addresses the root causes identified in research, not just symptoms. Frame the solution in terms of outcomes, not just features.]

## Key Performance Indicators (KPIs) for Customer

| KPI Name | Target |
|---|---|
| [KPI 1] | [Target 1] |
| [KPI 2] | [Target 2] |
| [KPI 3] | [Target 3] |
| [KPI 4] | [Target 4] |
| [KPI 5] | [Target 5] |

## Key Features in the Product (max 5)

| Feature Name | Feature Description |
|---|---|
| [Feature 1] | [1-2 sentence description with expected outcomes] |
| [Feature 2] | [1-2 sentence description with expected outcomes] |
| [Feature 3] | [1-2 sentence description with expected outcomes] |
| [Feature 4] | [1-2 sentence description with expected outcomes] |
| [Feature 5] | [1-2 sentence description with expected outcomes] |
```

## Important Notes

- This is for a Proof-of-Concept (PoC) that will be demoed to the customer. Frame everything accordingly — focus on demonstrable value, not theoretical capabilities.
- Every claim should be grounded in research. Cite sources where possible.
- The goal is to understand the customer, the industry, the problem they are trying to solve, and how best to create a product that not only solves their problem but provides real value.
- Prioritize business impact language over technical jargon in the customer-facing sections.
- KPIs should be measurable, time-bound where possible, and directly tied to the customer's business objectives.
- Features should be scoped to what can realistically be demonstrated in a PoC, not a full production system.
