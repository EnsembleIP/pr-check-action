# PR Check GitHub Action

This repo provides a reusable **GitHub Action** that:

- **Validates branch names** for PRs targeting release branches
- **Enforces cherry-pick branch naming** format (configurable via regex)
- **Automatically closes PRs** that don't match the required format
- **Posts helpful comments** explaining the required branch format

The action is implemented as a composite action using `actions/github-script@v7`.

## Usage

Add this to your repository workflow to validate branch names on pull requests targeting release branches:

```yaml
name: Validate PR Branch
on:
  pull_request:
    branches: [ release/* ]

permissions:
  contents: read
  pull-requests: write

jobs:
  validate-branch:
    runs-on: ubuntu-latest
    steps:
      - name: Validate cherry-pick branch format
        uses: ensembleip/pr-check-action@v1
```

## Versioning / pinning

`uses: ensembleip/pr-check-action@vx` requires that this repository has a **Git tag** named `vx` (or a branch named `vx`).

- For initial testing you can use `@main`.
- For production usage, prefer pinning to a commit SHA, or use a major tag like `@v1` that you keep updated to the latest `v1.x.y`.

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `branch-regex` | Regular expression pattern for validating cherry-pick branch names | No | `^bug/QD-\\d+-.+-cherry-pick$` |
| `example-branch` | Example branch name to display in error messages | No | `bug/QD-1234-bug-description-cherry-pick` |

## How It Works

1. **Checks if the PR targets a release branch** (branches starting with `release/`)
2. **Validates the head branch** against the configured regex pattern
3. If validation **fails**:
   - Creates a PR comment explaining the required format
   - Automatically closes the PR
   - Fails the GitHub Action check
4. If validation **passes**:
   - The check passes silently

## Customization

### Custom Branch Regex

You can customize the branch naming pattern by providing your own regex:

```yaml
- name: Validate cherry-pick branch format
  uses: ensembleip/pr-check-action@v1
  with:
    branch-regex: "^hotfix/.*-cp$"
    example-branch: "hotfix/my-fix-cp"
```

### Custom Example Branch

Provide a custom example branch name for clearer error messages:

```yaml
- name: Validate cherry-pick branch format
  uses: ensembleip/pr-check-action@v1
  with:
    example-branch: "bugfix/PROJ-5678-description-cherry-pick"
```

## Default Behavior

By default, the action:
- Only validates PRs targeting branches starting with `release/`
- Expects branch names matching: `bug/QD-<number>-<description>-cherry-pick`
- Shows example: `bug/QD-1234-bug-description-cherry-pick`

## Example PR Comment

When validation fails, the action posts a comment like this and closes the PR:

```markdown
This PR targets `release/v1.2.3`, so **only cherry-pick PRs are allowed** for release branches.

Your head branch is `bug/my-branch`, which does not match the required format:

- `bug/QD-1234-bug-description-cherry-pick`

Please rename your branch to the correct format and open a new PR. Closing this PR now.
```

## Permissions

The workflow using this action requires the following permissions:

- `contents: read` - to read repository contents
- `pull-requests: write` - to post comments and close pull requests

The action automatically uses `github.token` provided by GitHub Actions, so no token input is required.

## Notes

- The action only runs on pull request events. It will skip gracefully for other event types.
- Only PRs targeting branches starting with `release/` are validated.
- PRs that don't match the required format are automatically closed.
- The action uses the GitHub REST API to manage PRs efficiently.
