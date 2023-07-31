# Runtime experimental feature API

## 1. Summary

The runtime experimental feature API allows toggling the status of some [experimental features](./0193-experimental-features.md) at runtime.

Due to its nature, this route itself is permanently experimental, in that the way of using it is not covered by [Meilisearch's stability guarantee](https://github.com/meilisearch/engine-team/blob/main/resources/versioning-policy.md).

## 2. Motivation

Historically, experimental features in the engine must be enabled from the CLI or via environment variables.

The problem is that it requires restarting Meilisearch, so:

- It induces downtime.
- Makes experimental features harder to enable for [Meilisearch Cloud](https://www.meilisearch.com/pricing?utm_campaign=oss&utm_source=engine&utm_medium=specifications) instances.

The motivation of this feature is to remove these issues by allowing enabling and disabling experimental features at runtime.

Due to the nature of some experimental features they might not be in scope for this API. The experimental features in scope for this API are called [*runtime* experimental features](./0193-experimental-features.md#32-runtime-experimental-features), while the ones not in scope are called [*instance* experimental features](./0193-experimental-features.md#31-instance-experimental-features).

## 3. Functional Specification

### 3.1 Routes

Meilisearch exposes 2 routes to get or set the status of runtime experimental features.

- GET `/experimental-features`: get the status of all the runtime experimental features.
- PATCH `/experimental-features`: set the status of some of the runtime experimental features.

All routes return the status of the runtime experimental features after calling the route.

This response is a JSON object containing the following fields:

|Field name|Type|Experimental feature|
|----------|----|-----|
|`scoreDetails`|Boolean| [Score details](./0193-experimental-features.md#score-details) |
|`vectorStore`|Boolean| [Vector store](./0193-experimental-features.md#vector-store) |

The PATCH routes accept as payload a JSON object containing the same fields as in the response, with the following effects on the corresponding feature:
- Setting a field to `true` enables the feature.
- Setting a field to `false` disables the feature.
- Setting a field to `null` or omitting a field leaves its value unchanged.

### 3.2 Errors

- 🔴 [`bad_request`](./0061-error-format-and-definitions.md#bad_request) for unknown fields in the payload or whenever a field is not a boolean.

## 4. Technical Details

N/A

## 5. Future Possibilities

N/A