---
description: Post-change verification — run before committing
---

Verify the change you just made. Do not summarise what you did; check it.

1. `git diff --stat` — what actually changed vs. what was supposed to change. Flag any
   file you didn't intend to touch.
2. Run the tests. If you cannot run them, say so plainly — never imply they passed.
3. Re-read `.context/constraints.md` and confirm the diff doesn't violate one.
4. Scan the diff for secrets, keys, tokens, connection strings.
5. Check whether the change invalidates anything in `rules.md`, `features.md`, or
   `overview.md`. Update only what's actually now wrong.

Report exactly this:

```
Files changed:   <n>  (<any unintended ones>)
Modules touched: <list>
Constraints:     OK | VIOLATES <n>
Tests:           passed | failed <which> | NOT RUN — <why>
Secrets scan:    clean | FOUND <what>
Docs needing update: none | <file — what's now wrong>
Risk left:       <one line>
```

If anything failed, lead with that. Do not bury it.
