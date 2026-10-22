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
    semver version. E.g. `v1.2.3`
  - Move the main version tag, e.g. `v1`.
  - Push the tags to this repo.
- Update the calling workflow to refer to the tag again:
  `users: n0-computer/workflows/.github/workflow/the_workflow.yaml@v1`
- Merge the calling workflow into its repo's main branch.
