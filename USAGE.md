# Usage

Four questions, four sections.

---

## 1. How is this "centralized" if the files live in each project?

Two things are centralized; one deliberately isn't.

| Thing | Where | Why |
|---|---|---|
| **Rules** — how the AI must behave, in every project | hub only (`standards/`), symlinked out | one copy. Edit once, every project changes. Never copy-paste rules again. |
| **The dashboard** — what's happening across all projects | hub only (`index.md`, `projects/`) | one screen for "what's the state of everything" |
| **Project context** — state, decisions, rules, log | *inside each project repo* | must version atomically with the code |

That third row is the deliberate part. If `state.md` lived in the hub, it could not be
committed in the same commit as the code it describes — so it would drift, and code
review would never catch it. Keeping it in-repo also means each project's history is
recoverable independently:

```bash
git log --oneline -- .context/decisions.md      # every decision, over time
git show HEAD~20:.context/state.md              # what we were doing 20 commits ago
git show abc123:.context/rules.md               # the rules as of that commit
```

That is your "if I lose something locally, pick the reference from an old commit" —
and it works per-project, not as one giant blob.

**One-time install:**

```bash
ln -s ~/projects/llm_wiki/bin/llmw /usr/local/bin/llmw
mkdir -p ~/.claude/commands ~/.claude/agents
ln -s ~/projects/llm_wiki/commands/*.md ~/.claude/commands/
ln -s ~/projects/llm_wiki/agents/code-explorer.md ~/.claude/agents/
```

Then merge `hooks/settings.json` into `~/.claude/settings.json`, replacing `HUB_PATH`
with `/Users/rajeevkumarachra/projects/llm_wiki`. Because it goes in the **user-level**
settings file, the hooks fire in *every* project — including ones you haven't set up
yet, where they exit silently.

Verify: `llmw doctor`

**Editing the universal rules later:**

```bash
vim ~/projects/llm_wiki/standards/00-core.md
cd ~/projects/llm_wiki && git commit -am "rules: no unrelated refactors"
```

Live in every project on the next session. No propagation step — it's a symlink.

---

## 2. Fresh project

```bash
mkdir ~/projects/newthing && cd ~/projects/newthing
git init && git commit --allow-empty -m "init"

llmw init
```

That creates `.context/` (6 files), `CLAUDE.md`, the `standards/` symlink, the hub
page, and the git post-commit hook.

Then, before writing any code — this is the highest-value five minutes in the whole
system:

```bash
claude
```

```
> I'm building <what>. Read .context/ and fill in overview.md (architecture,
> stack, how to run) and rules.md (the domain invariants). Ask me what you
> need. Then /adr the stack choice.
```

Commit it:

```bash
git add .context CLAUDE.md && git commit -m "chore: project context"
```

From now on every session starts already knowing the project. You never re-explain it.

---

## 3. Existing project

Same first step, different second step — here the code already exists, so let the agent
read it instead of you dictating.

```bash
cd ~/projects/old-project
llmw init
claude
```

```
> /onboard
```

`/onboard` delegates the codebase exploration to the `code-explorer` subagent, so the
"reading 40 files" transcript never enters your main context window — you get the
findings, not the transcript. It fills `overview.md`, `rules.md`, and `state.md` from
the real code and recent git log, and marks anything it inferred rather than confirmed
with `(inferred)`.

**Then review it.** This is not optional. The agent will get some domain rules wrong,
and a wrong `rules.md` is worse than an empty one, because every future session will
trust it. Read `rules.md`, delete what's wrong, fix what's close.

```bash
llmw lint                                    # catches leftover template text
git add .context CLAUDE.md && git commit -m "chore: add project context"
```

**If the project already has a `CLAUDE.md`:** `llmw init` won't overwrite it. Add the
imports yourself at the top and move anything long out into `.context/overview.md`:

```markdown
@.context/standards/00-core.md
@.context/standards/10-code.md
@.context/standards/20-safety.md
```

