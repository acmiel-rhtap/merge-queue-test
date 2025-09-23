# merge-queue-test

Testing merge queue behavior, particularly with Konflux.

## Great

- We go from needing N PR builds + 1 push build to needing just 1 MQ build
  (except for intermittent failures - more details in the Bad section).

## Good

- Snapshots built by merge queue pipelines are release-able as long as the pipeline
  doesn't set the `quay.expires-after` label.
- Artifacts built in the merge queue have correct git metadata. I.e. the commit
  sha is the same as the final commit in main (on the Snapshot, in the revision
  annotation on the container, in git metadata baked directly into binaries, etc.)
- By enabling PipelineRun cancellation and disabling the "Require all queue entries
  to pass required checks" setting, a user can theoretically achieve *less than
  one build per PR*[^1]. The effectiveness of this would scale with the rate of
  PR merges in the repo. (This doesn't currently work in RH Konflux, because
  cancellation is enabled only for PRs, not for pushes).

## Neutral

- If the PR author and/or reviewer want to test the Konflux pipeline before clicking
  the "Merge when ready" button, they have to comment `/test <pipelineRun name>`.
  On the bright side, it's nice that this works with no further configuration.
- Merge queues are not available for personal repos, so this feature is only usable
  for organizations. But that seems like a minor inconvenience at most.

## Bad

- If the build passes but then something else fails in the merge queue (e.g. Conforma
  checks or the build of a different component in a monorepo), then we have multiple
  images pushed and multiple plausible-looking Snapshots. The extra images will not
  expire[^2]. The user needs to be somewhat cautious when manually releasing Snapshots.
  - For the same reason, auto-releasing wouldn't be a good idea as-is. Integration
    Service would have to do one additional check before auto-releasing a Snapshot:
    "Is the revision really present in the target branch?"
- And, for a similar reason, mono-repos get hit harder by unreliable builds.
  Normally, if a PR triggers the builds of N components and some of those builds
  fail, the user only needs to re-trigger the failing builds. Same goes for post-merge
  builds. In a merge queue, however, all N builds need to pass together. If one
  fails, they all need to run again.

## Blocker

- To make the merge queue wait until the build pipeline succeeds, the user has to
  set the pipeline as a required status check. This by itself is an inconvenience,
  but what makes it really bad is two missing GitHub features:
  - [Require different set of checks to enter merge queue vs. in the merge queue](https://github.com/orgs/community/discussions/103114)
  - [Required checks should only be required if they run](https://github.com/orgs/community/discussions/13690)
- The combination of the missing features has these effects:
  - The pipeline must run at least once before the PR can enter the merge queue.
  - In a monorepo, if the PR only triggers some of the pipelines, the merge will
    be blocked forever.
- Luckily, this PaC feature (or bugfix?) would solve both of the above:
  [Mark skipped PipelineRuns as successful on GitHub](https://github.com/openshift-pipelines/pipelines-as-code/issues/1746)

[^1]: Let's say we merge 2 PRs in quick succession. PR 1 starts its pipeline. PR 2 gets
  merged, starts pipeline 2, this cancels pipeline 1. Pipeline 2 succeeds, both PRs
  get merged.
[^2]: This would be solvable with a different expiration mechanism. For example:
  instead of relying on the Quay-specific label, we could extend the existing cronjob
  (which garbage-collects orphaned SBOMs, source containers etc. to also expire binary
  images built for commits that are not associated with any branch).
