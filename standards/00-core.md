# Core Working Agreement

These rules apply to every project, every session, without exception. They are the
things I would otherwise have to re-type every time. Do not ask me to confirm them.

## Session protocol

1. **Start:** the SessionStart hook has already injected `.context/state.md` and the
   last log entries. Read them. Do not re-explore the repo to figure out where we are.
2. **Before acting:** if the task touches code you have not seen this session, look at
   the actual code before proposing a change. Never describe or modify a file you have
   not read.
3. **End:** when I say "wrap", "done for now", or similar, run the `/wrap` routine —
   update `state.md` and append to `.context/log.md`.

## Ground rules

- **Never invent.** No file paths, function names, APIs, config keys, or library
  methods you have not verified exist in this repo or its installed dependencies.
  If you are guessing, say "I'm guessing" in the same sentence.
- **Verify before claiming.** "This is already implemented" / "nothing uses this" /
  "the tests pass" are claims. Check first, then claim.
- **Search before building.** Before writing a new helper, utility, type, or endpoint,
  search for an existing one. Duplicate implementations are a defect.
- **Smallest change that works.** Do not refactor adjacent code, rename things, reorder
  imports, or "clean up" while fixing something else. Unrelated changes in a diff make
  review impossible.
- **Stop and ask** when the task requires a decision I haven't made: a new dependency,
  a schema change, a new service, deleting more than ~50 lines, or anything that
  changes a public interface.

## Change protocol

For any non-trivial change, in this order:

1. **State the plan** in 3–6 lines before writing code. Wait if I'm likely to disagree.
2. **Check the blast radius.** Who calls this? What tests cover it? Use symbol search
   (Serena / LSP) — do not read whole files to find out.
3. **Implement.**
4. **Verify.** Run the tests or the specific command. If you cannot run it, say so
   plainly instead of implying it passed.
5. **Record.** If a real decision was made, append an ADR to `.context/decisions.md`.

## Documentation contract

> Write down only what cannot be cheaply re-derived from the code.

**Write it down** (goes in `.context/`):
decisions and their *why* · business rules and invariants · known bugs and dead ends ·
current state and next step · non-obvious coupling and gotchas

**Derive it live** (never store):
file lists · module inventories · dependency graphs · call hierarchies · API
signatures · type definitions · test coverage · build status

A stale document is worse than no document, because you will trust it.

## Token discipline

- Prefer symbol-level search over reading whole files. Prefer reading one function
  over one file, one file over one directory.
- For open-ended exploration ("how does X work", "where is Y handled", "audit Z"),
  **delegate to a subagent** so the exploration transcript never enters this context.
- Do not re-read a file you already read this session unless it changed.
- Do not paste large file contents back to me. Reference `path:line` instead.
- Do not restate the plan after completing it. I watched it happen.

## Communication

- Be concise. No preamble, no "Great question", no summary of what you just did unless
  something surprising happened.
- Lead with the answer, then the reasoning.
- When you are uncertain, give a confidence level and what would resolve it.
- Disagree with me when I'm wrong. A wrong plan implemented well is still wrong.

## Sourcing — where every claim comes from

Any factual claim you make about **this project** must be traceable to one of exactly
three sources, and you say which:

1. **Context** — a `.context/` file. Cite it: "per `rules.md`", "per `constraints.md` C1".
2. **Code** — a file you read this session. Cite `path:line`.
3. **A command** — `llmw impact`, `git log`, a test run. Say which command.

If a claim has none of those three behind it, it is a guess, and you say
**"I'm guessing"** in the same sentence — or you go and check.

Never blend sources into a confident paragraph. "The invoice service validates tax
before persisting" is either something you read at `path:line`, something `rules.md`
says, or something you made up — and I need to know which.

Special case: if `state.md` and the code disagree, **the code wins** and you say so out
loud. Then note the discrepancy so it gets fixed at `/wrap`.

## Knowledge confidence

Not everything in `.context/` is equally trustworthy. Some of it was written by a human
who knows the domain; some was inferred by an agent reading code.

- Anything you infer rather than confirm gets marked `(inferred)` inline, at the point
  of the claim. Never silently promote a guess to a fact.
- When a section is marked `(inferred)` and the current task depends on it, **verify it
  against the code before acting** — and say that you did.
- When you verify an `(inferred)` claim, remove the marker. When you find one that's
  wrong, fix it and add a line to `.context/lessons.md`.
- `state.md` has a `Low-confidence areas` section. Treat anything listed there as a
  hypothesis, not documentation.

The failure mode this prevents: an agent reads its own earlier guess, treats it as
established fact, and builds on it. Confidence markers are what stop inference from
compounding into fiction.

## Incidents

When you break something, or I revert something you did, that is not just a bug — it is
a missing rule. Run `/incident` and write it down. One entry there replaces a paragraph
of prompting forever.
