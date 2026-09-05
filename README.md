## Overview

Github [introduced](https://help.github.com/en/articles/about-issue-and-pull-request-templates) Issue & PR Template in 2016. This repo contains the Issue Template and Pull Request Template that we use at Dwarves Foundation. To make things easier, we have adopted them and we think they are great to help the team improve the productivity.

- [Issue Template](/ISSUE_TEMPLATE.md)
- [Pull Request Template](/PULL_REQUEST_TEMPLATE.md)

## Contribution

We love pull requests. If you have something you want to add or remove, please open a new pull request. Please leave all PRs open for at least a week to get feedback from everyone.

## Reusable workflows

Workflows any dwarvesf repo can call from its own `.github/workflows/`:

- `no-agent-attribution.yml`: blocks commits and PR bodies with coding-agent
  attribution.
- `mini-ci.yml`: runs a repo's test gate on the self-hosted Mac Mini runner
  instead of GitHub-hosted runners. Trusted CI only.

To adopt `mini-ci`, add one job:
`uses: dwarvesf/.github/.github/workflows/mini-ci.yml@master` with a
`test_command` input. The file header lists the inputs and the scope rule.

## CI runners: private repos run on the Mac Mini

GitHub-hosted runners cost money on private repos, so the org's Actions budget is **$0** (stop-usage on). The rule:

| Repo | Runner | Why |
|---|---|---|
| private | `runs-on: [self-hosted, pr-shared]` (PR checks, no secrets) or a trusted per-repo container for deploys | zero GitHub spend |
| public | `ubuntu-latest` is fine | GitHub-hosted minutes are free on public repos |

**New private repo checklist**

1. New workflow: pick the org template "Dwarves CI (self-hosted, private repos)" (Actions tab, "New workflow"), or set `runs-on: [self-hosted, pr-shared]` yourself. Every private repo can already target the pool; no ops ticket needed.
2. Never write `ubuntu-latest` / `macos-latest` / `windows-latest` in a private repo. With the $0 budget the job dies once the included minutes are gone, and it costs money before that.
3. The pool image is Debian slim: bash, git, jq, python3, curl, tar, unzip. `setup-node` / `pnpm/action-setup` bring node and pnpm. No `gh`; use curl against the REST API. macOS jobs: `runs-on: [self-hosted, macOS]`.
4. Deploys with secrets do not run on the pool. Ask ops for a trusted container (`gh-runner-container` in ops-toolkit).

A weekly scan (`hosted-drift` in ops-toolkit `tools/gh-runner-pr-pool`) lists private repos still on hosted labels.
