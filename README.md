# Specifications Workflow

This repository manages the specifications of the Meilisearch API. Specifications are meant to describe the expected behavior on a high level and point out identified corner cases.

## Draft State

To start a new specification, a new branch must start from `main` or from `release-vX.X.X` if the changes to write are already planned for an upcoming release.

If a new specification file need to be introduced it must follow the pattern: `PR_number-feature-name.md`. e.g. if PR number 12 is about facetting, the newly introduced specification file will be named `0012-facetting.md`.

> Note that a pull request not strictly dealing about a specification conception will be tagged as `Not A Spec`. e.g. A pull-request updating this file will be tagged with the `Not A Spec` label.

## Review State

Once the specification is ready for review in the owner's eyes, the owner can then switch the PR to open.

The PR must be tagged as `Ready For Review` to enter this stage.

To be validated, it must be reviewed and approved by peers, ideally:

- One person from the Integration team.
- One person from the Core team.
- One person from the Product team.

## Merge State

To be merged, a specification pull-request should follow the given rules:

- It must be approved as described in the [Review State](#review-state) section.
- It must be branched on a given `release-vX.X.X` branch.
- It must be tagged with:
  - A `vX.X.X` tag indicating in which release the described changes will be introduced.
  - A `QX:YYYY` tag indicating in which quarter and year the described changes will be introduced.
  - The `Telemetry` tag, if telemetry changes are introduced.
  - The `OpenAPI` tag, if the [open-api](open-api.yaml) specification will see changes introduced.

---

# Release Worfklow

The following steps should happens the day a Meilisearch release is shipped:

- Pull-requests describing changes for a release are merged and squashed into the corresponding `release-v.X.X.X` branch.
- `release-vX.X.X` is merged into `main`.
- `open-api.yml` version is deployed on bump.sh.

---

## Specification File Format

Meilisearch's feature specifications are made up of five sections, described below.

### 1. Summary

Summrize the content of the specification with a short paragraph.

### 2. Motivation

Explains what use cases does it support.

### 3. Functional Specification

This section gives a high level overview of the feature. It should avoid technical language so that it can be understood by a general audience (think user-level).

- Describes the API resource and endpoints. (Methods, URL, query parameters, body definition, status code).
- Explains the feature through examples.
- Lists error cases.

### 4. Technical Details (Optional)

When needed, we recommend describing practical aspects of implementation, e.g. specific algorithmic choices. If none, fill the section body with "n/a".

### 5. Future Possibilities (Optional)

This last section includes any related topics or features which are not currently in Meilisearch and will not be added now, but which may be explored in the future. If none, fill the section body with "n/a".
