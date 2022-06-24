# Release Workflow

This repository uses a system similar to git-flow and has 4 types of branches.

- `main`
- `release-v{X.X}` (Contains all changes for a given release)
- `develop` (Contains reviewed/approved changes, may contain changes that will not go into the same release)
- Work branches dedicated to a specification addition/change.

## Add/Change a specification

- Create a new branch starting from an up-to-date develop.

## Release Process

- Create a `release-v{X.X}` branch starting from an up-to-date `main`.

### Insert a change to a release.

- Merge squash the pull-request modifying/adding a specification in `develop`, if it has not been done earlier.
- Move to the `release-v{X.X}` branch and cherry pick the previous commit created during the merge squash operation.

### Release a release

- Merge squash the `release-v{X.X}` branch pull-request into `main`.
- Pull `main` into `develop`.

### Rebase current work branch to develop

- Move to the given `branch_name`
- `git pull origin branch_name`
- `git rebase origin/develop`

### Publish the Open API for the new release

- Publish the new release on bump.sh by transferring the `open-api.yml` file from the `main` branch to it.