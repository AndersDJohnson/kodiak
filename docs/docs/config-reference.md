---
id: config-reference
title: Configuration Reference
---

Kodiak's configuration file is a TOML file and should be placed at `.kodiak.toml` (repository root) or `.github/.kodiak.toml`.

For Kodiak to run on a pull request:

1. Kodiak must be installed on the repository.
2. A configuration file must exist in the repository.
3. GitHub branch protection must exist on the target branch.

## configuration fields

### `version`

- **type:** `number`
- **required:** `true`

`1` is the only valid setting for this field.

### `merge.automerge_label`

- **type:** `string` or `string[]`
- **default:** `"automerge"`

Label to enable Kodiak to merge a PR.

By default, Kodiak will only act on PRs that have this label. You can disable this requirement via `merge.require_automerge_label`.

Kodiak can only merge pull requests passing your GitHub branch protection rules.

```toml
merge.automerge_label = "🚀 merge it!"
```

If multiple labels are specified in an array, any of the specified labels will trigger merge.

```toml
merge.automerge_label = ["🚢 it!", "merge it!"]
```

### `merge.require_automerge_label`

- **type:** `boolean`
- **default:** `true`

Require that the automerge label (`merge.automerge_label`) be set for Kodiak to merge a PR.

When disabled, Kodiak will immediately attempt to merge any PR that passes all GitHub branch protection requirements.

### `merge.automerge_dependencies.versions`

- **type:** `string[]`
- **options:** `"major"`, `"minor"`, `"patch"`
- **default:** `[]`

