# HawkScan CI/CD Integrations

Live integration tests for various CI/CD platforms.

## Branch Conventions
Pushes and PRs to the `main` branch should kick off builds for every CI/CD integration defined in this repository. To reduce noise, we attempt to limit CI/CD triggers to main, and branches following a naming convention.

For instance, branches that match the wildcard expression `github/*` should only trigger Github Actions builds. And branches named `concourse/*` should only trigger Concourse builds.

## Active pipelines

The following pipelines are defined:
* GitHub Actions
* Travis CI

These are pending:
* *Armory Spinnaker*
* *AWS CodeBuild*
* *Azure Pipeline*
* *Circle CI*
* *Concourse*
* *GitLab*
* *Google Cloud Build*
* *Jenkins*
