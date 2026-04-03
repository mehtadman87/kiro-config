---
title: Research-Validated Development
inclusion: auto
name: research-validated-development
description: When creating specs, making architecture decisions, selecting technologies, or designing solutions that need validation against current best practices.
---

# Research-Validated Development

## Core Principle
Every significant decision during spec creation, architecture design, and implementation must be validated through web search research. This ensures solutions are grounded in current best practices, official documentation, and proven patterns rather than assumptions or outdated knowledge.

## When to Research (Mandatory)
- Creating or modifying requirements documents (requirements.md)
- Creating or modifying design documents (design.md)
- Creating or modifying task lists (tasks.md)
- Selecting AWS services or architectural patterns
- Choosing libraries, frameworks, or dependencies
- Troubleshooting errors, build failures, or runtime issues
- Making trade-off decisions between competing approaches
- Designing API contracts, data models, or integration patterns

## Research Validation Process
1. Before finalizing any spec document section, search for current best practices
2. Cross-reference architectural decisions against official AWS documentation
3. Validate service selections against the AWS Well-Architected Framework pillars:
   - Operational Excellence
   - Security
   - Reliability
   - Performance Efficiency
   - Cost Optimization
   - Sustainability
4. Verify library/dependency choices are actively maintained and well-adopted
5. Check for known issues, limitations, or anti-patterns with chosen approaches

## Architecture-First Approach
- Always design and validate the backend AWS architecture before frontend work
- Start with infrastructure and service design, then move to API contracts, then frontend
- Sequence: AWS Services → Infrastructure (CDK/CloudFormation) → API Layer → Frontend
- Validate each layer against AWS documentation before proceeding to the next

## Error and Issue Resolution
- When encountering errors, always search for the specific error message or code
- Check official documentation and known issues before attempting fixes
- Look for community-validated solutions on Stack Overflow, GitHub issues, and AWS forums
- Prefer official AWS troubleshooting guides over blog posts

## What to Document from Research
- Source URLs for key architectural decisions
- AWS service quotas or limitations that affect the design
- Known gotchas or caveats discovered during research
- Alternative approaches considered and why they were rejected

## No Shortcuts Policy
- Never use placeholder, mock, or hardcoded data in any layer of the solution
- Always implement real integrations, even in early iterations
- If a real integration isn't feasible yet, document it as technical debt with a clear remediation plan
- Prefer working end-to-end slices over partially mocked features
