---
description: Record an AI mistake or a revert — turns one bad session into a permanent rule
argument-hint: [what went wrong]
---

Record this incident in `.context/lessons.md`: $ARGUMENTS

1. **Append to `## Incidents`** (never edit past entries):

```
### [YYYY-MM-DD] <tool> — <what broke>

- **What happened:**
- **Why it happened:** what the AI didn't know, or what wasn't written down
- **Blast radius:** files / modules / did it reach main?
- **Recovery:** the exact command used
- **Rule extracted:** the one-line imperative
- **Prevention:** which context file should have said this
```

2. **Add the extracted rule to `## Standing rules`** at the top of the file. One line,
   imperative, no explanation. If Standing rules is now over ~15 lines, tell me which
   old ones have become obvious and can go — that section is loaded every session, so
   it must stay short.

3. **Fix the root cause.** "Why it happened" almost always points at a missing entry in
   `rules.md`, `constraints.md`, or `glossary.md`. Add it there too. An incident that
   only produces a lessons entry will happen again.

Confirm in two lines: the rule added, and which context file you fixed.
