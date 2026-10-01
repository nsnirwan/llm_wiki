# Context Index — {{PROJECT_NAME}}

<!-- One line per page. The agent reads this to decide what to open. -->

## Persistent — human knowledge, hand-maintained

| Page | Contents | Read when |
|---|---|---|
| `overview.md` | architecture, setup, key paths, env, services | onboarding or structural change |
| `rules.md` | business invariants and domain rules | touching business logic |
| `constraints.md` | architectural constraints, layering, module ownership | writing or moving any code |
| `glossary.md` | domain terms and disambiguations | domain-heavy work, naming things |
| `features.md` | feature → entry point / module / grep key | "where is X implemented" |

## Session — overwritten

| Page | Contents | Read when |
|---|---|---|
| `state.md` | where we left off, goal, next step, blockers | always (auto-injected) |

## Historical — append only

| Page | Contents | Read when |
|---|---|---|
| `decisions.md` | ADR log with status | making or revisiting a design choice |
| `lessons.md` | AI incidents, reverts, rules learned the hard way | before risky changes |
| `log.md` | chronological session and commit history | needing history |

## Derived — never stored, always generated

| Command | Answers |
|---|---|
| `llmw impact <path>` | what breaks if I change this |
| `llmw feature <name>` | which files currently implement this feature |
| `llmw todos` | TODO / FIXME / HACK / deprecated debt |
| `llmw timeline` | how the architecture evolved |
