# Cross-Project Patterns

Solutions that recur across projects. This is the hub-level answer to "we already
solved this in the other project."

Add a pattern only when it appears in **two or more** projects. One occurrence is not a
pattern, it's a solution.

## Index

| Pattern | Used in | File |
|---|---|---|
| | | |

## Page format

```markdown
# <Pattern name>

**Used in:** project-a, project-b, project-c
**Status:** stable | evolving

## Problem
## Approach
## Reference implementation
project-a `path/to/file` — the cleanest version

## Gotchas
Things that went wrong in at least one project.

## Do not use when
```

Keep the reference a *pointer*, not a copy. Copied code here goes stale; a pointer to
`project-a path/to/file` stays honest because the file either exists or it doesn't.
