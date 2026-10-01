# Lessons & Incident Registry — {{PROJECT_NAME}}

<!--
  APPEND ONLY. This is the highest-leverage file in the whole system.

  It combines two things that are really the same thing:
    - times an AI broke something, and
    - times you had to revert.

  Why it matters: every entry here is a prompt you no longer have to type. The rule
  extracted from an incident goes into the "Standing rules" section, which the agent
  reads at session start via CLAUDE.md.

  HOW IT'S LOADED: the SessionStart hook extracts the "Standing rules" bullets below
  and injects them into every session automatically (~150 tokens). The agent does not
  have to decide to open this file. The Incidents section is NOT injected — it loads
  on demand.

  Therefore: only `- ` bullet lines directly under "## Standing rules" get injected.
  Keep them one line each, imperative, no explanation.

  KEEP IT TIGHT. Cap "Standing rules" at ~15 lines — you pay for them every session.
  If it grows past that, the oldest entries have become obvious and should be deleted.
-->

## Standing rules (auto-injected at every session start)

<!-- One line each. Imperative. No explanation — the incident below has that. -->

- Never rename an API route. Add the new one and deprecate the old.
- Always verify migration ordering before generating a migration.
- Never modify tax/GST validation without asking — the rules are not all in code.

---

## Incidents

Format: `## [YYYY-MM-DD] <tool> — <what broke>`

### [{{DATE}}] example — template entry, delete me

- **What happened:** the agent removed a validation check while refactoring.
- **Why it happened:** the rule it enforced was not written anywhere it could see.
- **Blast radius:** N files / which module / did it ship?
- **Recovery:** `git revert abc123`, then re-added the check with a test.
- **Rule extracted:** → added to Standing rules above.
- **Prevention:** documented the rule in `rules.md` so it's visible next time.
