# Settings API

## 1. Summary

This specification describes the global settings API endpoints and cross-reference related sub-resources APIs.

## 2. Motivation
N/A

## 3. Functional Specification

### 3.1. Sub Resource Settings API List

| API Resource                                                       | Description                                                  |
|--------------------------------------------------------------------|--------------------------------------------------------------|
| [displayed-attributes](0123-displayed-attributes-setting-api.md)   | `displayedAttributes` sub-resource API endpoints definition  |
| [searchable-attributes](0123-searchable-attributes-setting-api.md) | `searchableAttributes` sub-resource API endpoints definition |
| [filterable-attributes](0123-filterable-attributes-setting-api.md) | `filterableAttributes` sub-resource API endpoints definition |
| [sortable-attributes](0123-sortable-attributes-setting-api.md)     | `sortableAttributes` sub-resource API endpoints definition   |
| [ranking-rules](0123-ranking-rules-setting-api.md)                 | `rankingRules` sub-resource API endpoints definition         |
| [stop-words](0123-stop-words-setting-api.md)                       | `stopWords` sub-resource API endpoints definition            |
| [separator-tokens](0123-separators-settings-api.md)                | `separatorTokens` sub-resource API endpoints definition      |
| [non-separator-tokens](0123-separators-settings-api.md)            | `nonSeparatorTokens` sub-resource API endpoints definition   |
| [dictionary](0123-user-dictionary-settings-api.md)                 | `dictionary` sub-resource API endpoints definition           |
| [synonyms](0123-synonyms-setting-api.md)                           | `synonyms` sub-resource API endpoints definition             |
| [distinct-attribute](0123-distinct-attribute-setting-api.md)       | `distinctAttribute` sub-resource API endpoints definition    |
| [typo-tolerance](0117-typo-tolerance-setting-api.md)               | `typoTolerance` sub-resource API endpoints definition        |
| [pagination](157-pagination-setting-api.md)                        | `pagination` sub-resource API endpoints definition           |
| [faceting](157-faceting-setting-api.md)                            | `faceting` sub-resource API endpoints definition             |
| [faceting](0123-proximity-precision.md)                            | `proximityPrecision` sub-resource API endpoints definition   |

Each setting is exposed as a sub-resource of the `indexes/:index_uid/settings` endpoints. e.g. The ranking rules setting of a Meilisearch index is exposed at `indexes/:index_uid/settings/ranking-rules`.

Some setting changes cause a complete re-indexing of the documents. See [4.1. Triggering Documents Re-indexing](#41-triggering-documents-re-indexing).

### 3.2. API Endpoints Definition

Manipulate all settings of a Meilisearch index.

#### 3.2.1. `GET` - `indexes/:index_uid/settings`

Fetch the settings of a Meilisearch index.

##### 3.2.1.1. Response Definition

| Field                    | Type                    | Required |
|--------------------------|-------------------------|----------|
| `displayedAttributes`    | Array of String         | true     |
| `searchableAttributes`   | Array of String         | true     |
| `filterableAttributes`   | Array of String         | true     |
| `sortableAttributes`     | Array of String         | true     |
| `rankingRules`           | Array of String         | true     |
| `stopWords`              | Array of String         | true     |
| `separatorTokens`        | Array of String         | true     |
| `nonSeparatorTokens`     | Array of String         | true     |
| `dictionary`             | Array of String         | true     |
| `synonyms`               | Object                  | true     |
| `distinctAttribute`      | String / `null`         | true     |
| `typoTolerance`          | Object                  | true     |
| `pagination`             | Object                  | true     |
| `faceting`               | Object                  | true     |
| `proximityPrecision`     | String / `null`         | true     |

The attributes ordering in the response payload is equivalent to the order described in the table above.

##### 3.2.1.2. Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.2.2. `PATCH` - `indexes/:index_uid/settings`

Modify the settings of a Meilisearch index.

##### 3.2.2.1. Request Payload Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `displayedAttributes`    | Array of String / `null` | false    |
| `searchableAttributes`   | Array of String / `null` | false    |
| `filterableAttributes`   | Array of String / `null` | false    |
| `sortableAttributes`     | Array of String / `null` | false    |
| `rankingRules`           | Array of String / `null` | false    |
| `stopWords`              | Array of String / `null` | false    |
| `separatorTokens`        | Array of String / `null` | false    |
| `nonSeparatorTokens`     | Array of String / `null` | false    |
| `dictionary`             | Array of String / `null` | false    |
| `synonyms`               | Object / `null`          | false    |
| `distinctAttribute`      | String / `null`          | false    |
| `typoTolerance`          | Object / `null`          | false    |
| `pagination`             | Object / `null`          | false    |
| `faceting`               | Object / `null`          | false    |
| `proximityPrecision`     | String / `null`          | false    |

- The request payload accepts partial definitions, the value of missing fields will remain unchanged.
- The request payload accepts the `null` value for any setting, which will reset the setting to its default value.

##### 3.2.2.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.2.2.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
 - 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.

Errors related to a sub-resource are described in its respective specification. See [3.1. Sub Resource Settings API List](#31-sub-resource-settings-api-list).

###### 3.2.2.3.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.2.2.2. Response Definition](#3222-response-definition).

> Otherwise, Meilisearch will create the index in a lazy way. See [3.2.2.4. Lazy Index Creation](#3224-lazy-index-creation).

##### 3.2.2.4. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.2.2.3.1. Async Errors](#32231-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.2.2.2. Response Definition](#3222-response-definition).

#### 3.2.3. `DELETE` - `indexes/:index_uid/settings`

Reset the settings of a Meilisearch index to the default values.

##### 3.2.3.1. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.2.3.2. Default Values

See [3.1. Sub Settings API Resource List](#31-sub-settings-api-resource-list) to get the default values of each setting.

##### 3.2.3.3. Errors

 - 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.

Errors related to a sub-resource are described in its respective specification. See [3.1. Sub Resource Settings API List](#31-sub-resource-settings-api-list).

###### 3.2.3.3.1. Asynchronous Index Not Found Error

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related async `task` resource. See [3.2.3.1. Response Definition](#3231-response-definition).

#### 3.2.4. General Errors

These errors apply to all endpoints described here.

##### 3.2.4.1 Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 4. Technical Details

### 4.1. Triggering Documents Re-indexing

Meilisearch favors search speed and makes a trade-off on indexing speed by computing internal data structures to get search results as fast as possible.

Changing any of the following index settings will cause documents to be re-indexed:

- `searchableAttributes`
- `filterableAttributes`
- `sortableAttributes`
- `distinctAttribute`
- `stopWords`
- `separatorTokens`
- `nonSeparatorTokens`
- `dictionary`
- `proximityPrecision`

## 5. Future Possibilities

n/a
