# Command Cheatsheet

`$` = terminal · `>` = typed inside Claude Code

---

## 0. One-time setup (once per machine, not per project)

```bash
~/projects/llm_wiki/bin/llmw-install
```

Symlinks the CLI, slash commands and subagent; merges the hooks into
`~/.claude/settings.json` without touching your existing settings (backs it up first);
then runs `llmw doctor`. Global checks should be all `ok`; per-project failures are
expected until you're inside a project.

**On a new machine:** clone the hub, run the installer, then `llmw adopt` inside each
project after cloning it. Full walkthrough: [`docs/new-machine.md`](docs/new-machine.md).

**Multiple Claude accounts on one machine** (work vs personal, separate billing):

```bash
llmw-profile add work                              # creates config dir, installs
llmw-profile add personal                          # llmw layer, runs /login
llmw-profile shell-init >> ~/.zshrc && source ~/.zshrc
```

Then from inside any project — same repo, same `.context/`, different account:

```bash
cw work
cw personal
cw                        # no arg = list profiles + login status
```

`llmw-profile doctor <name>` verifies the profile has the slash commands and hooks.
Project context is untouched by profile switching — it lives in the repo, not the account.

---

## 1. NEW project

```bash
$ mkdir ~/work/newthing && cd ~/work/newthing
$ git init && git commit --allow-empty -m "init"

$ llmw init                    # .context/, CLAUDE.md, standards symlink, git hook, hub page
$ llmw doctor                  # ← GATE. All green before opening claude.
$ claude
```

```
> I'm building <what it does, who uses it, the stack>.
> Fill in .context/overview.md and .context/glossary.md from that.
> Ask me anything you need.

> /adr chose <stack> because <reason>

> /wrap
```

```bash
$ llmw lint                    # catches leftover {{placeholders}}
$ git add -A && git commit -m "chore: project context"
$ llmw sync                    # register in hub index
```

Then go to **section 3**.

---

## 2. EXISTING project

```bash
$ cd ~/work/old-erp
$ llmw init                    # keeps your CLAUDE.md, prepends the standards imports
$ llmw doctor                  # ← GATE. Do not open claude until this is clean.
$ claude
```

**`llmw doctor` is not optional here.** If you skip it and `init` didn't complete,
`/onboard` used to scaffold `.context/` itself — producing something that looks right
but has no standards symlink, no git hook, and no hub page. `/onboard` now refuses in
that situation, but checking first is still faster than finding out later.

`llmw init` is safe to re-run any time. It backfills missing context files, never
overwrites existing ones, and repairs the symlink, hook, and hub page.

```
> /onboard
```

`/onboard` sends a subagent to read the codebase and fills `overview.md`, `rules.md`,
`state.md`. Anything it guessed is marked `(inferred)`.

**Then review — this step is not optional.** Open `.context/rules.md` and
`.context/constraints.md`. Delete what's wrong, fix what's close. A wrong `rules.md` is
worse than an empty one, because every future session will trust it.

```
> Add to constraints.md: controllers never call the DB directly, always via services.
> Add to glossary.md: "Order" means a business purchase order, NOT the `order` table.

> /wrap
```

```bash
$ llmw lint
$ git add -A && git commit -m "chore: add project context"
$ llmw sync
```

**If the project already had a `CLAUDE.md`:** `llmw init` keeps it and prepends the
three standards imports automatically (saving `CLAUDE.md.bak` first). Nothing to do by
hand. Just move anything long out into `.context/overview.md` so the file stays under
~70 lines — it's prepended to every request.

---

## 3. DAILY — the loop you actually repeat

```bash
$ cd ~/work/erp
$ claude                       # context auto-injects. Say nothing. Just start.
```

Work normally. Then:

```
> /wrap                        # ← the ONE thing you must not skip
```

```bash
$ git add -A && git commit -m "feat: whatever you did"     # log line appends automatically
```

That's it. Three steps: `claude` → `/wrap` → `commit`.

`/wrap` is **once per session**, not per change. Ten edits over two hours = one `/wrap`.

### Add these when the situation arises

| Situation | Command | Where |
|---|---|---|
| about to touch something risky | `/preflight <what you're changing>` | in claude |
| before committing something non-trivial | `/verify` | in claude |
| you made a real design decision | `/adr <what you decided>` | in claude |
| AI broke something / you reverted | `/incident <what happened>` | in claude |
| lost the thread mid-session | `/catchup` | in claude |
| "what breaks if I change this?" | `llmw impact services/inventory.py` | terminal |
| "where is GST implemented?" | `llmw feature GST` | terminal |
| "what's the debt here?" | `llmw todos` | terminal |
| "when did this become a microservice?" | `llmw timeline` | terminal |
| working in claude.ai chat (no file access) | `llmw brief \| pbcopy`, paste as first message | terminal |

`/incident` is the highest-value one. Every entry is a paragraph of prompting you never
type again.

---

## 4. WEEKLY — across all projects

```bash
$ llmw status
```

```
PROJECT       BRANCH     STATE  LOG   COMMIT  VERDICT
erp           main       1d     0d    0d      ok
crm           feat/auth  20d    0d    0d      DRIFTED (code moved, state didn't)
pos           main       45d    45d   60d     dormant
```

For anything `DRIFTED` or `STALE`:

```bash
$ cd <that project> && claude
```

```
> /catchup
> /wrap
```

Then `llmw sync` to refresh the hub index.

---

## Repairing a project that's half set up

Symptoms: `llmw sync` shows an empty table · project missing from `llmw status` ·
`.context/standards` isn't a symlink · fewer than 10 files in `.context/`.

```bash
$ cd <project>
$ llmw init        # backfills missing files, repairs symlink/hook/hub page,
                   # prepends standards imports to CLAUDE.md (backup: CLAUDE.md.bak)
$ llmw doctor      # should now be 11 ok, 0 failed
$ llmw sync
$ git add -A && git commit -m "chore: complete context setup"
```

Nothing you or Claude already wrote is touched — `init` only fills gaps.

---

## The minimum viable version

If you remember nothing else from this file:

```
new/existing project:   llmw init  →  /onboard  →  review rules.md  →  commit
every session:          claude  →  work  →  /wrap  →  commit
every Monday:           llmw status
```

Everything else is optional. These three lines are the system.
