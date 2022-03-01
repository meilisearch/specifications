- Title: Typo Tolerance API
- Start Date: 2022-02-22

# Typo Tolerance API

## 1. Functional Specification

## 1.1. Summary

This specification describes the configuration options for the typo tolerance feature, both as settings for an index and as overridable parameters at search time.

## 1.2. Explanations

Meilisearch exposes an API to configure the typo tolerance behavior at the level of index settings.

[GET/POST/DEL] - `indexes/:index_uid/settings`
[GET/POST/DEL] - `indexes/:index_uid/settings/typoTolerance`

The [Search API](0118-search-api.md) permits to override those index settings at search time.

[GET/POST] - `indexes/:index_uid/search`

> See [Search API](0118-search-api.md)

- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

If the instance is secured by a master-key, the auth layer will return the following errors:

- 🔴 Accessing these routes without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

`POST` HTTP verb errors:

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.

## 1.2.1. Typo Tolerance Payload

| Field                | Type            | Required |
|----------------------|-----------------|----------|
| enabled              | Boolean         | True     |
| disableOnAttributes  | Array of String | True     |
| disableOnWords       | Array of String | True     |
| minWordSizeFor1Typo  | Integer         | True     |
| minWordSizeFor2Typos | Integer         | True     |

### 1.2.1.1 `enabled`

- Type: Boolean
- Required: True
- Default: `true`

Whether the typo tolerance feature is enabled.

> The presence of `typo` in the ranking rules setting does not influence the activation/deactivation of the typo tolerance feature. If the `rankingRules` parameter of the index settings does not contain the `typo` rule, the results are not sorted according to the number of typos found.

### 1.2.1.2 `disableOnAttributes`

- Type: Array of String
- Required: True
- Default: `[]`

`disableOnAttributes` disable the typo tolerance feature on the specified document attributes.

### 1.2.1.3 `disableOnWords`

- Type: Array of String
- Required: True
- Default: `[]`

`disableOnWords` disable the typo tolerance feature for a set of query terms given during a search query.

### 1.2.1.4 `minWordSizeFor1Typo`

- Type: Integer
- Required: True
- Default: `5`

`minWordSizeFor1Typo` customize the minimum size for a word to tolerate 1 typo.

> Given the default value `5`, the search engine starts to apply typo tolerance on a query term if its length is at least equal to 5 characters.

### 1.2.1.5 `minWordSizeFor2Typos`

- Type: Integer
- Required: True
- Default: `9`

`minWordSizeFor2Typos` customize the minimum size for a word to tolerate 2 typos.

> Given the default value `9`, the search engine handles up to 2 typos on a query term if its length is at least equal to 9 characters.

## 2. Technical Details
tbd

## 3. Future Possibilities
n/a