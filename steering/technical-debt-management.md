---
title: Technical Debt Management
inclusion: always
---

# Technical Debt Management

## Core Principle
Technical debt must be visible, categorized, and tracked. Any fix, improvement, or refactoring that falls outside the scope of the current task must be logged in the project's open technical debt register rather than ignored or forgotten.

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

## Technical Debt File Locations

Two files track technical debt:

- **`.kiro/specs/open-tech-debt.md`** — All open (unresolved) technical debt items. This is the active working file.
- **`.kiro/specs/closed-tech-debt.md`** — All resolved/remediated technical debt items. Serves as an audit trail.

For feature-specific debt, use `.kiro/specs/{feature-name}/open-tech-debt.md` and `.kiro/specs/{feature-name}/closed-tech-debt.md`.

## Logging New Debt

Add new items to `open-tech-debt.md` using the entry format below. Assign the next sequential ID (TD-001, TD-002, etc.). IDs are permanent and must never be re-used or re-numbered, even after items are moved to the closed file.

## Resolving Debt

When technical debt is remediated or resolved:

1. Remove the entry from `open-tech-debt.md`
2. Add the entry to `closed-tech-debt.md` with two additional fields:
   - **Date Resolved**: YYYY-MM-DD
   - **Resolution**: Brief description of how it was resolved (commit, PR, refactor, etc.)
3. Do NOT re-number remaining items in `open-tech-debt.md`. IDs are permanent.

## Debt Entry Format (Open)
Each entry in `open-tech-debt.md` must include:
- **ID**: Sequential identifier (TD-001, TD-002, etc.) — permanent, never re-used
- **Date Identified**: YYYY-MM-DD
- **Category**: One of [Code Quality, Architecture, Security, Performance, Testing, Documentation, Infrastructure, Dependencies]
- **Severity**: Critical / High / Medium / Low
- **Description**: Clear description of the debt and why it exists
- **Impact**: What happens if this debt is not addressed
- **Remediation**: Suggested approach to resolve it
- **Related Task**: The spec task that introduced or discovered this debt

## Debt Entry Format (Closed)
Each entry in `closed-tech-debt.md` includes all open fields plus:
- **Date Resolved**: YYYY-MM-DD
- **Resolution**: How it was resolved

## Severity Guidelines
- **Critical**: Security vulnerabilities, data integrity risks, or production stability threats
- **High**: Significant performance issues, missing error handling, or architectural concerns that will compound
- **Medium**: Code quality issues, missing tests, or suboptimal patterns that slow development
- **Low**: Nice-to-have improvements, minor refactoring, or documentation gaps

## Review Cadence
- Review `open-tech-debt.md` at the start of each new spec or feature
- Prioritize critical and high severity items before starting new features
- Consider bundling related debt items into dedicated cleanup tasks
