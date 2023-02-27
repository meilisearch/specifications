# Multi-search API

## 1. Summary

The multi-search endpoint performs a batch of search queries as if using the [search API](./0118-search-api.md).

## 2. Motivation

- Perform multiple queries in a single HTTP request

## 3. Functional Specification

Meilisearch exposes 1 route to perform multi-search requests:

- POST `/multi-search`

If a master key is used to secure a Meilisearch instance, the auth layer returns the following errors:

- 🔴 Accessing these routes without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing these routes with a key that does not have permissions (i.e. other than the master key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

If any of the search queries fail to execute, the response returns the corresponding error instead of the array of results.

`POST` HTTP verb errors:

- 🔴 Omitting the Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.

### 3.1. Search Payload Parameters

| Field        | Type             | Required |
|--------------|------------------|----------|
| [`queries`](#311-queries)  | Array of Objects | True     |

#### 3.1.1. `queries`

- Type: Object
- Required: True

The `queries` object contains the list of search queries to perform.

Each element of this array is a JSON object with the required field `indexUid`, the uid of the index to be searched. Other fields of this object are optional, and identical to the ones in the [existing search routes](./0118-search-api.md#31-search-payload-parameters) (`q`, `limit`, etc.).

### 3.2. Search Response Properties

| Field                     | Type          | Required |
|---------------------------|---------------|----------|
| [`results`](#321-results) | Array[Object] | True     |

#### 3.2.1. `results`

- Type: Array[Objets]
- Required: True

Results of the search queries as an array of search results.

Each element of this array is a JSON object representing the results of the search query with the same index in the `queries` array from the request, and an additional `indexUid` field that is equal to the `indexUid` field in that search query.

The other fields of an element from the `results` array are identical to the fields of the [response from the other search routes](./0118-search-api.md#31-formatting-search-results).

## 2. Technical Details
n/a

## 3. Future Possibilities

- Allow specifying an index uid pattern instead of an index uid to produce searches on all indexes matching the pattern.
- Allow additional arguments to the request specifying a strategy to aggregate results from the multiple searches.
