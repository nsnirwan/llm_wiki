# Managing Multiple Projects With AI Agents — Research & Recommendation

**Author:** compiled for Rajeev · **Date:** 2026-08-04
**Question:** What is the best way to run many AI-assisted projects in parallel so that (a) tokens stay cheap, (b) old context is recoverable, (c) every session knows where the last one stopped, and (d) basic instructions never have to be re-typed?

---

## 1. The five families of solutions

Everything in this space is one of five patterns. Most tools are a combination.

### A. Instruction files — `AGENTS.md` / `CLAUDE.md`

A markdown file at the repo root that the agent auto-loads. It tells the agent how to build, test, and behave.

`AGENTS.md` became the de-facto standard: formalised as an open spec in August 2025 (led by OpenAI with Google, Cursor and Factory), donated to the Linux Foundation's Agentic AI Foundation in December 2025, and by then adopted by 60,000+ open-source projects and 20+ agent tools — Claude Code, Codex CLI, Cursor, Aider, Devin, Copilot, Gemini CLI, Windsurf, Amazon Q.

- **Solves:** #10 inconsistent style, "stop re-prompting basics", partial #2 architecture.
- **Fails at:** memory over time. It's static; it doesn't know what happened yesterday.
- **The trap:** this file is prepended to *every* request. It is the most expensive file you own — you pay for it on every turn, forever. Keep it small.

### B. Persistent memory layers (MCP servers)

Mem0/OpenMemory, Zep/Graphiti, Basic Memory, mcp-memory-service, the official Knowledge Graph server. Structured stores the agent writes facts into and retrieves from.

Benchmarks: Zep/Graphiti scores 63.8% on LongMemEval's temporal-reasoning split vs Mem0's 49.0%, because its temporal knowledge graph tracks how facts *change* rather than just storing them. Mem0 has the biggest ecosystem (41k stars, chosen by AWS as exclusive memory provider for their Agent SDK).

- **Solves:** #1 forgetting, #7 cross-device, #14 task history.
- **Fails at:** inspectability. When retrieval misses, you get a number, not a reason — the cause could be a chunk boundary, a stale index, or an unlucky neighbour, none of which you can see. Zep and Mem0 cloud also route your code context through a third party.
- **Verdict:** real capability, wrong first step. Adds a daemon, a database, and an opaque failure mode to a problem that plain files still solve at your scale.

### C. Spec-driven development frameworks

The spec is the source of truth; code is the artifact. Write what you want → plan → decompose into tasks → let the agent implement.

- **GitHub Spec Kit** (80k+ stars, templates for 24+ agents). Its core idea worth stealing: a **"constitution"** — a markdown file of immutable principles that apply to every change in every session. That is exactly your "universal instruction" requirement.
- **OpenSpec** (52k stars, most actively maintained).
- **BMAD-METHOD** (48k stars) — six named persona agents across a full SDLC. Powerful and heavy: reported at $2,000+/month/developer in API spend. That is the opposite of your token goal.

- **Solves:** #4 decisions, #19 impact analysis, #9 duplicate work.
- **Fails at:** overhead. Full SDD on a solo multi-project workload is ceremony you will abandon in three weeks.

### D. Semantic retrieval over code (LSP / symbol-level)

**Serena** is the reference implementation: an MCP toolkit that exposes Language Server Protocol capabilities so the agent explores code at the *symbol* level — find this function, list its callers, edit this method — instead of reading whole files or grepping.

- **Solves:** #6 token limits, #8 hallucinated structure, #11 dependency graph, #3 unconsidered dependencies.
- **Key insight:** this is the correct answer for *code* facts. Symbol graphs and call hierarchies should be **queried live, never written into a document.** A hand-maintained dependency graph is stale the day after you write it.

### E. LLM-maintained wiki (the Karpathy `llm-wiki` pattern)

Three layers — immutable raw sources / an LLM-written markdown wiki / a schema file — plus three operations (ingest, query, lint) and two navigation files (`index.md` catalog, `log.md` chronology). No embeddings: read the index, drill into 3–5 pages.

- **Solves:** #1, #4, #5, #13, #14, #15, #17 — the "why" and "where were we" layer.
- **Caveat for your use case:** Karpathy's raw sources are *immutable articles*. Yours is *code that changes hourly*. Applied naively to code, the wiki drifts and becomes actively harmful — a confidently wrong doc is worse than no doc, because the agent trusts it.
- A commenter on the gist (`sturlese`/hippocampus) reports the plain-file version breaks down "somewhere in the low thousands of pages, when the index no longer fits in context." You are nowhere near that.

---

## 2. The two findings that should drive your design

### Finding 1 — Progressive disclosure beats loading everything

