# {{PROJECT_NAME}}

<!--
  KEEP UNDER ~70 LINES. Prepended to every request — you pay for it on every turn.
  Everything else loads on demand. Never paste documentation here.
-->

## Universal standards

@.context/standards/00-core.md
@.context/standards/10-code.md
@.context/standards/20-safety.md

## This project

{{ONE_LINE_DESCRIPTION}}

- **Stack:** {{STACK}}
- **Run:** `{{RUN_CMD}}`
- **Test:** `{{TEST_CMD}}`
- **Entry point:** {{ENTRY_POINT}}

## Context — load only what the task needs

**Persistent** (human knowledge; hand-maintained)

| File | Read it when |
|---|---|
| `.context/constraints.md` | writing or moving any code — layering, ownership, forbidden patterns |
| `.context/rules.md` | the task touches business logic or validation |
| `.context/glossary.md` | domain terms are involved, or you're naming something |
| `.context/overview.md` | you need architecture or setup |
| `.context/features.md` | "where is feature X implemented" |

**Historical** (append-only)

| File | Read it when |
|---|---|
| `.context/lessons.md` | before any risky change — full incident history (the standing rules from it are already injected) |
| `.context/decisions.md` | making or revisiting a design choice |
| `.context/log.md` | you need history |

**Derived** — never read a document for these; run the command:

| Question | Command |
|---|---|
| what breaks if I change this? | `llmw impact <path>` |
| which files implement feature X? | `llmw feature <name>` |
| what's the debt here? | `llmw todos` |
| how did the architecture evolve? | `llmw timeline` |

`.context/state.md` and the standing rules from `.context/lessons.md` are auto-injected
at session start — don't re-read them.
Do not open all of these at the start. Open the one the task needs.

## Workflow

- Non-trivial change → `/preflight` first, `/verify` before committing.
- AI broke something → `/incident` immediately, while you remember why.
- Real decision made → `/adr`.
- Stopping → `/wrap`.

## Project-specific overrides

<!-- Wins over the universal standards. Keep short. -->
- (none yet)
