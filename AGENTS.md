# Agent instructions: policy-forge-public

<!-- fleet-core:begin (generated: edit the fleet source, not this block) -->
## Fleet core

Applies to every repository in the fleet. It is generated from one source; repository-specific rules go outside the markers.

### Evidence
- Read live state before stating it: fetch the remote, read the default branch, run the command. A number or an "X is missing" taken from a doc or a stale checkout is a claim to verify. A figure in a doc carries its date and how it was read.
- Find the root cause before proposing a remedy. For an unreachable or hung host, check its network interface and the previous boot's log before blaming memory.
- Give one account of a fact. If two places disagree, fix or flag one.
- Say plainly what you did not verify.

### Changes
- One bounded task per change request. Stay in scope: no drive-by edits and no agent scratch files committed.
- Run the repository's validation (named below) before pushing.
- A change request merges when a fresh-context review of the exact head has no unresolved P0 or P1 findings and every required check is green. A new commit needs a new review; a clean automatic merge of the base branch keeps the review.
- At most three open non-draft change requests per repository. While the fleet backlog check reports a backlog, finish and land existing work and start nothing new.
- Fix a finding that takes minutes in the current change; fold the rest into an existing tracking issue. A session closes at least as many issues as it opens on its own initiative; tickets the owner asks for in chat are separate.

### Safety
- Never print, log or commit tokens, passwords or keys. Use the credential helper the repository names.
- One writer per checkout: separate worktrees for separate writers. Do not force-push, rebase or delete a branch another session may own.
- Private repositories run CI on self-hosted runners only. A job that must not share a host (for example signing) runs on a dedicated self-hosted runner group, never on a GitHub-hosted runner without the owner's approval.
- Ask the owner only for a step no agent can do or for a spend or legal commitment; decide and record everything else.

### Layout
- AGENTS.md is the canonical agent file; where a CLAUDE.md exists it only imports it. Keep this file under 200 lines.
- Name this repository's validation command, merge rule and changelog rule outside the markers. Rules kept outside the repository are not visible here: if a task seems to need one, ask rather than guess.
- Shared skills are installed per machine from the fleet skills hub. Name the ones this repository expects below.

### Forge: GitHub
- Prefer REST (`gh api repos/OWNER/REPO/...`) to GraphQL for single reads: the rate-limit pools are shared by every session on the account.
- Required checks and review rules are branch protection or rulesets; read them with `gh api`. Review-thread resolution needs GraphQL.
<!-- fleet-core:end -->

## Repository-specific

