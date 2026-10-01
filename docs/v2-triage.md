# Review Response — What I Took, Reshaped, and Rejected

Your 15 suggestions, triaged against the governing rule. Nine went in close to as
written, four were reshaped, two I'd argue against.

---

## Your four-bucket taxonomy — adopted, with one correction

```
Permanent  → architecture, business rules, constraints, glossary
Derived    → dependency graph, APIs, module tree, feature files
Session    → current task, progress, next step
Historical → decisions, changelog, lessons, mistakes
```

This is right, and it's now the literal structure of `.context/index.md` and the
`CLAUDE.md` table.

**The correction:** *Derived isn't a storage bucket.* The other three answer "where does
this live"; Derived answers "don't store this, run this command." Filing it alongside
the others invites someone to eventually create a `derived/` folder and cache things
into it — which is exactly the failure the bucket exists to prevent. So in the
implementation it's a table of **commands**, not files.

**The addition:** each bucket needs an **expiry rule**, or the taxonomy is descriptive
rather than operational.

| Bucket | Expires when | Enforced by |
|---|---|---|
| Permanent | the architecture actually changes | `/verify` step 5 — checks if the diff invalidated a doc |
| Derived | instantly, every time | it's a command; there's nothing to expire |
| Session | every session | `/wrap` overwrites `state.md` |
| Historical | never — append only | `/adr`, `/incident` append; status fields mark what's no longer binding |

---

## Taken as written

**#3 Impact Engine** — you called this the biggest missing piece and you were right.
Now `llmw impact <path>`. Two independent signals:

1. **Static references** — who mentions it now.
2. **Historical coupling** — what has changed *with* it across the last 40 commits.

The second one is the part I want to draw attention to, because it finds coupling that
neither grep nor an LSP can see. In the test repo, static analysis found one referencing
file; co-change analysis surfaced four, including a test that names it nowhere. That
gap *is* hidden coupling — and it's free, from `git log`.

Serena/LSP and this are complements, not substitutes: LSP gives precise symbol callers,
git gives empirical coupling. Neither sees what the other sees.

**#5 + #6 AI Mistake Registry + Recovery Log** — merged into one file, `lessons.md`,
because they're the same event recorded twice. Two sections: a short **Standing rules**
list (imperative one-liners, loaded every session) and a longer **Incidents** log
(full detail, read only when relevant). New `/incident` command.

This is the single highest-leverage addition you proposed. Every entry is a paragraph of
prompting you never type again. One constraint I added: Standing rules is **capped at
~15 lines**. It's loaded every session, so an uncapped registry becomes the context bloat
it was meant to prevent. `/incident` will tell you which old rules have become obvious.

And a rule the command enforces: an incident that only produces a lessons entry will
happen again. `/incident` requires you to also fix the root cause — the missing line in
`rules.md`, `constraints.md`, or `glossary.md` that let it happen.

**#8 Domain Glossary** — `glossary.md`, structured around **disambiguation** rather than
definition. The valuable column isn't "means" but "does NOT mean" — your own example
(`Order` = business order, *not* the `order` table) is exactly the confusion that
causes real damage, and a plain dictionary wouldn't capture it.

**#9 Architectural Constraints** — `constraints.md`, correctly separate from
`rules.md`. Your framing was precise: business rules are *what the domain requires*,
constraints are *what shape the code is allowed to take*. My test for whether something
belongs here: **you'd reject a PR for it, but the tests would pass.**

Added a "Checkable" column — many constraints can be expressed as a grep that should
return nothing, which turns a document into a lint.

**#14 Decision Status** — five states on every ADR: Accepted, Experimental, Deprecated,
Superseded, Reverted. Cheapest high-value item on your list.

`Reverted` is the one I'd emphasize: entries recording things you tried and undid are
the most valuable in the file, because they're the ones that stop you re-trying. `/adr`
now explicitly forbids deleting them.

**#11 Cross-Project Knowledge** — `patterns/` in the hub, with a two-project minimum
(one occurrence is a solution, not a pattern) and a rule that pages hold **pointers,
not copies**. A copied snippet goes stale silently; a pointer to `project-a path/file`
stays honest because the file either exists or it doesn't.

**#7 Knowledge Confidence** — implemented as a **convention plus a field**, not a file.
`(inferred)` markers inline at the point of the claim, a `Low-confidence areas` section
in `state.md`, and a rule in `00-core.md` requiring verification before acting on an
inferred claim. Percentages would be false precision; a binary verified/inferred is
honest and actually gets maintained.

The failure mode this stops is worth naming: an agent reads its own earlier guess,
treats it as fact, and builds on it. Confidence markers are what keep inference from
compounding into fiction.

**#10 Session Goals** — added a `Goal` section above `Right now` in `state.md`. Not a
new file: your example was ~80% already covered by Right now / Next step / Blocked. The
genuinely missing piece was the multi-session *outcome* — "finish vendor sync" survives
several sessions while "right now" changes hourly.

