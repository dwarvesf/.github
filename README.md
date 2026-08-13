## Overview

Github [introduced](https://help.github.com/en/articles/about-issue-and-pull-request-templates) Issue & PR Template in 2016. This repo contains the Issue Template and Pull Request Template that we use at Dwarves Foundation. To make things easier, we have adopted them and we think they are great to help the team improve the productivity.

- [Issue Template](/ISSUE_TEMPLATE.md)
- [Pull Request Template](/PULL_REQUEST_TEMPLATE.md)

## Contribution

We love pull requests. If you have something you want to add or remove, please open a new pull request. Please leave all PRs open for at least a week to get feedback from everyone.

## Reusable workflows

Workflows any dwarvesf repo can call from its own `.github/workflows/`:

- `no-agent-attribution.yml` — blocks commits and PR bodies with coding-agent
  attribution.
- `mini-ci.yml` — runs a repo's test gate on the self-hosted Mac Mini runner
  instead of GitHub-hosted runners. Trusted CI only.

To adopt `mini-ci`, add one job:
`uses: dwarvesf/.github/.github/workflows/mini-ci.yml@master` with a
`test_command` input. The file header lists the inputs and the scope rule.