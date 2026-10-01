# Hub Log

Global chronology across all projects. Newest at the bottom.
Greppable: `grep "^## \[" log.md | tail -10`

## [2026-08-04] init | context hub created

- Researched approaches; chose in-repo `.context/` + thin hub over memory MCP / SDD frameworks.
- Scaffolded standards, templates, `llmw` CLI, hooks, slash commands, code-explorer subagent.

## [2026-08-04] add | status + doctor commands

- Added `llmw status` (cross-project drift detection) and `llmw doctor` (install verification).
- Wrote USAGE.md covering centralization model, fresh/existing project onboarding, and verification.

## [2026-08-04] v2 | context types + impact engine

- Added constraints.md, glossary.md, lessons.md, features.md (anchors not file lists).
- ADR Status/Type fields; llmw timeline derives architecture evolution from them.
- Impact engine: static refs + git co-change coupling. Also feature/todos commands.
- New commands: /preflight /verify /incident. Confidence convention in 00-core.md.
- Reviewed 15 suggestions: 9 taken, 4 reshaped, 2 declined (see docs/v2-triage.md).

## [2026-08-05] fix | init backfills, onboard refuses without setup

- Bug: `llmw init` skipped template creation entirely when `.context/` existed, so a
  project scaffolded by `/onboard` never got glossary/features/lessons/index.
- Bug: `/onboard` created `.context/` itself when missing, silently skipping the
  standards symlink, git hook, and hub page registration. Found on traumafrei-web.
- Fixes: init now backfills per-file and prepends standards imports to an existing
  CLAUDE.md (with .bak); /onboard now hard-refuses unless `llmw doctor` is clean;
  doctor checks the import lines and all 10 context files. Cheatsheet gates on doctor.

## [2026-08-05] fix | standing rules now auto-injected; sed comment-strip bug

- Standing rules from lessons.md are now Tier 1 (injected by SessionStart), not
  on-demand. They were the highest-value content behind the least reliable loading path.
- Bug found while testing: `sed '/<!--/,/-->/d'` deletes to EOF when a comment opens
  and closes on the SAME line (sed looks for the end address from the next line on).
  This was silently truncating state.md injection too. Replaced with an awk
  strip_comments() helper that handles both forms.

## [2026-08-05] add | llmw-install and llmw adopt

- `bin/llmw-install`: one-shot machine setup (CLI, commands, subagent, hook merge with
  backup + JSON validation, doctor run).
- `llmw adopt`: re-points a hub page at the project's current path, refreshes the
  standards symlink, reinstalls the post-commit hook. Needed on every fresh clone
  because git does not clone .git/hooks and the symlink/path are machine-specific.
- docs/new-machine.md written.

## [2026-08-05] add | llmw brief + sourcing discipline

- `llmw brief [full]`: paste-able context block for surfaces with no file access
  (claude.ai chat). Includes explicit rules-of-engagement telling the model it cannot
  read the repo and must not invent paths.
- 00-core.md: added "Sourcing" — every project claim must cite context file, path:line,
  or a command, else be labelled a guess. Code wins over state.md on conflict.

## [2026-08-05] add | llmw-profile (multi-account)

- `bin/llmw-profile`: add / list / run / doctor / shell-init. Each profile is its own
  CLAUDE_CONFIG_DIR (separate credentials + transcripts) but shares settings.json by
  symlink so model/permissions/hooks stay in sync.
- Provisioning is the point: a bare CLAUDE_CONFIG_DIR has no slash commands, no agents
  and no SessionStart hook, so switching accounts silently disables the whole system.
  `add` installs the llmw layer into the profile.
- Fixed `llmw doctor` and `llmw-install` to honour CLAUDE_CONFIG_DIR (previously
  hard-coded $HOME/.claude, so they reported false failures under a profile).
