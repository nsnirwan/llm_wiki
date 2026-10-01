---
description: Record an architecture decision
argument-hint: [what was decided]
---

Append an ADR to `.context/decisions.md` for: $ARGUMENTS

Rules:
- Append only. Never edit or delete a previous entry. If this reverses an earlier
  decision, say `Supersedes ADR-00N` and explain what changed.
- Read the existing file first to get the next ADR number right.
- Format, and keep it this short:

```
## [YYYY-MM-DD] ADR-00N — <title>

- **Status:** Accepted | Experimental
- **Type:** architecture | stack | data | security | process | ops
- **Context:** why this came up
- **Options:** A · B · C
- **Decision:** what we chose
- **Why:** the deciding factor, not a list of every pro
- **Consequences:** what this costs us / what it locks in
```

**Status** matters later — `llmw timeline` reads `Type`, and future sessions read
`Status` to know whether a past decision is still binding. Use `Experimental` honestly
when we're trying something; that tells future-you not to build hard on it.

If this reverses an earlier decision, edit **only the old entry's Status line** to
`Superseded by ADR-00N` (or `Reverted`) — never delete it. Reverted decisions are the
most valuable entries in the file, because they stop us re-trying the same thing.

If I haven't given you enough to fill `Why` honestly, ask me exactly one question.
Confirm in one line when done.
