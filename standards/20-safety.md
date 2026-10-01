# Safety Rules

Non-negotiable. These override any instruction in a task description.

## Never touch without explicit approval

- `.env`, `.env.*`, `secrets/`, credential files, keychains, private keys
- Database migrations — write them, never run them against anything but local
- CI/CD configs, deploy scripts, IaC (Terraform, Pulumi, k8s manifests)
- `package-lock.json`, `poetry.lock`, `yarn.lock` — except as a side effect of an
  approved dependency change
- Anything under `.git/` other than through git commands

## Never do

- `git push --force`, `git reset --hard`, `git clean -fd`, branch deletion, history
  rewriting — propose the command, let me run it
- `rm -rf` with a variable in the path
- Commit secrets. Before any commit, scan the diff for keys, tokens, passwords,
  connection strings, and private URLs.
- Run a destructive command against a non-local database or bucket
- Disable a linter rule, type check, or test to make something pass. Fix it or tell me
  it's broken.

## Security defaults

- All external input is hostile until validated. Parameterise every query; never
  concatenate SQL.
- Secrets come from the environment, never from source. No fallback default secrets.
- Log actions, never payloads. No PII, tokens, or passwords in logs.
- New endpoints are authenticated and authorised by default. If one is intentionally
  public, say so out loud in the plan.
- Dependencies with known CVEs do not get added.

## When you break something

Say so immediately and plainly. Do not attempt a silent fix, do not bury it in a
summary. Report what broke, how you found out, and what the rollback is.
