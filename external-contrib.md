# Options for Outside Contributor Secret Usage in GitHub PRs

## Skip Jobs Requiring Secrets

- The simplest approach to the problem.
- Just skip over or stub out jobs if the required secrets are not present.
- In practice, this would mean skipping Ironbank tests on PRs (and Chainguard in the future).
- Tests could still be run pre-release.
- Primary flavor specific changes tend to be Renovate PRs which do not run on a fork
- If we want we could still add a slash command or another option to trigger specific flavor tests, but not require them/link them to the PR directly

## Use Slash Commands as Trigger

- Ironbank workflows would be triggered by `issue_comment` and `repository_dispatch`.
- This would provide a "ChatOps" like experience where a maintainer comments `/test` or similar to trigger checks requiring secrets.
- Part of the triggered workflow would checkout the PRs code.
- Secrets would work, enabling the full test suite to run.
- Upstream and other workflows that don't require secrets could still run directly on `pull_request`
- [slash-command-dispatch](https://github.com/peter-evans/slash-command-dispatch) is already used in some places in the company.
- Downsides:
  - Workflows must be on `main` before they are used since the slash command will trigger a `main` workflow.
  - Extra process for triggering workflows (although this could reduce runner minute usage for renovate PRs)
  - Decent amount of complexity and/or external action usage to make this process seamless (i.e. have to update the PR pipeline status "manually")
  - May be unable to keep everything in the same concurrency group if in a separately triggered workflow

## Use workflow run as trigger

- Ironbank workflows would be triggered by `workflow_run` in response to `pull_request` workflows
- This would be a fully automated process
- Part of the triggered workflow would checkout the PRs code.
- Secrets would work, enabling the full test suite to run.
- Upstream and other workflows that don't require secrets could still run directly on `pull_request`
- Downsides:
  - Workflows must be on `main` before they are used since the slash command will trigger a `main` workflow?
  - Decent amount of complexity and/or external action usage to make this process seamless (i.e. have to update the PR pipeline status "manually")
  - May be unable to keep everything in the same concurrency group if in a separately triggered workflow

## Use PR Target as Trigger

- Ironbank workflows would be triggered by `pull_request_target`.
- As part of the workflow, we would checkout the PRs code.
- This is inherently dangerous ([learn more](https://securitylab.github.com/research/github-actions-preventing-pwn-requests/)) and not "protected" by the workflow approval org settings ([see documentation](https://docs.github.com/en/enterprise-cloud@latest/actions/managing-workflow-runs/approving-workflow-runs-from-public-forks#about-workflow-runs-from-public-forks)).
- We could run the job under a GitHub environment with required approvers (and filter based on user identity to auto-run unicorn PRs but require approval on true external users?).
- Downsides:
  - Workflows/workflow changes must be on `main` before they are used since the `pull_request_target` will use the `main` workflows
  - "Deploying" to an environment in a PR is very noisy and could be confusing as no deployment is actually happening

## Create a staging branch for external changes

- PRs from external contributors would be into a `staging` type branch (NOT main)
- These PRs would only run upstream checks
- Merges from `staging` to `main` would run additional check where secrets would be accessible
- Downsides:
  - Added complexity for external contributions
  - Little benefit over the "skip job" option - PRs still have to be merged somewhere and follow-on fixes would likely be handled internally prior to release

