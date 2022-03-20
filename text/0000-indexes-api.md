# Indexes API

## 1. Summary

This specification describes the indexes API endpoints permitting to list, create, update and delete Meilsearch indexes.

Indexes contain a set of documents to be searchable and have their specific settings.

## 2. Motivation
N/A

## 3. Functional Specification

### 3.1. API Endpoints Definition

Manipulate indexes of a Meilisearch instance.

- [3.1.1. `GET` - `indexes`](#311-get---indexes)
- [3.1.2. `GET` - `indexes/:index_uid`](#312-get---indexesindexuid)
- [3.1.3. `POST` - `indexes`](#313-post---indexes)
- [3.1.4. `PUT` - `indexes/:index_uid`](#314-put---indexesindexuid)
- [3.1.5. `DELETE` - `indexes/:index_uid`](#315-delete---indexesindexuid)

#### 3.1.1. `GET` - `indexes`

List all indexes of a Meilisearch instance.

##### 3.1.1.1. Response Definition

Array made of index API resource object. See [3.1.2.1. Response Definition](#3121-response-definition) section.

#### 3.1.2. `GET` - `indexes/:index_uid`

Fetch an index of a Meilisearch instance.

##### 3.1.2.1. Response Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `uid`                    | String                   | true     |
| `name`                   | String                   | true     |
| `primaryKey`             | String / `null`          | true     |
| `createdAt`              | String                   | true     |
| `updatedAt`              | String                   | true     |

The `name` field takes the value of given `uid` at creation/update.

##### 3.1.2.2. Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.1.3. `POST` - `indexes`

Create an index.

##### 3.1.3.1. Request Payload Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `uid`                    | String                   | true     |
| `primaryKey`             | String / `null`          | false    |

##### 3.1.3.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.1.3.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending a value with a different type than `String` for `uid` will return a [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending an invalid `uid` returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a value with a different type than `String` or `null` for `primaryKey` will return a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

###### 3.1.3.3.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.2.2.2. Response Definition](#3222-response-definition).
- 🔴 Sending a `uid` that already exists returns an [index_already_exists](0061-error-format-and-definitions.md#index_already_exists) error.

#### 3.1.4. `PUT` - `indexes/:index_uid`

Update an index.

The `primaryKey` field can be updated when the index is empty. When the `primaryKey` is set to `null`, the indexing process will try to auto-infer the `primaryKey` by searching the first attribute containg `id` in the document payload to index.

##### 3.1.4.1. Request Payload Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `primaryKey`             | String / `null`          | false    |

##### 3.1.4.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.1.4.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.

###### 3.1.4.3.1. Async Errors

- 🔴 When updating the `primaryKey`, if the previous `primaryKey` value has already been used for a document, the API returns an [index_primary_key_already_exists](0061-error-format-and-definitions.md#index_primary_key_already_exists) error.
- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.1.5. `DELETE` - `indexes/:index_uid`

Delete an index.

##### 3.1.4.1. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.1.4.2. Errors

###### 3.1.4.2.1 Async Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.1.6. General Errors

These errors apply to all endpoints described here.

##### 3.1.6.1 Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 4. Technical Details
N/A

## 5. Future Possibilities
N/A