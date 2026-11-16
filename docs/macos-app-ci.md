# macOS app CI standard (moved)

The standard now lives in consolelabs/.github, because the macOS apps move to the consolelabs org: [consolelabs/.github docs/macos-app-ci.md](https://github.com/consolelabs/.github/blob/main/docs/macos-app-ci.md).

This repo keeps a mirror of the reusable workflow, `.github/workflows/macos-app-ci.yml`, for dwarvesf repos that have not moved yet (blurred today). consolelabs repos may only call consolelabs-owned workflows, so the consolelabs copy is the source of truth; change both together.

In short: every test job runs on `[self-hosted, macos-ui]` and every Release build on `[self-hosted, macos-build]`, both inside the macOS VM on the Mac Mini. No macOS app CI job runs on the Mini host.
