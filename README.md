# Specifications Workflow

This repository manages the specifications of the Meilisearch API. Specifications are meant to describe the expected behavior on a high level and point out identified corner cases.

## Draft State: create a new PR

To start a new specification, a new branch must start
- from `release-vX.X.X` if the related changes are already planned for the release `vX.X.X`
- from `main` if you don't know in which release the changes will be integrated

If a new specification file needs to be introduced, you must create a new file in [this folder](https://github.com/meilisearch/specifications/tree/main/text) following the pattern: `PR_number-feature-name.md`. e.g. if PR number 12 is about facetting, the newly introduced specification file will be named `0012-facetting.md`.

> Note that a pull request not strictly dealing about a specification conception will be tagged as `Not A Spec`. e.g. A pull-request updating this file will be tagged with the `Not A Spec` label.

The [pull-request template](pull_request_template.md) must be filled in when the pull-request is created.

## Review State

It's up to the maintainers of this repository to decide when the PR is ready to be reviewed and which persons should review it.

The PR must be tagged as `Ready For Review` to enter this stage.

To be validated, it must be reviewed and approved by peers, ideally:

- One person from the Integration team.
- One person from the Core team.
- One person from the Product team.

## Merge State

To be merged, a specification pull-request should follow the given rules:

- It must be approved as described in the [Review State](#review-state) section.
- The PR must point to the right `release-vX.X.X` branch.
- It must be tagged with:
  - A `vX.X.X` tag indicating in which release the described changes will be introduced.
  - A `QX:YYYY` tag indicating in which quarter and year the described changes will be introduced.
  - The `⚠ Breaking` tag, if breaking changes are introduced.
  - The `Telemetry` tag, if telemetry changes are introduced.
  - The `OpenAPI` tag, if the [open-api](open-api.yaml) specification will see changes introduced.

---

# Release Worfklow

The following steps should happen the day a Meilisearch release is shipped:

- Pull-requests describing changes for a release are squashed and merged into the corresponding `release-vX.X.X` branch.
- `release-vX.X.X` is squashed and merged into `main`.
- `open-api.yml` version is deployed on bump.sh.

---

## Specification File Format

Meilisearch's feature specifications are made up of five sections, described below.

### 1. Summary

Summarize the specification with a short paragraph.

### 2. Motivation

Explain which use cases are supported.

### 3. Functional Specification

This section gives a high level overview of the feature. It should avoid technical language so that it can be understood by a general audience (think user-level).

- Describe the API resource and endpoints. (Methods, URL, query parameters, body definition, status code).
- Explain the feature through examples.
- List error cases.

### 4. Technical Details (Optional)

When needed, we recommend describing practical aspects of implementation, e.g. specific algorithmic choices. If none, fill the section body with "n/a".

### 5. Future Possibilities (Optional)

This last section includes any related topics or features which are not currently in Meilisearch and will not be added now, but which may be explored in the future. If none, fill the section body with "n/a".
