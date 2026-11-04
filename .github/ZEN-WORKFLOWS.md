# Zen-Technology fork: CI/CD guide

This fork tracks [openai/openai-cli](https://github.com/openai/openai-cli). Upstream
workflow files are left untouched so syncing never conflicts. Zen-specific
automation lives in `zen-*.yml` files only.

## One-time setup

1. **Enable Actions**: Actions tab, then *I understand my workflows, go ahead and enable them*.
2. **Sync token**: upstream edits `.github/workflows/` regularly, and `GITHUB_TOKEN` can never write workflow files. Create a fine-grained PAT:
   - Resource owner: `Zen-Technology`. Repository access: this repo (or all three forks)
   - Permissions: **Contents: Read and write** and **Workflows: Read and write**
   - Save it as the repository or organization secret **`ZEN_FORK_SYNC_TOKEN`**
3. Run **Actions > Zen fork housekeeping > Run workflow** once.

## Zen workflows

| Workflow | Trigger | Purpose |
| --- | --- | --- |
| `zen-upstream-sync.yml` | daily 05:23 UTC, manual | Calls GitHub's *Sync fork* API: fast-forwards `main`, or merges when the fork has its own commits. Fails with an explanation on conflicts or a missing token. |
| `zen-fork-housekeeping.yml` | daily 05:53 UTC, manual | Disables the upstream-only workflows listed below so they stop creating skipped runs. |

## What upstream workflows do in this fork

| Workflow | In the fork |
| --- | --- |
| `ci.yml` | **Runs** on PRs and pushes to `main`/`next`: lint, Go tests (Linux/macOS/Windows), shell compatibility, artifacts, error audit |
| `codeql.yml` + `codeql-upload.yml` | **Runs**: CodeQL for Go, JS/TS, Python, Actions, with results in Security > Code scanning |
| `help-compatibility.yml` | **Runs** on PRs: help/terminal compatibility matrix (macOS Intel, Windows x64/ARM, WSL) |
| `tokenizer-stress.yml` | **Runs** daily: Windows tokenizer stress test. Disable it in the Actions tab if you do not change the tokenizer. |
| `create-releases.yml` | Disabled by housekeeping (upstream release bot, needs OpenAI's GitHub App) |
| `update-openai-go.yml` | Disabled by housekeeping (hourly upstream SDK bump bot) |
| `dispatch-publish-release.yml`, `publish-release.yml` | Disabled by housekeeping (upstream signing, notarisation and publishing) |
| `sdk-cross-link.yml` | Disabled by housekeeping (internal repository only) |

All guarded jobs check `github.repository == 'openai/openai-cli'`, so they would
be skipped even if enabled. No secrets are needed for the workflows that run.

## Verified locally (Go 1.26.10)

- `go mod verify`: all modules verified
- `go build ./...`, `go vet ./...`, `go test ./... -run '^$'`: clean
- `go test ./internal/... ./cmd/...`: 45 packages ok. `internal/updatenotice` `TestReleaseDialerFallsBackBeforeDeadline` needs an IPv6 loopback, which the verification sandbox lacks. GitHub runners have one.

## Syncing by hand

```sh
git remote add upstream https://github.com/openai/openai-cli.git
git fetch upstream
git merge upstream/main
git push origin main
```
