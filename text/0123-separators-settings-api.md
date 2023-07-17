# Stop Words Setting API

## 1. Summary

This specification describes the `separatorTokens` and `nonSeparatorTokens` index setting API endpoints.

## 2. Motivation
N/A

## 3. Functional Specification

### 3.1. Explanations

TBD

#### 3.1.1. Usage Examples

***Request payload `PUT`- `/indexes/articles/settings/separator-tokens`***
```json
["|", "/", "&sep"]
```

***Request payload `PUT`- `/indexes/articles/settings/non-separator-tokens`***
```json
["@", "#"]
```

### 3.2. Global Settings API Endpoints Definition

`separatorTokens` and `nonSeparatorTokens` are sub-resources of `/indexes/:index_uid/settings`.

See [Settings API](0123-settings-api.md).

### 3.3. API Endpoints Definition

Manipulate the `separatorTokens` and `nonSeparatorTokens` settings of a Meilisearch index.

#### 3.3.1. `GET` 

##### 3.3.1.1. `GET` - `indexes/:index_uid/settings/separator-tokens`

Fetch the `separatorTokens` setting of a Meilisearch index.

***Response Definition:***
- Type: Array of String
- Default: `[]`

##### 3.3.1.2. `GET` - `indexes/:index_uid/settings/non-separator-tokens`

Fetch the `nonSeparatorTokens` setting of a Meilisearch index.

***Response Definition:***
- Type: Array of String
- Default: `[]`

##### 3.3.1.3. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.3.2. `PUT`

##### 3.3.2.1. `PUT` - `indexes/:index_uid/settings/separator-tokens`

Modify the `separatorTokens` setting of a Meilisearch index.

###### 3.3.2.1.1. Request Payload Definition

- Type: Array of String / `null`

Setting `null` is equivalent to using the [3.3.3.1. `DELETE` - `indexes/:index_uid/settings/separator-tokens`](#3331-delete---indexesindexuidsettingsseparator-tokens) API endpoint.

###### 3.3.2.1.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.2.2. `PUT` - `indexes/:index_uid/settings/non-separator-tokens`

Modify the `nonSeparatorTokens` setting of a Meilisearch index.

###### 3.3.2.2.1. Request Payload Definition

- Type: Array of String / `null`

Setting `null` is equivalent to using the [3.3.3.2. `DELETE` - `indexes/:index_uid/settings/non-separator-tokens`](#3332-delete---indexesindexuidsettingsnon-separator-tokens) API endpoint.

###### 3.3.2.2.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.2.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a request payload value type different of `Array of String`, `[]`,  or `null` returns an [invalid_settings_stop_words](0061-error-format-and-definitions.md#invalid_settings_stop_words) error.

###### 3.3.2.3.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.3.2.2. Response Definition](#3222-response-definition).

> Otherwise, Meilisearch will create the index in a lazy way. See [3.2.2.4. Lazy Index Creation](#3224-lazy-index-creation).

##### 3.3.2.4. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.3.2.3.1. Async Errors](#33231-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.3.2.2. Response Definition](#3322-response-definition).

#### 3.3.3. `DELETE`
##### 3.3.3.1 `DELETE` - `indexes/:index_uid/settings/separator-tokens`

Reset the `separatorTokens` setting of a Meilisearch index to the default value `[]`.

###### 3.3.3.1.1. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.3.2 `DELETE` - `indexes/:index_uid/settings/non-separator-tokens`

Reset the `nonSeparatorTokens` setting of a Meilisearch index to the default value `[]`.

###### 3.3.3.2.1. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.3.3. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.

###### 3.3.3.3.1. Asynchronous Index Not Found Error

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related async `task` resource. See [3.3.3.1. Response Definition](#3331-response-definition).

#### 3.3.4. General Errors

These errors apply to all endpoints described here.

##### 3.3.4.1 Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 4. Technical Details

### 4.1. Triggering Documents Re-Indexing

Meilisearch favors search speed and makes a trade-off on indexing speed by computing internal data structures to get search results as fast as possible.

Modifying this index setting cause documents to be re-indexed.

## 5. Future Possibilities
n/a