# GitLab Repo Mirror Pull Sync - Docker Image + GitLab Scheduled Pipeline

Useful Docker image that can be used in combination with a [GitLab Scheduled Pipelines](https://docs.gitlab.com/ee/ci/pipelines/schedules.html) for pulling remote repositories.

Normally, GitLab Community Edition (GitLab CE) _only_ offers you to mirroring repositories via `push` direction. This Docker image with yaml file + scheduler allows you to sync a repository via **`pull` mirror direction**!

Have the "GitLab mirror pull" feature for free, even when using GitLab CE! Use the steps below to get started.

## Docker Image

We are following Docker image in our pipeline: [danger89/repo_mirror_pull](https://hub.docker.com/r/danger89/repo_mirror_pull) ([source](./Dockerfile))

## GitLab Pipeline

[GitLab Scheduled Pipeline](https://docs.gitlab.com/ee/ci/pipelines/schedules.html) yaml file (`.gitlab-ci.yml`). No changes are needed, the variables can be changed in the GitLab Schedule (see next heading).

If the rebase fails the pipeline aborts automatically (so no git push).

```yml
repo_pull_sync:
  image: danger89/repo_mirror_pull:latest
  rules:
    - if: '$CI_PIPELINE_SOURCE == "schedule"'
    - if: $REMOTE_URL
    - if: $REMOTE_BRANCH
    - if: $ACCESS_TOKEN
  before_script:
    - git config --global user.name "${GITLAB_USER_NAME}"
    - git config --global user.email "${GITLAB_USER_EMAIL}"
  script:
    - git checkout $CI_DEFAULT_BRANCH
    - git pull
    - git remote remove upstream || true
    - git remote add upstream $REMOTE_URL
    - git fetch upstream
    - git rebase upstream/$REMOTE_BRANCH
    - git push "https://${GITLAB_USER_LOGIN}:${ACCESS_TOKEN}@${CI_REPOSITORY_URL#*@}" "HEAD:${CI_DEFAULT_BRANCH}"
```

*Note:* If you want to use `git merge` instead of `git rebase` that is up to you. You can change the script above to your needs.

## Create GitLab Schedule

Create Project Access token fist:

* Go to: `Settings -> Access Tokens`. Check 'api' as the scope. Save the secret token for later.

Create a new GitLab Schedule:

* Go to: `CI/CD -> Schedules -> New schedule`. With the following 3 variables:
  * `REMOTE_URL` (example: `https://github.com/project/repo.git`)
  * `REMOTE_BRANCH` (example: `master`)
  * `ACCESS_TOKEN` (see Access Token in the first step! Example: `gplat-234hcand9q289rba89dghqa892agbd89arg2854`)
* Save pipeline schedule
* Press the Play button to trigger the Schedule prematurely (for testing purpose)
