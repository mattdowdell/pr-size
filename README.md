# pr-size

A GitHub Action for labelling pull requests with size categories.

<!-- TODO: Add picture -->

There is a limit to how much code can be effectively reviewed in a single pull
request. A [Smartbear study][1] found that reviewing more than 200-400 lines
reduces the ability of reviewers to identify defects. Furthermore, smaller
changes are typically lower risk and require less effort to qualify. With that
in mind, this action seeks to promote smaller pull requests by making it easier
to identify excessively large changes.

Each pull request is assigned a label that identifies its size. The thresholds
for the sizes and the names of the labels can be customised using the
[action inputs](#inputs).

| Category          | Lines changed | Label      |
| ----------------- | ------------- | ---------- |
| Extra Small       | 1-10          | `size/XS`  |
| Small             | 11-100        | `size/S`   |
| Medium            | 101-200       | `size/M`   |
| Large             | 201-400       | `size/L`   |
| Extra Large       | 401-800       | `size/XL`  |
| Extra Extra Large | 801+          | `size/XXL` |

The calculation of the lines changed is determined by adding the number of lines
added and removed in each file. Whitespace only changes to lines, such as
indentation modifications are excluded. Additionally, files that are marked as
generated or vendored in `.gitattributes` are also excluded, allowing automated
changes to be filtered out. For example:

```gitignore
vendor/**    linguist-vendored
generated/** linguist-generated
```

See [Linguist's overrides][2] for further documentation.

[1]: https://smartbear.com/learn/code-review/best-practices-for-peer-code-review
[2]: https://github.com/github-linguist/linguist/blob/main/docs/overrides.md

## Usage

```yaml
name: PR Size
on:
  pull_request:
jobs:
  pr-size:
    name: PR Size
    runs-on: ubuntu-latest
    permissions:
      contents: read       # for checkout
      pull-requests: write # for creating/adding/removing labels
    steps:
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      # TODO: change to a tag once a release has been created
      - uses: mattdowdell/pr-size@main
```

## Inputs

| Name           | Type   | Default           | Description                                                                  |
| -------------- | ------ | ----------------- | ---------------------------------------------------------------------------- |
| `xs-threshold` | String | `10`              | The maximum number of lines changed for an extra small label to be assigned. |
| `s-threshold`  | String | `100`             | The maximum number of lines changed for a small label to be assigned.        |
| `m-threshold`  | String | `200`             | The maximum number of lines changed for a medium label to be assigned.       |
| `l-threshold`  | String | `400`             | The maximum number of lines changed for a large label to be assigned.        |
| `xl-threshold` | String | `800`             | The maximum number of lines changed for an extra large label to be assigned. |
| `xs-label`     | String | `size/XS`         | The name of the label for a very small number of lines changed.              |
| `s-label`      | String | `size/S`          | The name of the label for a small number of lines changed.                   |
| `m-label`      | String | `size/M`          | The name of the label for a medium number of lines changed.                  |
| `l-label`      | String | `size/L`          | The name of the label for a large number of lines changed.                   |
| `xl-label`     | String | `size/XL`         | The name of the label for a very large number of lines changed.              |
| `xxl-label`    | String | `size/XXL`        | The name of the label for a very, very large number of lines changed.        |
| `github-token` | String | [github.token][3] | The token to use for managing labels.                                        |

<!-- TODO: discuss how labels can be modified post-creation -->

[3]:
  https://docs.github.com/en/actions/security-for-github-actions/security-guides/automatic-token-authentication

## Outputs

| Name       | Type   | Description                                        |
| ---------- | ------ | -------------------------------------------------- |
| `label`    | String | The label assigned to the pull request.            |
| `size`     | String | The calculated size of the pull request's changes. |
| `includes` | String | The files included in the size calculation.        |
| `excludes` | String | The files excluded from the size calculation.      |

## Recipes

Labelling pull requests with a size is a good start, but the real value comes
from tracking the size of changes over time. The below recipes are aimed at
achiving that using the [`gh`][4] CLI.

These example use the `gh api` subcommand instead of `gh pr list` to avoid
missing data due to the `--limit` option.

[4]: https://cli.github.com/

### List all sized PRs

The following lists all pull requests merged to the `main` branch with their
respective size labels.

```sh
gh api \
  /repos/mattdowdell/pr-size/pulls\?state=closed \
  --paginate \
  --jq '.[]
    | select((.merged_at != null) and (.base.ref == "main"))
    | "PR #\(.number): Merged at: \(.merged_at), \(.labels[].name | select(. | startswith("size/")))"'
```

Example output:

```text
PR #27: Merged at: 2024-12-23T09:04:04Z, size/M
PR #26: Merged at: 2024-12-21T11:34:34Z, size/XXL
PR #23: Merged at: 2024-12-20T10:05:25Z, size/XL
PR #21: Merged at: 2024-12-23T09:35:51Z, size/M
PR #15: Merged at: 2024-12-19T12:19:42Z, size/S
```

### List sized PRs between 2 points

It can be useful to track the size of changes between 2 points in time, e.g. the
changes going into an upcoming release. The below lists the pull requests merged
to the `main` branch between 2 dates with their respective size labels.

```sh
gh api \
  /repos/mattdowdell/pr-size/pulls\?state=closed \
  --paginate \
  --jq '.[]
    | select(
      .merged_at > "2024-12-19T12:19:42Z" and
      .merged_at <= "2024-12-23T09:04:04Z" and
      .base.ref == "main"
    )
    | "PR #\(.number): Merged at: \(.merged_at), \(.labels[].name | select(. | startswith("size/")))"'
```

If the pull requests between 2 commits or tags are preferred, each date can be
identified with the following and substituted in:

```sh
git show --no-patch --date=format:'%Y-%m-%dT%H:%M:%S' --format=%cd <commit-or-tag>
```

Example output:

```text
PR #27: Merged at: 2024-12-23T09:04:04Z, size/M
PR #26: Merged at: 2024-12-21T11:34:34Z, size/XXL
PR #23: Merged at: 2024-12-20T10:05:25Z, size/XL
```

### Count frequency of sizes over all PRs

Drilling down into the sizes of individual pull requests can help understand why
it was a particular size. However, it is also useful to track trends over time,
such as how many pull requests of each size have been merged. The below counts
the number of pull requests merged to the `main` branch for each size.

```sh
gh api \
  /repos/mattdowdell/pr-size/pulls\?state=closed \
  --paginate \
  --jq '[ .[] | select(.merged_at != null and .base.ref == "main") ]
    | map({ size: .labels[].name | select(. | startswith("size/")) })
    | group_by(.size)
    | .[]
    | "\(length) x \(.[0].size)"'
```

Example output:

```text
2 x size/M
1 x size/S
1 x size/XL
1 x size/XXL
```

### Count frequency of sizes between dates

Comparing of the size of changes across multiple releases can help identify
trends, including the number of changes in a release and the sizes of those
changes. The below counts the number of pull requests merged to the `main`
branch between 2 dates for each size.

```sh
gh api \
  /repos/mattdowdell/pr-size/pulls\?state=closed \
  --paginate \
  --jq '[ .[] | select(
      .merged_at > "2024-12-19T12:19:42Z" and
      .merged_at <= "2024-12-23T09:04:04Z" and
      .base.ref == "main"
    ) ]
    | map({ size: .labels[].name | select(. | startswith("size/")) })
    | group_by(.size)
    | .[]
    | "\(length) x \(.[0].size)"'
```

See [above](#list-sized-prs-between-2-points) for how to convert commits and
tags into dates.

Example output:

```text
1 x size/M
1 x size/XL
1 x size/XXL
```

### List PRs with a specific size

It is not impossible for a change to be large out of necessity, even whilst
striving for smaller pull requests. The below will list all pull requests merged
to the `main` branch with a `size/XXL` label.

```sh
gh api \
  /repos/mattdowdell/pr-size/pulls\?state=closed \
  --jq '.[]
    | select(.merged_at != null and .base.ref == "main" and .labels[].name == "size/XXL")
    | "PR #\(.number)"'
```

Example output:

```text
PR #26
```
