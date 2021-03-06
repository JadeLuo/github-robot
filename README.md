# Angular Github bot

> A Github Bot built with [probot](https://github.com/probot/probot) to triage issues and PRs 

## Setup

```
# Install dependencies
yarn install

# Run the bot
npm start
```


# Usage
This bot is only available for repositories of the [Angular organization](http://github.com/angular/).
See [docs/deploy.md](docs/deploy.md) if you would like to run your own instance.

### Adding the bot:
1. Create `.github/angular-robot.yml` based on the following template
2. [Configure the Github App](https://github.com/apps/wheatley)
3. It will start scanning for opened issues and pull requests to monitor

A `.github/angular-robot.yml` file is required to enable the plugin. The file can be empty, or it can override any of these default settings:
```yaml
# Configuration for angular-robot

# options for the merge plugin
merge:
  # the status will be added to your pull requests
  status:
    # set to true to disable
    disabled: false
    # the name of the status
    context: "ci/angular: merge status"
    # text to show when all checks pass
    successText: "All checks passed!"
    # text to show when some checks are failing
    failureText: "The following checks are failing:"

  # comment that will be added to a PR when there is a conflict, leave empty or set to false to disable
  mergeConflictComment: "Hello? Don't want to hassle you. Sure you're busy. But--this PR has some conflicts that you probably ought to resolve.
\nThat is... if you want it to be merged someday..."

  # label to monitor, it will be removed if one of the checks doesn't pass
  mergeLabel: merge
  # label to override the checks, if present then the merge label will not be removed even if a check fails, leave empty or set to false to disable
  overrideLabel: override

  # list of checks that will determine if the merge label can be added
  checks:
    # the PR shouldn't have a conflict with the base branch
    noConflict: true
    # list of labels that a PR needs to have, checked with a regexp (e.g. "PR target:" will work for the label "PR target: master")
    requiredLabels:
      - "PR target:"
      - "cla: yes"

    # list of labels that a PR shouldn't have, checked after the required labels with a regexp
    forbiddenLabels:
      - "PR target: TBD"
      - "cla: no"

    # list of PR statuses that need to be successful
    requiredStatuses:
      - "continuous-integration/travis-ci/pr"
      - "code-review/pullapprove"
      - "ci/circleci: build"
      - "ci/circleci: lint"

  # the comment that will be added when the merge label is removed, leave empty or set to false to disable
  # {{MERGE_LABEL}} will be replaced by the value of the mergeLabel option
  # {{OVERRIDE_LABEL}} will be replaced by the value of the overrideLabel option
  # {{PLACEHOLDER}} will be replaced by the list of failing checks
  mergeRemovedComment: "I don't like to brag, but I just saved you from a horrible, slow and painful death by removing the `{{MERGE_LABEL}}` label. Probably. Maybe...
\nThe following checks are failing:
\n{{PLACEHOLDER}}
\n
\nBut if you think that you know better than me, then please, go ahead and add the `{{OVERRIDE_LABEL}}` label. You'll be free to do whatever you want. Don't say that I didn't warn you."
```

# Plugins
The bot is designed to run multiple plugins.

### Merge plugin:
The merge plugin will monitor pull requests to check if they are mergeable. It will:
- check for conflicts with the base branch and add a comment when it happens
- check for required labels using regexps
- check for forbidden labels using regexps
- check that required statuses are successful
- add a status that is successful when all the checks pass
- remove the `merge` label if any of the checks is failing and add a comment to list the reasons

You can override this behavior by adding the `override` label (the name of the label is configurable), in which case the
bot will do nothing when you add the `merge` label.

When you install the bot on a new repository, it will start scanning for opened PRs and monitor them.

It will **not**:
- remove existing merge labels
- add a comment for conflicts until you push a new commit to the base branch
- add the new merge status until the PR is synchronized (new commit pushed), labeled, unlabeled, or receives another status update
