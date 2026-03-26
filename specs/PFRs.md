# AWS Product Feature Requests (PFRs)

This file tracks validated AWS Product Feature Request opportunities identified during development work. Each entry has been researched and validated before documentation.

## Summary Table

| ID | Title | AWS Service | Category | Priority | Status |
|----|-------|-------------|----------|----------|--------|
| [PFR-001](#pfr-001-kiro-organization-level-steering-and-hook-distribution) | Kiro Organization-Level Steering and Hook Distribution | Kiro IDE | Missing Feature | High | Open |

---

### PFR-001: Kiro Organization-Level Steering and Hook Distribution

**Date Identified:** 2026-03-26
**AWS Service:** Kiro IDE
**Category:** Missing Feature / Collaboration
**Priority:** High
**Status:** Open

**Describe the Problem or Context.**
Kiro steering documents and agent hooks are stored at the workspace level (`.kiro/steering/` and `.kiro/hooks/`) and are distributed today only through manual processes such as cloning a GitHub template repository, running curl commands, or copying files. There is no native mechanism for an organization, team, or enterprise account to centrally publish, version, and enforce a shared set of steering documents and hooks across all developers and projects. Every developer must manually bootstrap their workspace from a shared repo, and any updates to org-wide standards require each developer to re-pull and re-copy files. There is no concept of an "org-level" or "team-level" steering layer that sits above the workspace level and is automatically applied.

**Describe the target user(s).**
Enterprise customers and large development organizations using Kiro at scale, including platform engineering teams responsible for enforcing coding standards, security policies, and development workflows across hundreds of developers and dozens of projects. AWS Solutions Architects, DevOps leads, and engineering managers who need consistent AI behavior across their entire organization without relying on each developer to manually maintain their local `.kiro` configuration.

**Describe the desired future state.**
Kiro should support an organization-level or team-level configuration layer that sits above the workspace level. Specifically:

1. An org admin or team lead should be able to publish a canonical set of steering documents and hooks to a central location (e.g., an S3 bucket, a Kiro-managed registry, or an AWS Organizations policy attachment).
2. When a developer opens any workspace in Kiro, org-level steering and hooks should be automatically applied without any manual setup, in addition to any workspace-level overrides.
3. Updates to org-level steering should propagate to all developers automatically on next IDE sync or session start, without requiring each developer to re-clone or re-copy files.
4. Workspace-level steering should be able to override or extend org-level steering, following a clear precedence model (org < team < workspace).
5. Org admins should be able to mark certain steering documents as locked (non-overridable) to enforce mandatory security or compliance policies.

**Known Workarounds.**
The current workaround is to maintain a shared GitHub repository (such as the one this project represents) containing the `.kiro` directory, and instruct developers to clone it as a template or copy its contents into their projects. This approach has significant limitations: it requires manual bootstrapping per project, updates are not automatically propagated, there is no enforcement mechanism to ensure developers have applied the latest standards, and there is no way to distinguish between org-mandated policies and optional team preferences. The workaround scales poorly beyond small teams.

**Customer Specific Use Case.**
An AWS Solutions Architect team of 50 engineers needs every member to apply consistent PFR tracking, email drafting standards, and security scanning hooks across all customer engagement workspaces. Today, each SA must manually copy the `.kiro` config into every new workspace they create. When the team lead updates the PFR tracking steering document to add a new required field, all 50 SAs must manually re-pull and re-copy the file. A native org-level distribution mechanism would allow the team lead to publish the update once and have it automatically reflected for all team members on their next Kiro session, with no manual action required per developer.

**Research Validation.**
- Kiro hooks documentation confirms hooks are workspace-local with no org-level distribution: https://kiro.dev/docs/hooks/
- Kiro steering documentation confirms steering files live in `.kiro/steering/` at the workspace level: https://kiro.dev/docs/
- The Kiro GitHub repository issue tracker and feature request mechanism: https://github.com/kirodotdev/Kiro/issues/new?template=feature_request.yml
- No evidence of a centralized org-level steering registry or hook distribution mechanism was found in current Kiro documentation or changelog as of March 2026: https://kiro.dev/changelog/
- Comparable capability exists in GitHub Copilot's organization-level custom instructions (`.github/copilot-instructions.md` at the org level), confirming this is a recognized pattern in the AI IDE space.

---
