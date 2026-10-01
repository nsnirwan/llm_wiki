---
description: Bootstrap .context/ for a project that doesn't have one yet
---

**STOP — precondition check. Do this first, before anything else.**

Run `llmw doctor` and read the "Current project" section. If ANY of these fail —
`.context/ present`, `standards symlink`, `all 10 context files`,
`CLAUDE.md imports standards` — then stop immediately and tell me:

> `llmw init` hasn't been run (or didn't complete) in this project. Run `llmw init`
> first, then re-run `/onboard`.

Do **not** create `.context/` yourself, do not create any file inside it, and do not
work around a missing setup. If you scaffold it manually the project silently loses its
standards symlink, git hook, and hub registration — everything looks fine and isn't.
This has happened before; refusing is the correct behaviour.

Only continue past this point if `llmw doctor` is clean for the current project.

---

This project has `.context/` scaffolding but the files are still templates.
Fill them in from the actual codebase.

**Delegate the exploration to a subagent** so the exploration transcript doesn't
fill this context window. Ask it to report back only the findings, not the files it read.

Then write:

1. `.context/overview.md` — what this is, the 5–10 architectural components and how
   they talk, key paths table, setup commands, env vars, external services.
   **No file trees, no module inventories, no API signature dumps** — those are derived
   live, not stored.

2. `.context/rules.md` — business invariants and domain rules you can infer from
   validation code, guards, and constraints. Mark anything you're inferring rather
   than reading directly with `(inferred)`.

3. `.context/state.md` — from recent git log: what was last being worked on, and the
   obvious next step. Say "unknown" rather than guessing.

4. `CLAUDE.md` — fill in the stack/run/test/entry-point lines. Keep it under 70 lines.
   **Never remove the `@.context/standards/*` import lines at the top** — those are
   what load the universal rules.

5. `.context/glossary.md` and `.context/constraints.md` — populate from what you saw.
   Constraints especially: layering rules, module ownership, anything you'd reject a PR
   for. These are the two files agents violate most and they're usually left empty.

Then show me a diff summary and flag anything you were unsure about.