Context engineering was formalised by Anthropic in September 2025 as "curating and maintaining the optimal set of tokens during LLM inference." The measured effect it counters is **context rot**: agents lose effective IQ when the window is stuffed with irrelevant data. Loading all your docs upfront doesn't just cost money — it degrades answer quality, because the model spends capacity filtering noise before it starts.

The Agent Skills mechanism is the cleanest implementation: only name + description load at startup (~80 tokens each); the full body (275–8,000 tokens) loads only when the model judges it relevant; scripts and references load only at execution.

**Design consequence:** your instruction file should be a *table of contents with rules*, not a knowledge base. Everything else loads on demand.

### Finding 2 — Subagents are the cheapest context isolation available

A subagent runs in its own window with its own tools, then returns only a conclusion. Reported measurement: **subagents process ~67% fewer tokens than the equivalent Skills-in-main-context approach**, purely from isolation — the exploration transcript never pollutes the parent.

**Design consequence:** "go read the codebase and tell me how X works" should always be delegated, never done in your main window.

---

## 3. Scoring the options against your actual requirements

| | Save tokens | Recover old refs | Know where you left off | Universal instructions | Setup cost | Drift risk |
|---|---|---|---|---|---|---|
| AGENTS.md alone | ~ (can hurt if bloated) | ✗ | ✗ | ✓✓ | minutes | low |
| Memory MCP | ✓ | ✓ | ✓✓ | ✗ | high | opaque |
| Spec-driven (Spec Kit) | ✗ | ✓✓ | ✓ | ✓✓ | medium | low |
| Spec-driven (BMAD) | ✗✗ | ✓✓ | ✓✓ | ✓✓ | high | low |
| Serena / LSP | ✓✓ | ✗ | ✗ | ✗ | low | **none** |
| llm-wiki pattern | ✓✓ | ✓✓ | ✓✓ | ✓ | low | **high for code** |
| **Hybrid (recommended)** | ✓✓ | ✓✓ | ✓✓ | ✓✓ | low | managed |

No single family covers all four of your requirements. The hybrid is not a compromise — each family is genuinely best-in-class at a different one.

---

## 4. The governing rule

Everything below follows from one line:

> **Write down only what cannot be cheaply re-derived from the code.**

| Category | Where it lives | Why |
|---|---|---|
| Why we chose X over Y | `decisions.md` — **write it** | Lost forever otherwise; not in the code |
| Business rules, invariants | `rules.md` — **write it** | Implicit in code, expensive to reverse-engineer, dangerous to get wrong |
| Where we stopped, next step | `state.md` — **write it** | Exists only in your head |
| Known bugs, gotchas, dead ends | `state.md` / `log.md` — **write it** | Otherwise you re-investigate them |
| File inventory, module list | **derive** (`git ls-files`) | Stale in a day |
| Dependency graph, call hierarchy | **derive** (Serena/LSP) | Stale in an hour |
| API signatures, types | **derive** (read the file) | The code *is* the doc |
| Test coverage, build status | **derive** (run it) | Only the run is true |

Roughly half of your original 60-item list is in the "derive" column. Trying to maintain those as documents is where systems like this die.

---

## 5. Recommended architecture

**Per-project context lives in the project repo. A thin hub aggregates it.**

The reason not to put everything in one central repo: context outside the code repo cannot be committed atomically with the change it describes, so it drifts and code review never catches it. Context *inside* the repo versions with the code — and `git log`/`git show` gives you exactly your "recover a reference from an old commit" requirement, for free, per project.

```
~/projects/
├── llm_wiki/                    ← the hub (this repo)
│   ├── standards/               ← universal rules, ONE copy, symlinked out
│   ├── projects/<name>.md       ← per-project hub page
│   ├── index.md   log.md        ← catalog + global chronology
│   ├── bin/llmw                 ← the CLI
│   └── templates/               ← what gets installed into a project
│
└── <any-project>/
    ├── CLAUDE.md                ← THIN. loaded every turn. ~40 lines.
    └── .context/
        ├── state.md             ← where we left off  ← the resume file
        ├── decisions.md         ← append-only ADR log
        ├── rules.md             ← business invariants
        ├── overview.md          ← stack, entry points, how to run
        └── log.md               ← append-only session chronology
```

**Loading strategy (this is what saves the tokens):**

| Layer | Size | When loaded |
|---|---|---|
| `CLAUDE.md` | ~400 tok | every turn — kept deliberately tiny |
| `standards/*.md` | ~1,200 tok | once at session start, via symlink + `@import` |
| `state.md` + last 5 log lines | ~300 tok | auto-injected by the SessionStart hook |
| `decisions.md`, `rules.md`, `overview.md` | 1–3k tok each | **only when the agent decides it needs them** |
| code structure | 0 | derived live, never stored |