Keep the file under ~60 lines. It's prepended to every request — you pay for it on
every turn, forever.

---

## 4. Verifying it's actually staying current

Three levels, cheapest first.

### `llmw status` — the one you'll actually use

```
PROJECT              BRANCH     STATE    LOG      COMMIT   VERDICT
-------------------- ---------- -------- -------- -------- -------
api-service          main       1d       0d       0d       ok
mobile-app           feat/auth  20d      0d       0d       DRIFTED (code moved, state didn't)
scraper              main       45d      45d      60d      dormant
billing              main       -        -        -        NO GIT HOOK
```

Columns are days since last change. The logic that matters:

| Verdict | Means | Fix |
|---|---|---|
| `ok` | state is keeping up with the code | — |
| `DRIFTED` | commits are newer than `state.md` by 3+ days — **you've been coding without wrapping** | `cd` there, `/catchup`, then `/wrap` |
| `STALE` | nothing touched in 14+ days but the project isn't dormant | same |
| `dormant` | no commits in 30+ days — stale state is expected, not a problem | — |
| `NO GIT HOOK` | the post-commit hook is missing, so commits aren't being logged | `llmw init` |
| `MISSING` | hub page points at a path that no longer exists | fix or delete `projects/<name>.md` |

`DRIFTED` is the important one. It compares `state.md`'s mtime against commit recency,
so it catches the actual failure mode — you kept coding and stopped writing things
down — rather than just "this file is old," which would false-positive on every
finished project.

### `llmw doctor` — run once after install, then when something feels broken

Checks the plumbing: CLI on PATH, slash commands linked, subagent linked, both hooks
present in `~/.claude/settings.json`, hub is a git repo, and for the current project:
`.context/` exists, standards symlinked, `CLAUDE.md` present, post-commit hook
executable, context actually committed. Every failure prints the exact command to fix it.

### `llmw lint` — content quality, inside one project

`status` asks "is it recent?"; `lint` asks "is it real?" It catches leftover `{{...}}`
placeholders, sections still full of template text, and a `CLAUDE.md` that's grown past
80 lines (which costs you tokens on every single turn).

### Proving the auto-update works end to end

```bash
cd ~/projects/some-project

# 1. does the SessionStart hook produce context?
llmw session-start </dev/null
#    → should print branch, dirty files, recent commits, state.md, last 5 log lines.
#    → if it prints nothing, .context/ is missing — run llmw init

# 2. does the commit hook log?
git commit --allow-empty -m "test: hook check"
tail -5 .context/log.md
#    → should show a new "## [date] commit | test: hook check" entry
git reset --hard HEAD~1   # then undo it

# 3. is the SessionStart hook actually wired into Claude Code?
#    open claude and type: /catchup
#    → if it answers without reading any files, the hook fired.
#    → if it starts grepping around to figure out where you are, it didn't —
#      run llmw doctor
```

### Making the check automatic

Two options, pick one:

```bash
# shell: see status every time you open a terminal
echo 'llmw status' >> ~/.zshrc

# or weekly, via cron
(crontab -l 2>/dev/null; echo "0 9 * * 1 /usr/local/bin/llmw sync") | crontab -
```

Or ask me to set up a scheduled task that runs `llmw status` every Monday and messages
you only if something has drifted.

---

## Daily rhythm, condensed

| When | You do | System does |
|---|---|---|
| open Claude Code | nothing | injects state + branch + last 5 log lines (~300 tokens) |
| lost the thread | `/catchup` | re-orients from already-loaded context, reads nothing |
| explore unfamiliar code | ask normally | `code-explorer` subagent handles it in isolated context |
| made a real decision | `/adr <decision>` | appends to `decisions.md` |
| any commit | nothing | git hook appends a log line |
| before quitting | `/wrap` | rewrites `state.md`, appends session entry |
| Monday morning | `llmw status` | shows which projects drifted |

The only two things you have to remember are `/wrap` before you stop and `llmw status`
occasionally. Everything else is automatic — and `status` tells you when you forgot.
