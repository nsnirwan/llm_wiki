# Setting Up on a New Machine

The important thing to understand first: **almost nothing needs migrating.**

Each project's `.context/` is committed inside that project's own repo. Clone the
project, and its overview, rules, constraints, glossary, decisions, lessons, log, and
state all come back with it. That was the whole point of not centralizing them.

What actually has to be set up is the *plumbing*: the CLI, the slash commands, the
hooks, and the hub's record of where each project now lives on disk.

---

## 1. Prerequisites

```bash
git --version        # any recent version
python3 --version    # 3.8+ — used by the installer to merge settings.json safely
claude --version     # Claude Code installed and logged in
```

macOS: `readlink -f` needs macOS 12.3+. On older versions, `brew install coreutils`.

---

## 2. Clone the hub

```bash
mkdir -p ~/projects && cd ~/projects
git clone git@github.com:<you>/llm_wiki.git
```

The path doesn't have to match your old machine — every script resolves the hub from
its own location. `~/projects/llm_wiki` is just the convention.

---

## 3. Run the installer

```bash
~/projects/llm_wiki/bin/llmw-install
```

It does four things and reports each one:

1. Symlinks `llmw` into `/usr/local/bin` (or `~/.local/bin` if that isn't writable —
   it'll tell you to add it to your PATH if needed).
2. Symlinks all slash commands into `~/.claude/commands/` and the `code-explorer`
   subagent into `~/.claude/agents/`.
3. Merges the `SessionStart` and `SessionEnd` hooks into `~/.claude/settings.json` —
   **preserving everything already in that file**, and saving a `.bak` first. If the
   file isn't valid JSON it stops rather than damaging it.
4. Runs `llmw doctor`.

The global section should come back all `ok`. The per-project section will fail —
expected, you're not in a project yet.

---

## 4. Clone your projects, then adopt each one

```bash
cd ~/projects
git clone git@github.com:<you>/traumafrei-web.git
git clone git@github.com:<you>/munir_elearn.git
```

Then, in each:

```bash
cd ~/projects/traumafrei-web
llmw adopt
```

**`adopt` is the new-machine command.** It handles the three things that are
machine-specific:

- **The hub page path.** `projects/traumafrei-web.md` still records the *old* machine's
  absolute path. `adopt` rewrites it to where the project actually is now. Without this,
  `llmw status` shows `MISSING` and `llmw sync` produces empty rows.
- **The standards symlink.** `.context/standards` pointed at the old machine's hub. It's
  a symlink, so it may be committed but dangling. `adopt` deletes and recreates it.
- **The git post-commit hook.** Hooks live in `.git/hooks/`, which git does **not**
  clone. Every fresh clone needs this reinstalled — `adopt` does it.

Then verify:

```bash
llmw doctor          # want 13 ok, 0 failed
```

Repeat for each project. Finally:

```bash
llmw sync            # rebuild the index with the new paths
cd ~/projects/llm_wiki && git add -A && git commit -m "chore: adopt on new machine"
```

---

## 5. Confirm it's actually live

```bash
cd ~/projects/traumafrei-web
llmw session-start </dev/null
```

You should see branch, dirty files, recent commits, `state.md`, the last log entries,
and the standing rules block. If that prints, the injection works.

Then the real test — open Claude Code and type `/catchup`. If it answers without
grepping around the repo, the hook fired and you're done.

---

## Things that trip people up

**`.gitignore` swallowing the context.** Some projects ignore dotfile directories. Check
that `.context/` is actually tracked:

```bash
git ls-files .context/ | head
```

If that's empty, the context was never committed on the old machine and it is gone.
Run `llmw init` and `/onboard` to rebuild.

**Slash commands not appearing.** They're symlinks into `~/.claude/commands/`. If
Claude Code was already running during install, restart it.

**Standing rules not injecting.** They're extracted from `## Standing rules` in
`.context/lessons.md`. If you renamed that heading, the extraction won't match.

**A project you never pushed.** The hub page will point at a path that doesn't exist,
and `llmw status` will show `MISSING`. Either clone it and `llmw adopt`, or delete
`~/projects/llm_wiki/projects/<name>.md`.

---

## The short version

```bash
git clone <hub-repo> ~/projects/llm_wiki
~/projects/llm_wiki/bin/llmw-install

# per project, after cloning it:
cd <project> && llmw adopt && llmw doctor

llmw sync
```

Five minutes total. The knowledge itself needs no migration — it rode along in the
project repos.
