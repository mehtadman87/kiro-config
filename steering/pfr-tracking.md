---
title: AWS Product Feature Request (PFR) Tracking
inclusion: auto
name: pfr-tracking
description: When identifying AWS service gaps, limitations, workarounds, or PFR opportunities. Use when reviewing AWS architectures, troubleshooting service issues, or documenting feature requests.
---

# AWS Product Feature Request (PFR) Tracking

## Role Context

You are assisting an AWS Solutions Architect whose yearly goals include opening Product Feature Requests (PFRs) with AWS Product Teams. Leadership expects a high volume of PFRs. Part of the SA role is to look around corners, identify gaps in AWS service offerings, and advocate for improvements that would benefit customers.

## When to Identify PFR Opportunities

Actively watch for PFR opportunities whenever you are:

- Helping design or architect an AWS solution
- Troubleshooting an AWS service limitation or workaround
- Explaining a missing capability or unsupported configuration
- Identifying a gap between what a customer needs and what AWS natively provides
- Noticing friction in deploying, managing, observing, or optimizing AWS services
- Discovering a feature that exists in a competing service but not in AWS
- Finding that a workaround is required where a native capability should exist
- Encountering console UX limitations, API gaps, or missing automation support
- Reviewing AWS documentation that reveals undocumented constraints or limitations

## Research Requirement (Mandatory Before Documenting)

Before documenting any PFR, you MUST conduct thorough research using all available tools:

1. Use web search to verify the gap is real and not already addressed in a recent release
2. Check the AWS documentation (via MCP aws-docs server if available) for the latest feature state
3. Search for any existing workarounds or partial solutions
4. Validate the customer impact and use case breadth
5. Identify any related AWS blog posts, re:Invent sessions, or roadmap hints
6. Check if the feature exists in other AWS regions but not the relevant one

Only document a PFR after research confirms the gap is genuine and current.

## PFR Documentation Format

Document all validated PFR opportunities in a file called `PFRs.md` located at `.kiro/specs/PFRs.md`, alongside the technical debt tracker. Use the following format for each entry, which matches what AWS Product Teams require:

---

### PFR-[NNN]: [Feature Request Title]

**Date Identified:** [YYYY-MM-DD]
**AWS Service:** [Service name, e.g., Amazon Security Hub, AWS Lambda]
**Category:** [Console UX / API Gap / Missing Feature / Integration / Observability / Automation / Performance / Cost Optimization / Security / Compliance]
**Priority:** High / Medium / Low
**Status:** Open / Submitted / Closed

**Describe the Problem or Context.**
[Clear description of the current limitation, gap, or friction point. Be specific about what is missing and why it matters. Include any error messages, API limitations, or console constraints observed.]

**Describe the target user(s).**
[Who is affected: e.g., Security Engineers, DevOps teams, Solutions Architects, Enterprise customers with multi-account environments, regulated industry customers, etc.]

**Describe the desired future state.**
[What the feature or capability should look like when implemented. Be specific about the expected behavior, API shape, console experience, or configuration option.]

**Known Workarounds.**
[Document any existing workarounds, their limitations, and why they are insufficient. If no workaround exists, state that explicitly.]

**Customer Specific Use Case.**
[Concrete use case that illustrates the real-world impact. Reference the customer scenario or architecture pattern that surfaced this gap.]

**Research Validation.**
[Links to AWS documentation, blog posts, or other sources reviewed to confirm this gap is genuine and current. Include any evidence that this is on the roadmap or has been requested before.]

---

## PFRs.md File Management

- Create `.kiro/specs/PFRs.md` if it does not exist
- Append new PFR entries sequentially (PFR-001, PFR-002, etc.)
- Never delete or overwrite existing PFR entries
- Update the Status field when a PFR is submitted or resolved
- Include a summary table at the top of the file listing all PFRs with their ID, title, service, and status

## Proactive Identification

Do not wait to be asked. When you encounter a gap or limitation during any task, proactively:

1. Note it internally
2. Conduct the required research
3. Append the validated PFR to `PFRs.md`
4. Briefly mention to the user: "I identified a PFR opportunity and documented it in PFRs.md — [one sentence summary]."

This keeps PFR tracking continuous and ensures no opportunity is missed.
