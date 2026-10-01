# Code Standards

Defaults. A project may override any of these in its own `.context/overview.md` —
project-local rules always win.

## Style

- Follow the conventions already present in the file you are editing. The existing
  code is the style guide; this document is only the tiebreaker.
- Match the project's formatter and linter. Never reformat code you did not otherwise
  change.
- Names: descriptive over short. `userAccountBalance` beats `uab`. No single letters
  outside loop counters and math.
- Comments explain *why*, never *what*. If you need a comment to explain what the code
  does, rewrite the code.

## Structure

- Functions do one thing. If you need "and" to describe it, split it.
- Prefer pure functions; push side effects to the edges.
- No new abstraction layer until there are three concrete cases. Two is a coincidence.
- Dead code gets deleted, not commented out. Git remembers.

## Error handling

- Never swallow an exception silently. Catch narrowly, handle or re-raise.
- Errors carry context: what was attempted, with what input, why it failed.
- Fail fast on programmer error (bad arguments, impossible state). Fail gracefully on
  environmental error (network, disk, user input).
- Validate at the boundary — API handlers, CLI entry points, file parsers. Trust
  internals.

## Tests

- Every bug fix starts with a failing test that reproduces it.
- Test behaviour, not implementation. If a refactor breaks the test but not the
  feature, the test was wrong.
- Do not mock what you own. Mock the network, the clock, the filesystem — not your own
  modules.
- Never modify a test to make it pass. Either the code is wrong or the test's
  expectation was wrong, and you must say which.

## Dependencies

- Ask before adding one. Always name the alternative you considered and why the
  dependency wins.
- Prefer the standard library. Prefer one well-maintained dependency over three small
  ones.
- Pin versions. Never upgrade a major version as a side effect of another task.
