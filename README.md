# workflows

Collection of reusable workflows for n0 repos.

## Using a shared workflow:

To re-use a workflow use e.g.:

```yaml
jobs:
  min-crates:
    name: Min Crates
    uses: "n0-computer/workflows/.github/workflows/minimal-crates.yaml@main"
```

### Editing a shared workflow

- Start editing a shared workflow in a branch, e.g. `username/branch_name`.
- Use the workflow from the repo where the changes are needed, in a
  branch as well and make a draft PR.
  - Call the workflow by branch name, e.g.:
    `uses: n0-computer/workflows/.github/workflow/the_workflow.yaml@username/branch_name`.
  - This will fail the zizimor and pinact checks on this PR, fine for
    now.
  - Edit the shared workflow and caller until all other CI checks
    pass.
  - Merge the shared workflow branch to `main` in this repo.
  - Tag the new commit on `main` in this repo, use the next suitable
    semver version.
  - Push the tag to this repo, e.g. `v1.2.3`.
- Update the calling workflow to refer to the tag by hash, and use the
  tag name in a comment. e.g.:
  `users: n0-computer/workflows/.github/workflow/the_workflow.yaml@abc12..89def # v1.2.3`
- Merge the calling workflow into its repo's main branch.
- All other repos using the same shared workflow will get updated
  automatically by pinact at some point.