Kodiak will only automerge version upgrade types in this list. The author of the pull request must also be listed in [`merge.automerge_dependencies.usernames`](#mergeautomerge_dependenciesusernames).

Like [`merge.automerge_label`](#mergeautomerge_label), your pull request can only be merged if it passes your GitHub branch protection rules.

See ["Configuring automerge by upgrade type"](recipes.md##configuring-automerge-by-upgrade-type) for a full example.

```toml
# .kodiak.toml
[merge.automerge_dependencies]
# only auto merge "minor" and "patch" version upgrades.
# do not automerge "major" version upgrades.
versions = ["minor", "patch"]
usernames = ["dependabot"]
```

Dependency upgrade types are parsed from the pull request title. The following table shows version upgrade examples:

| title                           | upgrade |
| ------------------------------- | ------- |
| Bump lodash from 1.0.0 to 1.0.1 | patch   |
| Bump lodash from 2.5.1 to 2.8.0 | minor   |
| Bump lodash from 4.2.1 to 5.0.0 | major   |

If Kodiak cannot determine the upgrade type from the pull request title, Kodiak will not automerge the pull request.

See the [tests file](https://github.com/chdsbd/kodiak/blob/b1893ee6add4a1533bdac77999aad698e0b2e74c/bot/kodiak/test_dependencies.py#L10-L35) for more examples.

### `merge.automerge_dependencies.usernames`

- **type:** `string[]`
- **default:** `[]`

Kodiak will only automerge dependency upgrades for pull request authors in this list.

See ["Configuring automerge by upgrade type"](recipes.md##configuring-automerge-by-upgrade-type) for a full example.

```toml
# .kodiak.toml
[merge.automerge_dependencies]
versions = ["minor", "patch"]
# only automerge by upgrade version for pull requests authored by dependabot.
usernames = ["dependabot"]
```

<span id="mergeblacklist_title_regex"/> <!-- handle old links -->

### `merge.blocking_title_regex`

- **type:** `string`
- **default:** `"^WIP:.*"`
- **options:** Regex pattern or `""`
- **alias:** `merge.blacklist_title_regex`

If a PR's title matches this regex, Kodiak will not merge the PR. This is useful
to prevent merging work-in-progress PRs.

Setting `merge.blocking_title_regex = ""` disables this option.

#### example

```
merge.blocking_title_regex = ".*DONT\s*MERGE.*"
```

> **NOTE:** `merge.blocking_title_regex` is a new name for `merge.blacklist_title_regex`. Both options behave identically.

<span id="mergeblacklist_labels"/> <!-- handle old links -->

### `merge.blocking_labels`

- **type:** `string[]`
- **default:** `[]`
- **options:** List of label names
- **alias:** `merge.blacklist_labels`

Kodiak will not merge a PR with any of these labels.

#### example

```
merge.blocking_labels = ["wip"]
```

> **NOTE:** `merge.blocking_labels` is a new name for `merge.blacklist_labels`. Both options behave identically.

### `merge.method`

- **type:** `string`
- **default:** first valid merge method in list `"merge"`, `"squash"`, `"rebase"`
- **options:** `"merge"`, `"squash"`, `"rebase"`, `"rebase_fast_forward"`

Choose merge method for Kodiak to use.

If not configured, the first valid merge method in the list `"merge"`, `"squash"`, `"rebase"` will be used. For example, if `"squash"` and `"rebase"` are the only valid options on a repository, Kodiak will choose `"squash"` since it's the first valid option.

Kodiak will report a configuration error if the selected merge method is disabled for a repository.

If you're using the "Require signed commits" GitHub Branch Protection setting to require commit signatures, _`"merge"` or `"squash"` are the only compatible options_. `"rebase"` will cause Kodiak to raise a configuration error.

The `"rebase_fast_forward"` option is similar to `"rebase"`, but commits are rebased without being rewriten. This option is helpful if your build system depends on git commit hashes instead of git tree hashes for build deduplication.

Since the `"rebase_fast_forward"` option only works with fast forward merges, you must enable the "Require branches to be up to date before merging" branch protection setting with at least one status check.

### `merge.delete_branch_on_merge`

- **type:** `boolean`
- **default:** `false`

Once a PR is merged, delete the branch.

This option behaves like the GitHub repository setting "Automatically delete head branches", which automatically deletes head branches after pull requests are merged.

If "Automatically delete head branches" is enabled on the repository via the GitHub UI, Kodiak will _not_ attempt branch deletion.

### `merge.block_on_reviews_requested`

- **type:** `boolean`
- **default:** `false`

> **DEPRECATED**
>
> Due to limitations with the GitHub API this feature is fundamentally broken and cannot be fixed. Prefer the GitHub branch protection "required reviewers" setting instead.
>
> When a user leaves a comment on a PR, GitHub counts that as satisfying a review request, so the PR will be allowed to merge, even though a reviewer was likely just starting a review.
>
> See this issue comment for more information: [chdsbd/kodiak#153 (comment)](https://github.com/chdsbd/kodiak/issues/153#issuecomment-523057332)

If you request review from a user, don't merge until that user provides a review, even if the PR is passing all status checks.

### `merge.notify_on_conflict`

- **type:** `boolean`
- **default:** `true`

If there is a merge conflict, make a comment on the PR and remove the
automerge label.

This option only applies when `merge.require_automerge_label` is enabled.

### `merge.optimistic_updates`

- **type:** `boolean`
- **default:** `true`

_Assuming your status checks pass most of the time, it's useful to leave `merge.optimistic_updates` enabled._

If Kodiak is merging an out-of-date pull request that has running status checks, update the pull request's branch without waiting for the running status checks to finish.

This setting can speed up merges by not waiting for status checks to finish on out-of-date pull requests.

> **Note:** The "Require branches to be up to date before merging" GitHub Branch Protection setting must be enabled for Kodiak to update branches.

### `merge.dont_wait_on_status_checks`

- **type:** `string[]`
- **default:** `[]`
- **options:** List of check names

Don't wait for specified status checks when merging a PR. If a configured status check is incomplete when a PR is being merged, Kodiak will skip the PR.

Use this option for status checks that run indefinitely, like deploy jobs or the WIP GitHub App.

#### example

```
merge.dont_wait_on_status_checks = ["ci/circleci: deploy", "WIP"]
```

### `merge.update_branch_immediately`

- **type:** `boolean`
- **default:** `false`

> **DEPRECATED**
>
> Prefer [`update.always`](#updatealways), which will deliver better behavior in most use cases. `merge.update_branch_immediately` only affects PRs eligible for merging, while `update.always` will keep all PRs up-to-date.

Update PRs that are passing all branch requirements or are waiting for status checks to pass.

### `merge.prioritize_ready_to_merge`

- **type:** `boolean`
- **default:** `false`

If a PR is passing all checks and is able to be merged, merge it without
placing it in the merge queue.

This option adds some unfairness where PRs waiting in the queue the longest are not served first.

### `merge.priority_merge_label`

- **type:** `string`
- **default:** `null`

When the label is applied to a pull request, the pull request will be placed at the front of the merge queue.

If two PRs have the `merge.priority_merge_label` label, either one may be first in queue.

### `merge.do_not_merge`

- **type:** `boolean`
- **default:** `false`

Never merge a PR. This option can be used with `update.always` to automatically update a PR without merging.

### `merge.message.title`

- **type:** `enum`
- **default:** `"github_default"`
- **options:** `"github_default"`, `"pull_request_title"`

By default (`"github_default"`), the title depends on [merge.method](config-reference.md#mergemethod):

- In case of `"merge"`, GitHub will create a merge commit title of the form, `Merge pull request #X from Y/Z`.
- In case of `"rebase"`, there is no merge commit, and all commits of your pull request will be added to the target unchanged.
- In case of `"squash"`, the squashed commit title depends on the number of commits in the pull request. If the PR contains only one commit, GitHub uses the title of that commit for the squashed commit title. If there are multiple commits, GitHub uses the pull request title. In both cases the PR number is appended to the title, like `(#543)`. [See the official GitHub documentation for more information](https://docs.github.com/en/free-pro-team@latest/github/collaborating-with-issues-and-pull-requests/about-pull-request-merges#merge-message-for-a-squash-merge).

For `"pull_request_title"`, Kodiak uses the pull request title for both merge and squash commits.

### `merge.message.body`

- **type:** `enum`
- **default:** `"github_default"`
- **options:** `"github_default"`, `"pull_request_body"`, `"empty"`

By default (`"github_default"`), GitHub combines the titles of a PR's commits to create the body
text of a merge commit. `"pull_request_body"` uses the content of the PR to generate
the body content while `"empty"` sets an empty body.

### `merge.message.include_pr_number`

- **type:** `boolean`
- **default:** `true`

Add the PR number to the merge commit title.

This setting replicates GitHub's behavior of automatically adding the PR number to the title of merges created through the UI.

This option only applies when `merge.message.title` does not equal `"github_default"`.

### `merge.message.body_type`

- **type:** `enum`
- **default:** `"markdown"`
- **options:** `"markdown"`, `"plain_text"`, `"html"`

Control the text used in the merge commit. The GitHub default is markdown, but `"plain_text"` or `"html"` can be used to render the pull request body as text or HTML.

This option only applies when `merge.message.body = "pull_request_body"`.

### `merge.message.strip_html_comments`

- **type:** `boolean`
- **default:** `false`

Strip HTML comments (`<!-- some HTML comment -->`) from merge commit body.

This setting is useful for stripping HTML comments created by PR templates.

This option only applies when `merge.message.body_type = "markdown"`.

### `merge.message.cut_body_before`

- **type:** `string`
- **default:** `null`

Remove all content _before_ the configured string in the pull request body.

This setting is useful when we want to include only a part of the pull request description as the commit message.

This option only applies when `merge.message.body_type = "markdown"`.

### `merge.message.cut_body_after`

- **type:** `string`
- **default:** `null`

Remove all content _after_ the configured string in the pull request body.

This setting is useful when we want to include only a part of the pull request description as the commit message.

This option only applies when `merge.message.body_type = "markdown"`.

### `merge.message.include_coauthors`

- **type:** `boolean`
- **default:** `false`

If a PR includes commits authored by other users, add those users as coauthors to the pull request.

Coauthors are added using the [`Co-authored-by` trailer syntax](https://help.github.com/en/github/committing-changes-to-your-project/creating-a-commit-with-multiple-authors#creating-co-authored-commits-on-github).

This setting only applies when `merge.message.body = "pull_request"` is set.

#### Example commit

Notice the `Co-authored-by` trailer added by this option.

```text
Add new github login flow (#17)

This change adds the new GitHub social login flow. Steve provided some great UI tweaks.

Co-authored-by: Steve Dignam <7340772+sbdchd@users.noreply.github.com>
```

<img height="80px" title="Coauthors example screenshot" src="/img/coauthors-example.png"/>

### `merge.message.include_pull_request_author`

- **type:** `boolean`
- **default:** `false`

Add the pull request author as a coauthor of the merge commit using `Co-authored-by: jdoe <828352+jdoe@users.noreply.github.com>` syntax.

This setting only applies when `merge.message.body = "pull_request"` is set.

This setting was added to mitigate the fallout of GitHub's change to the
squash method on March 4th, 2020. GitHub reverted their change around
March 6th, 2020, making this option no longer necessary.

### `merge.message.include_pull_request_url`

- **type:** `boolean`
- **default:** `false`

Append the pull request's URL to the commit message body.

This can make accessing the relevant PR for a given commit easier.

This setting only applies when `merge.message.body = "pull_request"` is set.

### `update.always`

- **type:** `boolean`
- **default:** `false`

Update a PR whenever out of date with the base branch. The PR will be
updated regardless of merge requirements (e.g. failing status
checks, missing reviews, blacklist labels).

Kodiak will only update PRs with the `merge.automerge_label` label or if `update.require_automerge_label = false`.

When enabled, _Kodiak will not be able to efficiently update PRs._ If you have multiple PRs against a target like `master`, any time a commit
is added to `master` _all_ of those PRs against `master` will update. For `N` PRs against a target you will see at least `N(N-1)/2` updates. If
this configuration option was disabled you would only see at least `N-1` updates.

### `update.require_automerge_label`

- **type:** `boolean`
- **default:** `true`

When enabled, Kodiak will only update PRs that have an automerge label (configured via `merge.automerge_label`).

When disable, Kodiak will update any PR.

This option only applies when `update.always = true`.

### `update.autoupdate_label`

- **type:** `string`
- **default:** `null`

Pull requests with the `update.autoupdate_label` will be updated when they are out-of-date with their base branch.

This configuration option is similar to `update.always`, but only pull requests with the configured label are affected.

#### example

```toml
version = 1

[merge]
automerge_label = "ship it!"

[update]
autoupdate_label = "update me please!"
```

<span id="updateblacklist_usernames"/> <!-- handle old links -->

### `update.ignored_usernames`

- **type:** `string[]`
- **default:** `[]`
- **alias:** `update.blacklist_usernames`

When a pull request's author matches `update.ignored_usernames`, Kodiak will never update the pull request, unless `update.autoupdate_label` is applied to the pull request.

If the user is a bot user, remove the `[bot]` suffix from their username. So instead of `dependabot-preview[bot]`, use `dependabot-preview`.

#### example

```
update.ignored_usernames = ["bernard-lowe", "dependabot-preview"]
```

> **NOTE:** `update.ignored_usernames` is a new name for `update.blacklist_usernames`. Both options behave identically.

### `approve.auto_approve_usernames`

- **type:** `string[]`
- **default:** `[]`
- **options:** List of GitHub usernames

If a PR is opened by a user with a username in the `approve.auto_approve_usernames` list, Kodiak will automatically add an approval to the PR.

If Kodiak's review is dismissed, it will add a review again.

This setting is useful when the "Required approving reviews" GitHub Branch Protection setting is configured and dependency upgrade bots like dependabot, greenkeeper, etc, run on the repository. When these bots open a PR, Kodiak can automatically add a review so dependency upgrade PRs can be automatically merged.

See the "[Automated dependency updates with Dependabot](recipes.md#automated-dependency-updates-with-dependabot)" recipe for an example of this feature in action.

## configuration labels

Some configuration options set in `.kodiak.toml` can be overriden with pull request labels.

Please [open an issue on GitHub](https://github.com/chdsbd/kodiak/issues/new/choose) if you would like support for more options.

### `merge.method`

The [`merge.method`](config-reference.md#mergemethod) option can be overriden with a label, allowing you to merge a pull request with a `merge.method` that differs from your `.kodiak.toml` setting.

#### Example

Label your pull request with the `kodiak: merge.method = 'rebase'` label to set `merge.method` to "rebase" for your pull request.

Here's a table of all the options:

| merge method | label                             |
| ------------ | --------------------------------- |
| merge        | `kodiak: merge.method = 'merge'`  |
| squash       | `kodiak: merge.method = 'squash'` |
| rebase       | `kodiak: merge.method = 'rebase'` |

## full examples

### minimal

```toml
# .kodiak.toml
# docs: https://kodiakhq.com/docs/config-reference
version = 1
```

### all options

Below is a Kodiak config with all options set and commented.

```toml
# .kodiak.toml

# Kodiak's configuration file should be placed at `.kodiak.toml` (repository
# root) or `.github/.kodiak.toml`.
# docs: https://kodiakhq.com/docs/config-reference

# version is the only required setting in a kodiak config.
# `1` is the only valid setting for this field.
version = 1

[merge]
# Label to enable Kodiak to merge a PR.

# By default, Kodiak will only act on PRs that have this label. You can disable
# this requirement via `merge.require_automerge_label`.
automerge_label = "automerge" # default: "automerge"

# Require that the automerge label (`merge.automerge_label`) be set for Kodiak
# to merge a PR.
#
# When disabled, Kodiak will immediately attempt to merge any PR that passes all
# GitHub branch protection requirements.
require_automerge_label = true

# If a PR's title matches this regex, Kodiak will not merge the PR. This is
# useful to prevent merging work-in-progress PRs.
#
# Setting `merge.blocking_title_regex = ""` disables this option.
blocking_title_regex = "" # default: "^WIP:.*", options: "" (disables regex), a regex string (e.g. ".*DONT\s*MERGE.*")

# Kodiak will not merge a PR with any of these labels.
blocking_labels = [] # default: [], options: list of label names (e.g. ["wip"])

# Choose merge method for Kodiak to use.
#
# Kodiak will report a configuration error if the selected merge method is
# disabled for a repository.
#
# If you're using the "Require signed commits" GitHub Branch Protection setting
# to require commit signatures, "merge" or "squash" are the only compatible options. "rebase" will cause Kodiak to raise a configuration error.
method = "merge" # default: first valid merge method in list "merge", "squash", "rebase", options: "merge", "squash", "rebase"

# Once a PR is merged, delete the branch. This option behaves like the GitHub
# repository setting "Automatically delete head branches", which automatically
# deletes head branches after pull requests are merged.
delete_branch_on_merge = false # default: false

# DEPRECATED
#
# Due to limitations with the GitHub API this feature is fundamentally broken
# and cannot be fixed. Prefer the GitHub branch protection "required reviewers"
# setting instead.
#
# When a user leaves a comment on a PR, GitHub counts that as satisfying a
# review request, so the PR will be allowed to merge, even though a reviewer was
# likely just starting a review.
#
# See this issue comment for more information:
# https://github.com/chdsbd/kodiak/issues/153#issuecomment-523057332
#
# If you request review from a user, don't merge until that user provides a
# review, even if the PR is passing all status checks.
block_on_reviews_requested = false # default: false

# If there is a merge conflict, make a comment on the PR and remove the
# automerge label. This option only applies when `merge.require_automerge_label`
# is enabled.
notify_on_conflict = true # default: true

# Don't wait for in-progress status checks on a PR to finish before updating the
# branch.
optimistic_updates = false # default: true

# Don't wait for specified status checks when merging a PR. If a configured
# status check is incomplete when a PR is being merged, Kodiak will skip the PR.
# Use this option for status checks that run indefinitely, like deploy jobs or
# the WIP GitHub App.
dont_wait_on_status_checks = [] # default: [], options: list of check names (e.g. ["ci/circleci: lint_api"])

# DEPRECATED
#
# Prefer `update.always`, which will deliver better behavior in most use cases.
# `merge.update_branch_immediately` only affects PRs eligible for merging, while
# `update.always` will keep all PRs up-to-date.
#
# Update PRs that are passing all branch requirements or are waiting for status
# checks to pass.
update_branch_immediately = false # default: false

# If a PR is passing all checks and is able to be merged, merge it without
# placing it in the merge queue. This option adds some unfairness where PRs
# waiting in the queue the longest are not served first.
prioritize_ready_to_merge = false # default: false

# Never merge a PR. This option can be used with `update.always` to
# automatically update a PR without merging.
do_not_merge = false # default: false

[merge.message]
# By default (`"github_default"`), GitHub uses the title of a PR's first commit
# for the merge commit title. `"pull_request_title"` uses the PR title for the
# merge commit.
title = "github_default" # default: "github_default", options: "github_default", "pull_request_title"

# By default (`"github_default"`), GitHub combines the titles of a PR's commits
# to create the body text of a merge commit. `"pull_request_body"` uses the
# content of the PR to generate the body content while `"empty"` sets an empty
# body.
body = "github_default" # default: "github_default", options: "github_default", "pull_request_body", "empty"

# Option to mitigate the fallout of GitHub's change to the
# squash method on March 4th, 2020. GitHub reverted their change around
# March 6th, 2020, making this option no longer necessary.
include_pull_request_author = false # default: false

# Append the Pull Request URL to the merge message. Makes navigating to the PR
# from the commit easier.
include_pull_request_url = false # default: false

# Add the PR number to the merge commit title. This setting replicates GitHub's
# behavior of automatically adding the PR number to the title of merges created
# through the UI. This option only applies when `merge.message.title` does not
# equal `"github_default"`.
include_pr_number = true # default: true

# Control the text used in the merge commit. The GitHub default is markdown, but
# `"plain_text"` or `"html"` can be used to render the pull request body as text
# or HTML. This option only applies when `merge.message.body = "pull_request_body"`.
body_type = "markdown" # default: "markdown", options: "plain_text", "markdown", "html"


# Strip HTML comments (`<!-- some HTML comment -->`) from merge commit body.
# This setting is useful for stripping HTML comments created by PR templates.
# This option only applies when `merge.message.body_type = "markdown"`.
strip_html_comments = false # default: false

# Remove all content before the configured string in the pull request body.
# This setting is useful when we want to include only a part of the pull request
# description as the commit message.
# This option only applies when `merge.message.body_type = "markdown"`.
cut_body_before = "<!-- commit body -->"

[update]

# Update a PR whenever out of date with the base branch. The PR will be updated
# regardless of merge requirements (e.g. failing status checks, missing reviews,
# blacklist labels).
#
# Kodiak will only update PRs with the `merge.automerge_label` label or if
# `update.require_automerge_label = false`.
#
# When enabled, _Kodiak will not be able to efficiently update PRs._ If you have
# multiple PRs against a target like `master`, any time a commit is added to
# `master` _all_ of those PRs against `master` will update. For `N` PRs against
# a target you will see at least `N(N-1)/2` updates. If this configuration
# option was disabled you would only see at least `N-1` updates.
always = false # default: false

# When enabled, Kodiak will only update PRs that have an automerge label
# (configured via `merge.automerge_label`). When disable, Kodiak will update any
# PR. This option only applies when `update.always = true`.
require_automerge_label = true # default: true
```

## other resources

See [`bot/kodiak/test/fixtures/config`](https://github.com/chdsbd/kodiak/tree/master/bot/kodiak/test/fixtures/config) for more examples and [`bot/kodiak/config.py`](https://github.com/chdsbd/kodiak/blob/master/bot/kodiak/config.py) for the Python models that codify this configuration.
