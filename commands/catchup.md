---
description: Re-orient at the start of a session or after a break
---

Re-orient me. Use what was already injected at session start — do not re-read files
you already have.

Output exactly this, nothing more:

**Where we are:** (one line from state.md "Right now")
**Next step:** (the first item under "Next step")
**Since last session:** (any commits in git log that aren't reflected in state.md — if
state.md looks stale relative to the commits, say so explicitly)
**Watch out for:** (anything under gotchas or dead ends relevant to the next step)

If `state.md` is more than 7 days older than the newest commit, lead with:
`⚠ state.md may be stale.`
