# Changelog

This file is the operator-facing release ledger. Every pull request adds one
line under `## Unreleased` with one of the allowed section labels, or
applies the `changelog-skip` label for a purely no-impact change. See EAS
§B2 Universal PR Essentials.

Allowed entry labels:

- `[BREAKING]` - incompatible API, data, deployment, or operator change.
- `[FEATURE]` - new user-visible or operator-visible capability.
- `[FIX]` - bug fix or compatibility repair.
- `[INTERNAL]` - tests, CI, docs, refactors, generated artifacts, or process.
- `[SECURITY]` - vulnerability fix, security control, or hardening change.

PR title prefix matches the highest-tier label added:
`[BREAKING]` -> `[major]`, `[FEATURE]` -> `[minor]`, others -> `[patch]`.

## Unreleased

### [BREAKING]

- None.

### [FEATURE]

- None.

### [FIX]

- None.

### [INTERNAL]

- Regenerate the fleet-core block in `AGENTS.md` to the current standard.
- Refresh the generated fleet core while preserving public repository guidance.

- Route the four workflows to the org's self-hosted runners (`LINUX_RUNNER_LABELS`, `MACOS_RUNNER_LABELS`); install Node via `actions/setup-node` and lychee and gitleaks as checksum-verified, architecture-matched binaries in user space.
- Adopt the fleet agent-file standard: `AGENTS.md` with the shared core and repository rules (including the public-repository text guard), and a `CLAUDE.md` that only imports it.
- CI hygiene: every workflow job has a `timeout-minutes` sized from its recorded run times, and the three pull-request workflows cancel a superseded pull-request run (push-to-main runs are never cancelled).

### [SECURITY]

- None.
