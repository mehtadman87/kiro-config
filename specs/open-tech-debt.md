# Open Technical Debt

Active, unresolved technical debt items. Items are moved to `closed-tech-debt.md` when resolved.

IDs are permanent and never re-used or re-numbered.

---

### TD-001

- **ID**: TD-001
- **Date Identified**: 2026-04-03
- **Category**: Documentation
- **Severity**: High
- **Description**: The GitHub repo (`mehtadman87/kiro-config`) stores `hooks/`, `steering/`, `settings/`, and `specs/` at the repository root, but the README's Option 1 (clone as template) and Option 2/3 (copy to existing project) instructions were written assuming a `.kiro/` subfolder wrapper exists in the repo. This mismatch caused install failures for users following the README. The instructions have been corrected to copy each subdirectory individually, but the underlying repo structure is still inconsistent with the `.kiro/` convention Kiro expects.
- **Impact**: Users following the README instructions get `No such file or directory` errors. The repo cannot be used as a drop-in template without manual path correction. New contributors adding files may place them in the wrong location.
- **Remediation**: Either (a) reorganize the GitHub repo to wrap all files under a `.kiro/` subdirectory so the structure mirrors what gets installed, or (b) keep the flat structure and ensure all README options consistently reference the flat paths. Option (a) is preferred as it makes the repo self-documenting and allows a single `cp -r kiro-config-main/.kiro/. .kiro/` command.
- **Related Task**: README install instruction fix (session 2026-04-03)

