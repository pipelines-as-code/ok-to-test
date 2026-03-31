# ok-to-test

Gate `pull_request_target` CI runs for external contributors. Trusted users pass automatically; external contributors need a maintainer to apply the `ok-to-test` label.

## How it works

The action checks the PR author against these conditions in order:

1. **Trusted bots** -- configurable list (default: `dependabot[bot]`, `renovate[bot]`)
2. **Org/team membership** -- checks team membership first (if configured), then org membership
3. **Repository permission** -- checks for `write` or `admin` collaborator access
4. **`ok-to-test` label** -- a maintainer added the label to approve the external PR

If any condition passes, the step succeeds and CI proceeds. If all fail, the step fails and blocks CI.

When the `ok-to-test` label triggers a run, the label is **removed immediately** so that subsequent pushes by the external contributor require re-approval.

## Requirements

This action only works with `pull_request_target` events.

## Usage

```yaml
on:
  pull_request_target:
    types: [opened, reopened, synchronize, labeled]

jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: pipelines-as-code/ok-to-test@v1

      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}

      # ... your CI steps
```

## Inputs

| Input | Description | Default |
|-------|-------------|---------|
| `github-token` | GitHub token with `repo` and `org:read` scopes | `${{ github.token }}` |
| `trusted-bots` | Comma-separated trusted bot usernames | `dependabot[bot],renovate[bot]` |
| `team-slugs` | Comma-separated org team slugs to check | (empty) |
| `required-permission` | Minimum repo permission: `write` or `admin` | `write` |

## Examples

### With team membership check

```yaml
- uses: pipelines-as-code/ok-to-test@v1
  with:
    team-slugs: "maintainers,contributors"
```

### Admin-only access

```yaml
- uses: pipelines-as-code/ok-to-test@v1
  with:
    required-permission: "admin"
```

### Custom trusted bots

```yaml
- uses: pipelines-as-code/ok-to-test@v1
  with:
    trusted-bots: "dependabot[bot],renovate[bot],my-custom-bot"
```

### With a custom token

```yaml
- uses: pipelines-as-code/ok-to-test@v1
  with:
    github-token: ${{ secrets.ORG_READ_TOKEN }}
    team-slugs: "my-team"
```

A token with `org:read` scope is needed if your org has private membership and you want the org/team membership checks to work for all members.

## Security considerations

- This action is designed for `pull_request_target` workflows where CI runs on the base branch but is triggered by external PRs.
- The `ok-to-test` label can only be added by users with write access to the repo, so it is inherently gated.
- The label is removed after each approved run, preventing a single approval from covering future (potentially malicious) pushes.
- If you have automation (bots, workflows) that adds labels to PRs, ensure it does not add the `ok-to-test` label unintentionally, as this will trigger CI for external contributors. If this is intentional from the automation side, no action is needed.

## License

Apache-2.0
