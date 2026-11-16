# macOS app CI standard

Every dwarvesf macOS app follows this. New apps start from the org template "Dwarves macOS app CI" (Actions tab, New workflow), which calls the reusable workflow `.github/workflows/macos-app-ci.yml`. Two goals: a warm PR run finishes in about a minute, and no CI job or test ever opens an app on the Mac Mini's own screen.

## Where each job runs

```
GitHub Actions (routes by runs-on label)
   │                                        │
   ▼ [self-hosted, macOS, ARM64]            ▼ [self-hosted, macos-ui]
Mac Mini host runner                     macos-ui VM on the Mac Mini
  Release build (pushes only)              xcodebuild test (the app opens here)
  never opens an app                       own screen, menu bar, keychain
```

| Job | Runner | Why |
|---|---|---|
| Unit and app-hosted tests | `[self-hosted, macos-ui]` | A test target with `TEST_HOST` launches the real app. In the VM it cannot steal the operator's screen, menu bar, login items or installed copy. |
| Release build | `[self-hosted, macOS, ARM64]` | Compiling opens nothing, so it stays on the host. |
| Agent UI checks (clicks, screenshots, permission sheets) | the same VM, through ops-toolkit `tools/mac-ui-vm` | Real input goes to the VM's own screen only. |
| Public repos | GitHub-hosted `macos-*` is allowed | Minutes are free on public repos; private repos never use them (the org budget is $0). |

## Speed rules

Measured on hacker-bar, hidden and blurred (Nov 2026): the queue wait was a few seconds; nearly all the time was cold builds and runs that did not need to happen.

| Rule | What it fixes (measured) |
|---|---|
| DerivedData and SwiftPM checkouts live on the runner's disk (`~/ci-cache/<owner>_<repo>/`), never in `$RUNNER_TEMP` | hacker-bar rebuilt from scratch every run: 80 s of its 106 s job. blurred keeps DerivedData and drops from 30 s to 9 s when nothing changed. |
| No `actions/cache` on self-hosted runners | The local disk already persists. hidden spends about 10 s per job restoring and saving its cache. |
| Release build only on pushes to the default branch | hacker-bar built Release on every PR and again on the merge push. |
| `paths-ignore` for docs, plus `concurrency` that cancels superseded PR runs (never a default-branch run) | hacker-bar ran 66 times in 24 h, docs-only commits included, on one serial runner. |
| Pure-logic code gets a test target without `TEST_HOST` | An app-hosted test pays about 15 to 20 s of app startup; hacker-bar's tests themselves run in about 2 s. blurred has no test host and is the fastest. |
| `xcodebuild -quiet` on the Release build only, never on tests | `-quiet` hides a failing test's assertion text; the first pilot run showed only test names. |

Each repo's DerivedData and SwiftPM checkouts sit under `~/ci-cache/<owner>_<repo>/` on the runner. Reset a bad cache by deleting that folder; the next run rebuilds cold. The VM disk is fixed size and nothing prunes these folders yet, so check `df -h` in the VM when a new app joins.

## The test VM

The `macos-ui` runners live in the `mac-ui` VM (ops-toolkit `tools/mac-ui-vm`): one runner per org (dwarvesf, consolelabs), Xcode copied from the Mini. The VM screen is 2560x1440, the Mini's point size, because some tests size UI to the screen: at 1920x1200 hacker-bar's menu showed 11 rows instead of 12 and two render tests failed.

Measured on the hacker-bar pilot: the test job took 62 s in the VM on a warm cache, against 106 s for the old job on the Mini host.

## Safety rules

- Test and Debug builds use a bundle id different from the shipping one (for example `.dev`), so a test run never touches the installed app or its login item.
- CI signs ad hoc. Developer ID signing and notarization run on a release machine, never on a runner.
- Test hosts are stopped by their exact DerivedData path, never by a broad `pkill` pattern: a broad pattern once killed a parallel CI run's test host.
- `timeout-minutes: 20` on every job.
- Callers use `@master`, like `mini-ci.yml`. A change to the reusable workflow reaches every app at once, so it goes through review and a pilot run first.

## New app checklist

1. Add the workflow from the org template and set `project`, `test_scheme`, `release_scheme`.
2. Give the test and Debug configurations their own bundle id.
3. Put pure-logic tests in a target without a test host.
4. Open the first PR and check that the test job ran on `macos-ui` and finished in about a minute on its second run.
