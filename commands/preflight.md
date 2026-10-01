---
description: Impact analysis before touching code — run this before any non-trivial change
argument-hint: [what you're about to change]
---

Before changing: $ARGUMENTS

Do this in order, and **stop after step 4** to show me the assessment. Do not start
editing.

1. **Check the constraints.** Read `.context/constraints.md`. Does the intended change
   violate a layering rule, module ownership boundary, or directional rule? If yes,
   say so and stop — propose a compliant alternative instead.

2. **Check the lessons.** Read the "Standing rules" section of `.context/lessons.md`.
   Has this exact class of change broken something before?

3. **Compute blast radius.** Run `llmw impact <path>`. Pay attention to the historical
   coupling section — files that change together are coupled even when nothing greps.
   If precise callers matter, use symbol search (Serena/LSP) as well.

4. **Report, in this shape and nothing more:**

```
Change:      <one line>
Constraints: OK | VIOLATES C<n> — <which>
Prior incident: none | lessons.md <date>
Direct refs: <n> files
Coupled:     <top 3 co-changing files>
Tests:       <which cover it> | ⚠ NONE
Risk:        low | medium | high — <the one reason>
Rollback:    <the specific command or steps>
Plan:        1. 2. 3.
```

If Risk is high or Tests is NONE, say plainly what you'd want in place first.
