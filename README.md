# workflows

Reusable GitHub Actions workflows for n0 repos.

Call them from a job, pinned to a release commit with the tag in a comment:

```yaml
jobs:
  min-crates:
    name: Min Crates
    uses: n0-computer/workflows/.github/workflows/minimal-crates.yaml@<sha> # v1.2.0
```

Notes for callers:

- Workflow-level `env:` in the caller does not reach a called workflow. Pass
  what a workflow needs as inputs.
- A called workflow can only narrow the permissions the caller grants. The
  caller's workflow should default to `contents: read` and grant more on the
  calling job where the table below says so.
- Checks report as `<calling job> / <called job>`. Update branch protection's
  required checks when moving a job here.
- `runs-on` inputs take runner labels as JSON, e.g. `'"ubuntu-latest"'` or
  `'["self-hosted","linux","X64"]'`.

## Workflows

| Workflow | Does | Inputs | Caller grants |
|---|---|---|---|
| `lint.yaml` | zizmor (all findings gate) and `pinact --check`, both installed by pinned hash. Reads the caller's `zizmor.yml` and `.pinact.yaml`. | none | |
| `cargo-deny.yaml` | `cargo deny check -Dwarnings` against the caller's `deny.toml`. | `arguments` | |
| `commit-check.yaml` | PR title is a conventional commit. Call from a `pull_request` workflow. | none | |
| `fmt.yaml` | `cargo make format-check`, then asserts `Cargo.lock` didn't move. | `toolchain` | |
| `codespell.yaml` | codespell, pinned version. | `ignore-words-list`, `skip` | |
| `clippy.yaml` | clippy with all, no default and default features. | `runs-on`, `toolchain`, `all-features-args`, `no-default-features-args`, `default-features-args` | |
| `msrv.yaml` | `cargo +<msrv> check --locked`. | `msrv` (required), `args`, `runs-on` | |
| `docs.yaml` | `cargo docs-rs` per package, or `cargo doc` on the workspace. | `toolchain`, `docs-rs-packages`, `rustdocflags`, `runs-on` | |
| `docs-preview.yaml` | Publishes PR docs to `generated-docs-preview` and comments the link. | `crate-dir` (required), `pr-number`, `toolchain`, `rustdocflags` | `contents: write`, `pull-requests: write` |
| `docs-cleanup.yaml` | Prunes old PR previews from `generated-docs-preview`. | `keep` | `contents: write` |
| `minimal-crates.yaml` | `-Z minimal-versions` check on Linux, macOS (ephemeral) and Windows. | `windows-runner` | |
| `notify-tests.yaml` | Posts a test run's result to Discord. Flaky runs list failed tests from the nextest reports. | `tests-result` (required), `flaky-report`, `title`; secret `webhook-url` | |
| `rust-binary-release.yaml` | Builds the binary for linux musl, macOS and Windows and attaches the archives to the GitHub release. | `bin-name` (required), `release-version`, `create-release`, `draft`, `upload-artifacts`, `upload-url`, `ref`, `static-musl`, `binary-artifacts`; secret `build-iroh-services-api-secret` | `contents: write` |

`sccache-probe` is a composite action the workflows above use.

## Examples

Lint on workflow changes:

```yaml
on:
  pull_request:
    paths: ['.github/**', '.pinact.yaml', 'zizmor.yml']
  push:
    branches: [main]
    paths: ['.github/**', '.pinact.yaml', 'zizmor.yml']

permissions:
  contents: read

jobs:
  lint:
    uses: n0-computer/workflows/.github/workflows/lint.yaml@<sha> # v1.2.0
```

Beta tests with a notification. The caller keeps its own tests job, a
reusable workflow can't be handed one to call:

```yaml
jobs:
  tests:
    uses: $/.github/workflows/tests.yaml
    with:
      rust-version: beta
  notify:
    needs: tests
    if: always()
    uses: n0-computer/workflows/.github/workflows/notify-tests.yaml@<sha> # v1.2.0
    with:
      tests-result: ${{ needs.tests.result }}
      title: Beta tests
    secrets:
      webhook-url: ${{ secrets.DISCORD_N0_GITHUB_CHANNEL_WEBHOOK_URL }}
```

Binary release (see sendme, dumbpipe, pigeons):

```yaml
jobs:
  release:
    permissions:
      contents: write
    uses: n0-computer/workflows/.github/workflows/rust-binary-release.yaml@<sha> # v1.2.0
    with:
      bin-name: sendme
      release-version: ${{ inputs.release_version }}
      create-release: ${{ github.event_name == 'push' || inputs.create_release == 'true' }}
      upload-artifacts: ${{ github.event_name == 'push' || inputs.upload_artifacts == 'true' }}
      upload-url: ${{ inputs.upload_url }}
```

## Editing a shared workflow

- Edit the shared workflow on a branch here, e.g. `username/branch_name`.
- Use it from the repo that needs the change, on a branch with a draft PR,
  calling it by branch name:
  `uses: n0-computer/workflows/.github/workflows/the_workflow.yaml@username/branch_name`.
  The caller's pinact check fails on the unpinned ref, fine for now.
- Iterate on the shared workflow and the caller until the other checks pass.
- Merge the branch to `main` here and tag the commit with the next semver
  version, e.g. `v1.2.3`.
- Point the caller at the tag's commit, with the tag in a comment:
  `uses: n0-computer/workflows/.github/workflows/the_workflow.yaml@<sha> # v1.2.3`,
  and merge it.
- Dependabot bumps the other callers. Their `.pinact.yaml` should exempt
  n0-owned refs from the cooldown (`min_age: 0` for
  `ActionRepoOwner == "n0-computer"`).

This repo lints itself with `lint.yaml`. Keep its actions pinned to SHAs and
its scripts free of template expressions, every caller inherits them.
