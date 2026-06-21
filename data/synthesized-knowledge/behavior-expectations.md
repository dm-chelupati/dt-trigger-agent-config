---
name: behavior-expectations
description: User-provided behavior preferences for how the agent should investigate incidents and handle code
---

## Incident Investigation Preferences

### Always pull latest code before investigating
- **What:** Run `git pull origin main` on all relevant repo clones before starting any incident investigation.
- **Why:** A stale local clone can miss recent commits that are the root cause. In INC0010012, the gateway `/v2/orders` bug was on `main` but the local clone was behind, causing the agent to miss the source-code-level root cause initially.
- **Source:** User feedback, 2026-06-17.

### Always investigate source code for root cause and suggest fixes
- **What:** Every incident investigation must include a source-code-level RCA. Don't stop at infrastructure-level findings (e.g., "a broken image was deployed"). Trace the issue back to the code, identify the exact line/commit, and propose or implement a fix via PR.
- **Why:** Infrastructure rollbacks are mitigations, not permanent fixes. The source code is the authoritative truth — if the bug is in `main`, the next deploy will reintroduce it.
- **Source:** User feedback, 2026-06-17.