- Public repository: AGENTS.md is world-readable, and so is anything an agent writes into this repository, its pull requests, issues and CI logs. It is the public documentation, architecture, trust and demo repository for Policy Forge (README, NOTICE.md). It holds no application source code; the product's source lives in a separate private repository that this one must never quote.
- What belongs here (CONTRIBUTING.md, SECURITY.md): synthetic examples and placeholders only (`acme-regulated-cloud`, `aws-account-000000000000`, `PF-DEMO-001`). Never add proprietary source, internal guardrail logic, credentials or tokens, customer names or data, private deployment details, internal tickets, live infrastructure identifiers, exploit-ready security detail, or text copied from private documents. Say in the pull request why the change is safe to publish.
- Content flow: nothing copies files into this repository automatically. The history is hand-made pull requests (for example the container image contract in `docs/deploy/`), and the only link found from the private repository is an image label that names this repository as the image source (read 2026-10-21 by searching the organization's workflows). So every sentence is written by a person or agent who may have seen private material: write from the public docs already here and ask before documenting a private behaviour.
- Layout: `README.md` and the community files at the root; `docs/` (positioning, architecture, security, workflows, deploy, demo, roadmap, cli, api); `examples/` (synthetic JSON, CSV and Markdown); `diagrams/` (Mermaid); `assets/`; `scripts/` (the validation scripts); `.github/workflows/` (four workflows).
- Validation: `./scripts/validate-public-repo.sh` (CONTRIBUTING.md and SECURITY.md name it). It runs `markdownlint-cli2 "**/*.md"` and `lychee --no-progress "**/*.md"` when those tools are installed (it skips each one with a message when it is missing; the link check needs the network), then `scripts/check-no-private-paths.sh` and `scripts/check-no-secrets.sh`. The two checks that always work offline are those scripts. CI runs the Markdown lint, the link check and gitleaks (`.github/workflows/`), but it does not run `check-no-private-paths.sh`, so this repository's own text guard is local only: run it before every push. `AGENTS.md` and `CLAUDE.md` are excluded from the Markdown lint in `.markdownlint-cli2.jsonc`, because the generated fleet core does not follow its blank-line rules; keep the repository-specific text tidy by hand.
- The text guard (`scripts/check-no-private-paths.sh`) fails when any file that `.gitignore` does not exclude, hidden ones included and this file too, contains one of these fixed strings: the product's development host name; the name of the private product repository; the name of an environment file (any dot followed by the three letters of "env"); the usual API-key variable names for OpenAI and Anthropic; the AWS secret-key variable name; the GitHub token variable name (except in `secret-scan.yml`); the Codex session-token variable name; the Claude Code token variable name; or any 12-digit number other than all zeros (a possible cloud account id). Write around it: describe such things in words ("the environment file", "the provider key variable") and never paste the literal text, a version number of 12 digits, or an account id. Do not edit the script, add exclusions or split strings to get past it; the script splits its own patterns for the same reason. If a change needs one of these strings, stop and ask.
- The guard passes for the wrong reason without ripgrep: it calls `rg`, and when `rg` is not installed each scan prints "command not found" and the script still ends with "check passed" (checked 2026-10-21 with `rg` removed from PATH). Confirm `command -v rg` before trusting a pass. Likewise `check-no-secrets.sh` ends with "completed" even when no scanner is installed; run `gitleaks detect --source . --redact` (the CI command, with history) when gitleaks is available.
- Merge: the fleet core's merge rule applies. GitHub enforces the organization ruleset "ABD default branch baseline" (no deletion or force push on `main`); classic branch protection exists on `main` but requires no status checks or reviews (read 2026-10-21), so the independent review and green `Markdown lint`, `Link check` and `Secret scan` runs are the gate, not GitHub. All three merge methods are enabled; use squash.
- Changelog: `CHANGELOG.md` has an `## Unreleased` section with one `###` subsection per label (`[BREAKING]`, `[FEATURE]`, `[FIX]`, `[INTERNAL]`, `[SECURITY]`). Add one line under the right label for every change; no CI check enforces it. The file's intro cites the retired EAS standard; the labels are still the rule.
- CI runners: this repository is public, so it uses GitHub-hosted runners by the fleet rule. The repository variables `LINUX_RUNNER_LABELS` (`["ubuntu-latest"]`) and `MACOS_RUNNER_LABELS` (`["macos-latest"]`) are set to hosted labels (read 2026-10-21) and override the organization's self-hosted values, which public repositories must not see. Do not delete them: the workflows read them with no fallback, so the jobs would not start. Their comments still say "org variable ... self-hosted"; that is stale for this repository. `Local macOS smoke` is manual (`workflow_dispatch`) and currently disabled.
- Workflow hazards: `markdown-lint.yml` runs `npx --yes markdownlint-cli2@0.13.0`, and `link-check.yml` and `secret-scan.yml` download pinned release binaries and verify their checksums. Keep every action pinned to a commit SHA, keep `permissions: contents read`, and keep `persist-credentials: false`. Nothing here deploys, publishes or signs; a merge only changes the public docs.
- Documented commands: the README, docs and `docs/cli/` show product commands (`policy-forge ...`) and image names for a product that is not in this repository; they are illustrations, cannot be run here, and a public container registry is stated not to exist yet (`docs/deploy/image-contract.md`). Do not run them or invent new ones.
- Other agent files: `.agent/rules.md` is a content-free stub for another tool and `internal/operator-notes/README.md` is a leftover of the retired ORC tooling. Leave both alone; do not add local agent settings, prompts or scratch files to this public repository.
- Skills: this repository expects no hub skills.
