# merge-queue-test

Testing merge queue behavior, particularly with Konflux.

## Findings

Personal repositories do not have merge queues. That's more of an inconvenience
than a blocker.

A user wanting to take advantage of merge queues would have to:

- Enable merge queues
- TBD: use a specific merge method?
- If their build takes long, increase the `Status check timeout`
- Set up required status checks
  - Add each merge queue pipeline as required, probably
  - Also add Conforma checks as required?
  - Set `Do not require status checks on creation`, otherwise the PR cannot even
    enter the merge queue.
    - ❌ nope, this does not work :(
    - People have been requesting this feature for more than a year, but GH doesn't
      have it. <https://github.com/orgs/community/discussions/103114>
- Set up crazy workaround to avoid the limitation where github doesn't allow having
  a different set of required checks to *enter* the merge queue vs. *in* the merge queue.
  - All checks have to run twice
  - On PR, the pipeline can just skip everything
  - On comment or merge-queue entry, the pipeline builds as usual
  - ITS checks fail on PR, because the pipeline doesn't return an `IMAGE_URL`. Sad.

Merge queue on GH has the `Require all queue entries to pass required checks` setting.

> When this setting is disabled, only the commit at the head of the merge group,
> i.e. the commit containing changes from all of the PRs in the group, must pass
> its required checks to merge.

Build just once when N PRs get merged at the same time? Could be interesting.

Fascinatingly, the Snapshot created for a merge queue pipeline is annotated with
the correct commit sha. At least if the merge method is via merge commit. See
`konflux-resources/interesting-snapshots/merged-via-merge-commit.yaml`.

^^ And that works for rebase as the merge method as well:

- `konflux-resources/interesting-snapshots/merged-via-rebase-1.yaml`
- `konflux-resources/interesting-snapshots/merged-via-rebase-2.yaml`

## Biggest blockers

For monorepos where the pipelineRuns are conditional, the required checks will never
start and the merge will always be blocked.

- Missing GitHub feature: <https://github.com/orgs/community/discussions/13690>
- Or missing PaC feature: <https://github.com/openshift-pipelines/pipelines-as-code/issues/1746>

The above also solves the "status checks required just to enter merge queue" problem.
