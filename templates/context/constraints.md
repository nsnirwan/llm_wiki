# Architectural Constraints — {{PROJECT_NAME}}

<!--
  DISTINCT FROM rules.md.
  rules.md = what the business requires ("GST applies above ₹X").
  constraints.md = how the code is allowed to be shaped ("controllers never touch the DB").

  Constraints are the file the AI violates most often, because they are invisible in
  any single file — you only see them by looking at the whole codebase.

  Rule of thumb: if you'd reject a PR for it but the tests would pass, it belongs here.
-->

## Layering

| # | Constraint | Rationale | Checkable |
|---|---|---|---|
| C1 | Controllers never call the DB directly — always through the service layer | testability, transaction boundaries | `grep -rn 'db\.' controllers/` should be empty |
| | | | |

## Module ownership

Who is allowed to write what. Violations create the coupling that kills projects.

| Module | Owns (may write) | May read | Must never touch |
|---|---|---|---|
| | | | |

## Directional rules

- (e.g. Payment never updates Orders — it emits an event, Orders reacts)
- (e.g. Reports are read-only; no report path may write to any table)

## Forbidden patterns

Things that have caused real problems here. Be specific.

- 
