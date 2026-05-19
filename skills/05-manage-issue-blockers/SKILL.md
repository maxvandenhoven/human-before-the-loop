---
name: 05-manage-issue-blockers
description: Add, remove, or list "blocked by" relationships between GitHub issues using GitHub's native issue-dependencies REST API. Use when create-issues needs to encode dependencies, or when inspecting blockers. Only works on github.com (not GHES).
---

# Manage issue blockers

GitHub's issue-dependency endpoints are not yet exposed as first-class `gh` flags, so the operations below use raw `gh api` calls. The API takes `issue_id` — the issue's numeric `databaseId` — NOT the repo-scoped issue number.

To convert an issue number to its databaseId:

    gh api /repos/{owner}/{repo}/issues/<number> --jq '.id'

`gh api` automatically substitutes `{owner}` and `{repo}` with the current repository.

## Add a blocker

To mark issue `<blocked_num>` as blocked by `<blocker_num>`:

    blocker_id=$(gh api /repos/{owner}/{repo}/issues/<blocker_num> --jq '.id')
    gh api \
      -X POST \
      /repos/{owner}/{repo}/issues/<blocked_num>/dependencies/blocked_by \
      -F issue_id="$blocker_id"

Use `-F` (uppercase) so `issue_id` is sent as an integer. `-f` (lowercase) sends a string and the API rejects it.

## Remove a blocker

The path parameter is also the databaseId, not the issue number:

    blocker_id=$(gh api /repos/{owner}/{repo}/issues/<blocker_num> --jq '.id')
    gh api \
      -X DELETE \
      /repos/{owner}/{repo}/issues/<blocked_num>/dependencies/blocked_by/$blocker_id

## List blockers of an issue

Returns the array of issues blocking `<num>`:

    gh api /repos/{owner}/{repo}/issues/<num>/dependencies/blocked_by

For just `(number, title)` pairs:

    gh api /repos/{owner}/{repo}/issues/<num>/dependencies/blocked_by \
      --jq '.[] | "#\(.number)  \(.title)"'

## List issues that this one blocks

    gh api /repos/{owner}/{repo}/issues/<num>/dependencies/blocking