**#4 Project Health** — split by derivability. TODO/FIXME/HACK/deprecated are now
`llmw todos` (derived, always accurate, groups by file so you can see where debt
concentrates). Human-judgement debt — "this module is unsafe," "don't touch reporting"
— stays in `state.md` and `constraints.md`. **Do not maintain a TODO document**; the
code is the document.

---

## Reshaped

**#1 Architecture Evolution Timeline** — you're right that "when did this become a
microservice" is a different question from "why." But it doesn't need a second file:
it's `decisions.md` sorted and filtered. Added a `Type` field
(architecture/stack/data/security/process/ops) and `llmw timeline`, which renders the
evolution and drops the noise. Verified in testing: a `process` ADR correctly doesn't
appear in the architecture timeline.

A separate timeline file would need manual sync with `decisions.md` and would drift
within a month.

**#2 Feature Map** — high value, and the highest drift risk on your whole list. A
feature → file-list map is wrong within days, and then it actively misleads.

So `features.md` stores **anchors, not inventories**: entry point, owning module, and a
**grep key**. The file list is derived by `llmw feature GST`, which runs the key against
the current tree — so it's never stale by construction. Tested: it resolves GST to the
live files, and if a file is added tomorrow it appears automatically with no maintenance.

The template says this explicitly: *if you find yourself listing files here, stop.*

**#12 Prompt Library** — right instinct, wrong artifact. A document full of prompts is
a document you have to remember to open. The same content as slash commands is one
keystroke and can't be forgotten. Your "before coding: analyze impact, generate tests,
rollback plan, regression risk" is now `/preflight`, which does exactly that and refuses
to start editing until you've seen the assessment.

Commands now: `/preflight` `/verify` `/incident` `/adr` `/wrap` `/catchup` `/onboard`.

**#15 AI Verification** — the checklist is valuable; a *document per task* is bloat that
nobody reads twice. It's now `/verify`, run before commit, and its output goes into the
log entry rather than a new file. It checks the diff for unintended files, runs tests
(and is required to say plainly if it *couldn't*), re-checks constraints, scans for
secrets, and flags which docs the change just invalidated.

---

## Where I'd push back

**#13 Release Knowledge** — skip for now. Version, changes, and breaking changes are
derivable from git tags and commit messages. Only "rollback plan" and "risk" are human,
and `/preflight` already produces both, per change, at the moment you actually know
them. A release doc for a solo developer across many projects is ceremony you'll abandon
by the third release — and an abandoned doc is worse than none.

Revisit if you start shipping versioned artifacts other people depend on. Then it's a
`CHANGELOG.md` generated from tags, not a context file.

**The general risk in the list as a whole.** You proposed 15 additions to a system with
6 files. Taken literally that's ~20 files per project, and the maintenance burden is
what kills wikis — the exact failure Karpathy's essay names. That's why roughly a third
of these became *commands* instead of files, and why the ones that did become files
(`glossary`, `constraints`, `lessons`) are all things that are impossible to derive and
cheap to write.

Current count: **9 files per project, 4 derived commands.** Anything further should have
to displace something.

---

## Is Obsidian useful here?

Short answer: **yes for the hub, no for the projects.** It's genuinely less useful for
this than for Karpathy's research wiki, and the reason is structural.

**Why it doesn't fit the project layer:**

- Your context is *distributed across many repos*, not gathered in one vault. Obsidian
  assumes a vault. You'd be fighting it.
- Nine files per project is well below the scale where graph view tells you anything.
  Graph view is for hundreds of interlinked pages.
- You're already in an editor and a terminal. A third window that shows the same
  markdown is friction, not leverage.
- Karpathy uses it because his wiki is the *product* — he reads it as the primary
  artifact. Your context files are *inputs to an agent*. You read them rarely, mostly
  to correct them.

**Where it earns its place — the hub:**

- **`patterns/` and cross-project browsing.** This is one directory with genuine
  link structure, and it grows. Backlinks answer "which projects use this auth pattern"
  natively.
- **Dataview over ADR status.** If you add YAML frontmatter to ADRs
  (`status: Experimental`, `type: architecture`), Dataview renders a live table of every
  experimental decision across every project. That's a real query you'd otherwise write
  a script for.
- **Reviewing `/onboard` output.** Reading and correcting a freshly generated
  `overview.md` + `rules.md` is genuinely nicer in a proper markdown editor with preview.

**If you want it:** open a vault on `~/projects/llm_wiki` only. Don't try to vault the
project repos — symlinking many `.context/` directories into one vault creates path
confusion for both Obsidian and the agent, for the sake of a graph view over nine files.

**Verdict:** worth 20 minutes for the hub, and only once `patterns/` has a few real
entries. It is not load-bearing, and nothing in this system depends on it. The tools
that would actually move the needle next are **Serena** (precise symbol-level retrieval,
the biggest remaining token win) and running `/preflight` consistently — not a notes app.
