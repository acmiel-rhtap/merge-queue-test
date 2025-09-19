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
    enter the merge queue

Merge queue on GH has the `Require all queue entries to pass required checks` setting.

> When this setting is disabled, only the commit at the head of the merge group,
> i.e. the commit containing changes from all of the PRs in the group, must pass
> its required checks to merge.

Build just once when N PRs get merged at the same time? Could be interesting.

Fascinatingly, the Snapshot created for a merge queue pipeline is annotated with
the correct commit sha. At least if the merge method is via merge commit. See
`konflux-resources/interesting-snapshots/merged-via-merge-commit.yaml`.
