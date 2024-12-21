# pr-size

A GitHub Action for labelling pull requests with size categories.

<!-- TODO: Add picture -->
<!-- 100 chars ------------------------------------------------------------------------------------>

There is a limit to how much code can be effectively reviewed in a single pull request. A
[Smartbear study][1] found that reviewing more than 200-400 lines reduces the ability of reviewers
to identify defects. Furthermore, smaller changes are typically lower risk and require less effort
to qualify. With that in mind, this action seeks to promote smaller pull requests by making it
easier to identify excessively large changes.

Each pull request is assigned a label that identifies its size. The thresholds for the sizes and the
names of the labels can be customised using the [action inputs](#inputs).

| Category          | Lines changed | Label      |
| ----------------- | ------------- | ---------- |
| Extra Small       | 1-10          | `size/XS`  |
| Small             | 11-100        | `size/S`   |
| Medium            | 101-200       | `size/M`   |
| Large             | 201-400       | `size/L`   |
| Extra Large       | 401-800       | `size/XL`  |
| Extra Extra Large | 801+          | `size/XXL` |

The calculation of the lines changed is determined by adding the number of lines added and removed
in each file. Whitespace only changes to lines, such as indentation modifications are excluded.
Additionally, files that are marked as generated or vendored in `.gitattributes` are also excluded,
allowing automated changes to be filtered out. For example:

```gitignore
# See https://github.com/github-linguist/linguist/blob/main/docs/overrides.md for further details
vendor/**    linguist-vendored
generated/** linguist-generated
```

[1]: https://smartbear.com/learn/code-review/best-practices-for-peer-code-review

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

| Name            | Type   | Default      | Description                                                                  |
| --------------- | ------ | ------------ | ---------------------------------------------------------------------------- |
| `xs-threshold`  | String | `10`         | The maximum number of lines changed for an extra small label to be assigned. |
| `s-threshold`   | String | `100`        | The maximum number of lines changed for a small label to be assigned.        |
| `m-threshold`   | String | `200`        | The maximum number of lines changed for a medium label to be assigned.       |
| `l-threshold`   | String | `400`        | The maximum number of lines changed for a large label to be assigned.        |
| `xl-threshold`  | String | `800`        | The maximum number of lines changed for an extra large label to be assigned. |
| `xs-label`      | String | `size/XS`    | The name of the label for a very small number of lines changed.              |
| `s-label`       | String | `size/S`     | The name of the label for a small number of lines changed.                   |
| `m-label`       | String | `size/M`     | The name of the label for a medium number of lines changed.                  |
| `l-label`       | String | `size/L`     | The name of the label for a large number of lines changed.                   |
| `xl-label`      | String | `size/XL`    | The name of the label for a very large number of lines changed.              |
| `xxl-label`     | String | `size/XXL`   | The name of the label for a very, very large number of lines changed.        |
| `github-token`  | String | github.token | TODO |

<!-- TODO: discuss how labels can be modified post-creation -->

## Outputs

| Name       | Type   | Description |
| ---------- | ------ | ----------- |
| `label`    | String | TODO |
| `size`     | String | TODO |
| `includes` | String | TODO |
| `excludes` | String | TODO |

## Recipes

<!-- TODO: populate -->
