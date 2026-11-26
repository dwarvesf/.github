# macOS app CI standard (moved)

The standard now lives in consolelabs/.github, because the macOS apps move to the consolelabs org: [consolelabs/.github docs/macos-app-ci.md](https://github.com/consolelabs/.github/blob/main/docs/macos-app-ci.md).

This repo keeps a mirror of the reusable workflow, `.github/workflows/macos-app-ci.yml`, for dwarvesf repos that have not moved yet (blurred today). consolelabs repos may only call consolelabs-owned workflows, so the consolelabs copy is the source of truth; change both together.

In short: a test goes where its needs put it, all inside the macOS VM on the Mac Mini. No macOS app CI job runs on the Mini host.

| Tier | Runner | Input |
|---|---|---|
| Logic (no `TEST_HOST`) | `[self-hosted, macos-build]` | `logic_scheme`, runs on every call |
| iOS Simulator (`destination` contains `iOS Simulator`, `test_action: test`) | `[self-hosted, macos-build]` | `screen_scheme` |
| App-hosted and render/UI | `[self-hosted, macos-ui]` | `screen_scheme`; with `ui_paths` set, a PR runs it only when the diff touches a listed path (pushes always run it) |
| Build (`test_action: build-for-testing`, Release build) | `[self-hosted, macos-build]` | `test_action`, `release_scheme` |

`test_scheme` still works as the old name of `screen_scheme`. Cancel superseded PR runs with `concurrency` in the caller, not in the reusable workflow. The full table, the `ui_paths` rules and a caller example are in the consolelabs doc linked above.
