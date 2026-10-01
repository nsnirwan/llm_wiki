# Decision Log — {{PROJECT_NAME}}

<!--
  APPEND ONLY. Newest at the bottom. Never edit or delete a past entry.
  To reverse a decision, add a NEW one and change the old entry's Status line only.

  STATUS is the one field you may edit retroactively:
    Accepted     — in force
    Experimental — trying it, may not stick; don't build hard on it
    Deprecated   — still true in code, but don't extend it
    Superseded   — replaced; says by which ADR
    Reverted     — we tried it and undid it. KEEP IT. This is the most valuable kind.

  TYPE drives the architecture timeline (`llmw timeline`):
    architecture · stack · data · security · process · ops
-->

Format: `## [YYYY-MM-DD] ADR-00N — Title`

---

## [{{DATE}}] ADR-001 — Adopt .context/ for project memory

- **Status:** Accepted
- **Type:** process
- **Context:** Sessions lose context; the same decisions get re-argued; re-explaining
  the project every session is expensive.
- **Options:** ad-hoc CLAUDE.md · memory MCP server · full spec-driven framework ·
  file-based `.context/` in-repo
- **Decision:** file-based `.context/` committed with the code, aggregated by a hub repo.
- **Why:** versions atomically with the code, inspectable, recoverable from git history,
  zero infrastructure.
- **Consequences:** requires discipline at session end (mitigated by hooks); does not
  scale past ~1000 pages (not a concern at current size).
