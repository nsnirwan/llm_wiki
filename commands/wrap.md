---
description: Save session state before stopping — updates state.md and log.md
---

Wrap up this session. Do this without asking me questions:

1. **Rewrite `.context/state.md`** (overwrite, don't append). It must answer
   "if I sat down tomorrow with no memory, what do I need to know?":
   - `Right now` — what is actively in progress, 1–2 lines
   - `Next step` — the very next concrete action, numbered
   - `Blocked / waiting on`
   - `Known bugs & gotchas` — add anything discovered this session, remove anything fixed
   - `Dead ends — do not retry` — approaches tried that failed, and why
   Update the `**Updated:**` date and branch. Keep the whole file under 40 lines.

2. **Append one entry to `.context/log.md`** in the form:
   `## [YYYY-MM-DD] session | <3-6 word summary>`
   followed by 2–5 bullets: what changed, what was decided, what's open.
   Never edit past entries.

3. **If a real decision was made**, append an ADR to `.context/decisions.md`
   (context / options / decision / why / consequences — five lines is plenty).

4. Report back in **two lines maximum**: what you wrote to state.md's "Next step",
   and whether an ADR was added. Nothing else.
