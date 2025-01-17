# CI/CD according to Truss

## What is this? Is this for me?

You just started on a brand-new project. It's day 1, and everything is
greenfield; there is no existing software, let alone anything to continuously
integrate or deploy. You are tasked to build CI/CD for the project. How do you
design toward your requirements? How do you set yourself up for success when
you don't even know what your requirements are going to be?

Or maybe you inherited a CI/CD system that the client already has. It's in bad
shape, and you are tasked to get it into good shape. But what is "good shape"
for CI/CD? What needs to be handled first?

If any of that feels familiar, this document is for you.

### This document is *not* prescriptive

Working with constraints is a fact of life that we accept to deliver value to
our clients. We expect you will make strategic compromises in engineering
architecture where it is necessary or expedient, rather than unthinkingly
conforming to rules handed down from outside the project.

Truss leadership knows that there is no one better equipped and informed to
make those compromises than the people working with the project's requirements
day to day, and we trust you to make those decisions.

Also, good documentation is hard. Keeping it up to date is even harder. Some of
what you are about to read is out of date. Use your judgment, absorb what is
useful, and disregard what is not.

### Treat these points as a guiding star

Follow the star if you are lost and need direction. Ask questions. If you know
better than what's written here, update it to help those who will come after
you.

## Platform

- DO: Use [Github Actions] (GHA).

*Especially* on FedGov projects, due to the FedRAMP status (or lack thereof)
for the alternatives.

GHA has good documentation, and a rich ecosystem for shared Actions. Deployment
orchestration is solid. Support for self-hosted runners is there.
Parallelization is good.

Using the same service for source control and CI/CD makes for a clearer access
control story.

- DON'T: Use [CircleCI] on Federal government projects.

CircleCI is not FedRAMP'd. That means you must use their self-hosted offering,
which always lags far behind on features and documentation.

We have had good experiences with CircleCI's cloud service. If a commercial
client wants it, then we should consider it.

However, using GHA means you have a smaller stack, so lean that way anyhow.

- DON'T: Use [Bitbucket Pipelines], at all.

Documentation is wholly inadequate.

It does not have feature parity with GHA or CircleCI, by a long shot.

The API is poorly implemented.

## Run time

CI/CD systems are classic accumulators of tech debt. Maintaining the
reliability and run time of these processes requires setting boundaries up
front and then doggedly sticking to them.

- DO: Maintain focus on how the run time for build and deploy workflows impacts
  developer effectiveness iterating on the app.

If a workflow increases the overhead on developers trying to iterate on the
software, it should be *removed* until it can be fixed.

This often shows up as: "We don't know how to make it faster, but that's on our
roadmap." No. Make the time now, or get rid of the slow thing until you can do
it right.

- DON'T: Lose sight of how long build and deploy workflows *will* inhibit fixes
  during an incident. Slow CI/CD is a security and availability risk to your
  project!

See above.

- DON'T: Tolerate flaky tests in CI.

Flaky tests in CI are even worse than a slow build because they add routine
toil for developers monitoring and re-running their builds.

"That test is flaky, just re-run it!" No. Remove the flaky test, or configure
it to run locally only. If it can be re-engineered later to fix the flakiness,
*then* it can be restored to CI.

- DO: Get buy-in from product management on these points early.

- DO: Norm early and often on what is an acceptable duration for builds and
  deployments, and then *stick to the norm*.

- DO: Parallelize as much as you can.

Include parallelization as part of the acceptance criteria for planned
expansions to CI/CD workflows.

Routinely revisit the overall architecture to look for what else could be
parallelized.

- DON'T: Gate deploy steps on any kind of human intervention.

A merge to the main branch should be the final human action required to deploy
to production.

If you need blue/green deploys or such to avoid unplanned downtime with this
model, then you need blue/green deploys.

- DON'T: Accept the introduction of manual testing.

Manual tests are time consuming and tedious. They are a drag on the development
cycle like flaky or slow tests, except far worse.

It is not temporary, no matter what anyone says. Our experience shows clearly
that once there are any manual tests, you are on a slippery slope.

## Runners

- DO: Use self-hosted runners if CD can deploy into production.

This avoids complex credential management solutions.

- DO: Have separate runners following the principle of least privilege.

For example, a runner that only runs pre-commit doesn't need to have the same
access as a runner that deploys to a test environment, which doesn't need the
same access as a runner that deploys to production.

If you are dealing with an ATO process, your security officer will appreciate
this. If you don't have one, it's still best practice.

## Builds

- DO: Pin the version of everything using deterministic, calculated tags, like
  the git commit digest, a unix timestamp, or a combination.[^1]

- DON'T: Tag anything with \`:latest\`, \`:staging\` or anything of such.[^2]

- DO: Use distroless base images in docker, and build everything on top of
  those.

## Artifact storage

- DO: Get money for [artifact storage] into the budget ASAP.

Budget amendments are hard, especially if they require contract modifications.
So get the paperwork in line well before you want artifact storage (and you'll
want artifact storage early).

Artifact storage is extremely cheap compared to the engineering time you will
bill the client later trying to work around not having it.

- DO: Set up artifact storage for caching builds.

Whether it's uploading a zip file to S3 or pushing a built image to ECR,
artifact caching and storage enables later optimizations for automated
vulnerability scanning, build promotion, sharing artifacts (e.g. docker layers)
between builds to accelerate build and deploy times, getting visibility into
failed builds, and more.

- DON'T: Fragment your artifact storage repositories.

For example, you need *one* ECR repo shared across AWS accounts.
    <!--  TODO: Explain why. -->

## Alerting

- DO: Alert on build failures from the main branch.

A broken main branch is a fire drill.

- DO: Control [alert fatigue].

- DON'T: Alert on successful builds.

- DON'T: Put alerts in their own Slack channel.

If there are so many alerts from CI/CD that you feel tempted to put them in a
separate channel, then you have too many alerts.

## Other

- DO: Apply the [steel cable] approach to building CI/CD.

[alert fatigue]: https://en.wikipedia.org/wiki/Alarm_fatigue
[artifact storage]: https://docs.github.com/en/actions/using-workflows/storing-workflow-data-as-artifacts
[bitbucket pipelines]: https://bitbucket.org/product/features/pipelines
[circleci]: https://circleci.com
[github actions]: https://github.com/features/actions
[steel cable]: https://playbook.truss.works/docs/01-how-we-execute/06-steel-cable/
[^1]: https://vsupalov.com/docker-better-image-tags/
[^2]: https://vsupalov.com/docker-latest-tag/