Session baseline lands around 2k tokens of project context instead of 20k+ of loaded documentation — and, per Finding 1, the answers are *better*, not just cheaper.

**Automation (Claude Code hooks):**

- `SessionStart` → its stdout is injected directly as context. Runs `llmw session-start`, which prints branch, dirty files, `state.md`, and the last 5 log lines. This is the mechanism that answers "where did I leave off" with zero prompting.
- `SessionEnd` → runs `llmw session-end`, appending a factual entry (commits made, branch, duration) to `.context/log.md`. It cannot ask the model to summarise, so the narrative half is a `/wrap` slash command you run before quitting.
- `post-commit` (git) → one-line append to the log. Cheap, always accurate.

**Explicitly rejected:** updating after *every* command. It burns tokens and floods the log with noise, polluting the very context you're protecting. Update at meaningful boundaries — commit and session end.

---

## 6. What to do, in order

1. **Week 1** — install the hub + `.context/` in your two most active projects. Write `state.md` by hand once. Turn on the hooks. *This alone solves "where did I leave off" and "stop re-prompting basics."*
2. **Week 2** — add Serena (`uvx --from git+https://github.com/oraios/serena serena start-mcp-server`). This is the single biggest token win, and it has zero drift risk because it derives everything live.
3. **Week 3** — start using `/adr` whenever you make a real decision. Add a `code-explorer` subagent for "how does X work" questions so exploration never enters your main context.
4. **Month 2+** — only if you feel the pain: a search tool over the wiki (qmd) when the index stops fitting; a memory MCP when files genuinely stop scaling. Do not start here.

**Do not build:** the dependency-graph doc, the module-ownership map, the impact-analysis document, the file inventory. Those are Serena queries and shell scripts, not documents.

---

## Sources

- [karpathy/llm-wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [AGENTS.md Spec (2026): Recommended Sections + AGENTS.md vs CLAUDE.md vs .cursorrules](https://www.morphllm.com/agents-md-guide)
- [AGENTS.md Complete Guide for Engineering Teams (2026)](https://blog.buildbetter.ai/agents-md-complete-guide-for-engineering-teams-in-2026/)
- [Claude Code Token Management: Save 50-70% on Pro Plan](https://richardporter.dev/blog/claude-code-token-management)
- [Practical workflow for reducing token usage in Claude Code](https://gist.github.com/dholdaway/8009f089d3407e14f3d753f2a70eb63e)
- [AI Agent Memory 2026: Progress Benchmark Report](https://mem0.ai/blog/state-of-ai-agent-memory-2026)
- [Best Memory & Knowledge MCP Servers in 2026 — ChatForest](https://chatforest.com/guides/best-memory-mcp-servers/)
- [AI Agent Memory Systems in 2026: Mem0, Zep, Hindsight, Memvid Compared](https://blog.devgenius.io/ai-agent-memory-systems-in-2026-mem0-zep-hindsight-memvid-and-everything-in-between-compared-96e35b818da8)
- [BMAD vs Spec Kit vs OpenSpec: Choosing Your Spec-Driven AI Framework in 2026](https://reenbit.com/bmad-vs-spec-kit-vs-openspec-choosing-your-spec-driven-ai-framework/)
- [9 Best AI Tools for Spec-Driven Development in 2026 — MarkTechPost](https://www.marktechpost.com/2026/05/08/9-best-ai-tools-for-spec-driven-development-in-2026-kiro-bmad-gsd-and-more-compare/)
- [oraios/serena — semantic retrieval MCP toolkit](https://github.com/oraios/serena)
- [Serena Review 2026: Semantic Code Retrieval Layer](https://vibecodinghub.org/tools/serena)
- [State of Context Engineering in 2026 — Aurimas Griciūnas](https://www.newsletter.swirlai.com/p/state-of-context-engineering-in-2026)
- [Progressive context disclosure — Thoughtworks Technology Radar](https://www.thoughtworks.com/en-us/radar/techniques/progressive-context-disclosure)
- [Agent Skills: The Context Engineering Revolution](https://www.blog.brightcoding.dev/2026/07/08/agent-skills-the-context-engineering-revolution)
- [Claude Code Session Hooks: Auto-Load Context Every Time](https://claudefa.st/blog/tools/hooks/session-lifecycle-hooks)
- [Sonovore/claude-code-handoff](https://github.com/Sonovore/claude-code-handoff)
- [Introducing knowhub: Share AI Assistant Rules Across Repos](https://medium.com/@yujiisobe/introducing-knowhub-share-ai-assistant-rules-across-repos-17fb6b09c114)
- [RepoSwarm: Give AI Agents Context Across All Your Repos](https://robotpaper.ai/reposwarm-give-ai-agents-context-across-all-your-repos/)
