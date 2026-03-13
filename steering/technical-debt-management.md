---
title: Technical Debt Management
inclusion: always
---

# Technical Debt Management

## Core Principle
Technical debt must be visible, categorized, and tracked. Any fix, improvement, or refactoring that falls outside the scope of the current task must be logged in the project's technical debt register rather than ignored or forgotten.

## When to Log Technical Debt
- A shortcut or workaround is taken to meet current task scope
- A better approach is identified but implementing it would exceed current scope
- A dependency needs upgrading but isn't blocking current work
- Code duplication is introduced that should be refactored later
- Error handling is simplified and needs hardening
- Performance optimizations are deferred
- Security improvements are identified but not immediately critical
- Test coverage gaps are noted
- Documentation is incomplete or outdated
- Infrastructure improvements are identified (cost, scaling, monitoring)

## Technical Debt Log Location
- Each spec/project should maintain a `technical-debt.md` file in its `.kiro/specs/{feature-name}/` directory
- For cross-cutting concerns, use `.kiro/specs/technical-debt.md` as a global register

## Debt Entry Format
Each entry must include:
- **ID**: Sequential identifier (TD-001, TD-002, etc.)
- **Date Identified**: When the debt was discovered
- **Category**: One of [Code Quality, Architecture, Security, Performance, Testing, Documentation, Infrastructure, Dependencies]
- **Severity**: Critical / High / Medium / Low
- **Description**: Clear description of the debt and why it exists
- **Impact**: What happens if this debt is not addressed
- **Remediation**: Suggested approach to resolve it
- **Related Task**: The spec task that introduced or discovered this debt
- **Status**: Open / In Progress / Resolved

## Severity Guidelines
- **Critical**: Security vulnerabilities, data integrity risks, or production stability threats
- **High**: Significant performance issues, missing error handling, or architectural concerns that will compound
- **Medium**: Code quality issues, missing tests, or suboptimal patterns that slow development
- **Low**: Nice-to-have improvements, minor refactoring, or documentation gaps

## Review Cadence
- Review the technical debt log at the start of each new spec or feature
- Prioritize critical and high severity items before starting new features
- Consider bundling related debt items into dedicated cleanup tasks
