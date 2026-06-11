# Archgate Pre-commit Hook

A git pre-commit hook that runs [`archgate check --staged`](https://cli.archgate.dev/guides/pre-commit-hooks/) before each commit. Architecture violations are caught on your machine before they reach a pull request.

## Prerequisites

Install the [Archgate CLI](https://cli.archgate.dev):

```bash
curl -fsSL https://cli.archgate.dev/install-unix | sh
```

Your project needs an `.archgate/` directory with ADRs. Run `archgate init` or `archgate adr import` to set one up.

## Installation

### Quick install (single developer)

Copy the hook into your repo:

```bash
curl -fsSL https://raw.githubusercontent.com/archgate/pre-commit-hook/main/hooks/pre-commit \
  -o .git/hooks/pre-commit && chmod +x .git/hooks/pre-commit
```

### Team install (version-controlled)

Clone this hook into your repo and point git at it:

```bash
# Copy the hook script into your repo
mkdir -p .githooks
curl -fsSL https://raw.githubusercontent.com/archgate/pre-commit-hook/main/hooks/pre-commit \
  -o .githooks/pre-commit && chmod +x .githooks/pre-commit

# Tell git to use your hooks directory
git config core.hooksPath .githooks
```

Commit `.githooks/pre-commit` so every clone gets the hook. Each developer runs `git config core.hooksPath .githooks` once after cloning (or add it to your onboarding docs).

### pre-commit framework

If your team uses the [pre-commit](https://pre-commit.com) framework, add this to `.pre-commit-config.yaml`:

```yaml
repos:
  - repo: https://github.com/archgate/pre-commit-hook
    rev: v1.0.0
    hooks:
      - id: archgate-check
```

Then run:

```bash
pre-commit install
```

### Lefthook

Add the check to `lefthook.yml`:

```yaml
pre-commit:
  commands:
    adr-check:
      run: archgate check --staged
```

### Husky

Add to `.husky/pre-commit`:

```bash
archgate check --staged
```

## How it works

The hook runs `archgate check --staged`, which:

1. Lists files in the git staging area (`git diff --cached --name-only`)
2. Matches staged files against ADR `files` globs to find applicable rules
3. Runs each rule against the staged files
4. Exits with code **0** (commit proceeds) or **1** (commit blocked)

Because only staged files are checked, the hook stays fast. A typical check with 3 staged files completes in under a second.

### Exit codes

| Code | Meaning |
|------|---------|
| 0 | All checks pass |
| 1 | Violations found (commit blocked) |
| 2 | Rule execution error (e.g. rule timed out) |

### Useful flags

Pass flags by editing the hook script or your hook manager config:

| Flag | Purpose |
|------|---------|
| `--staged` | Check only git-staged files (required for pre-commit) |
| `--verbose` | Show passing rules and timing |
| `--adr <id>` | Check a single ADR (useful for debugging) |
| `--max-warnings <n>` | Fail if warnings exceed a threshold |

## Running alongside other hooks

The hook exits early on failure, so put it alongside your other pre-commit checks. With `core.hooksPath`, add more commands to the same script:

```bash
#!/bin/sh
set -e

npm run lint
npm run typecheck
archgate check --staged
```

With Lefthook, each command runs independently:

```yaml
pre-commit:
  commands:
    lint:
      run: npm run lint
    typecheck:
      run: npm run typecheck
    adr-check:
      run: archgate check --staged
```

## Bypassing the hook

Developers can skip the hook with `--no-verify`:

```bash
git commit --no-verify -m "emergency fix"
```

This is expected. The pre-commit hook is the first line of defense, not the only one. Keep `archgate check` running in CI as the authoritative backstop. See the [CI integration guide](https://cli.archgate.dev/guides/ci-integration/).

## Related

- [Pre-commit hooks guide](https://cli.archgate.dev/guides/pre-commit-hooks/) on cli.archgate.dev
- [CI integration guide](https://cli.archgate.dev/guides/ci-integration/) for the merge gate
- [archgate/check-action](https://github.com/archgate/check-action) for GitHub Actions CI
- [archgate/awesome-adrs](https://github.com/archgate/awesome-adrs) for curated ADR packs

## License

Apache-2.0
