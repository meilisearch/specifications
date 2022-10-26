# Swap Indexes API

## 1. Summary

The swap indexes API allows to atomically deploy several new versions of indexes without any downtime for the search clients.

## 2. Motivation

It's critical to deploy a new version of an index without any downtimes to the search clients. This capability improves the development experience by allowing Meilisearch to better fit into their workflow.

## 3. Functional Specification

### 3.1. 0 dowtime deployment workflow

A user has big changes to make on an index. It could be important document schema or index settings changes.

The zero-downtimes deployment looks like this:

1. Search clients search on `indexA`.
2. The developer builds a new index `indexB` representing the new index version to deploy to the search clients.
3. When `indexB` is built and ready to be deployed, the developer sends an indexes swap request to Meilisearch for `indexA` and `indexB`.
4. `indexB` documents, settings and tasks are swapped with `indexA`.
5. Search clients search on the updated `indexA` without experiencing any downtime.

### 3.2. Deploying Multiple New Indexes Versions

The swap API supports multiple swap operations in an atomic fashion.

This means that for a search experience built using multiple indexes, Meilisearch is able to deploy all changes at once and thus clients will access the new version of all indexes at once without any downtime.

There is no need to deploy each new version of indexes one by one.

### 3.3. Enqueud Tasks After A Swap Operation Creation

Tasks enqueued after an `indexSwap` task creation date do not have their `indexUid` modified when the `indexSwap` will succeed. That is, if they are enqueued on `indexA`, they will run on the new version of `indexA`.

### 3.4. API Endpoints Definition

#### 3.4.1. `POST` - `/swap-indexes`

Send one or many indexes swap operation at once.

##### 3.4.1.1. Payload definition

The payload body expects an array of JSON objects representing swap operations.

```json
[
    {
        "indexes": ["indexA", "indexA_new"]
    },
    {
        "indexes": ["indexB", "indexB_new"]
    }
]
```

💡 In the given example, two swap operations will occurs atomically.

`indexA` data will be swapped with `indexA_new` data while `indexB` data will be swapped with `indexB_new` data.

> Sending `[]` is considered valid. No swap transactions will be performed.

###### Swap Object Definition

| Field                                            | Type                                  | Required |
|--------------------------------------------------|---------------------------------------|----------|
| [indexes](#33111-indexes)                        | Array of string representing indexUids| True     |

###### 3.4.1.1.1. `indexes`

- Type: Array of string
- Required: True

Determines which two indexes should exchange their data for their given swap object.

##### 3.4.1.2. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

```json
{
    "taskUid": 1,
    "indexUid": null,
    "status": "enqueued",
    "type": "indexSwap",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

> An `indexSwap` task is considered a global task; thus `indexUid` is null.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.4.1.3. Errors

###### 3.4.1.3.1. Synchronous Errors

- 🔴 If the instance is secured by a master key, accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 If the instance is secured by a master key, accessing this route with a key that does not have permissions (missing `indexes.swap` action or having a value for the `indexes` field of a swap operation not being defined in the API Key `indexes` array) (i.e. other than the master key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.
- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an `indexes` array not containing **exactly** 2 indexUids for a swap operation object returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

###### 3.4.1.3.2. Asynchronous Errors

- 🔴 Sending indexUids that do not exist within the `indexes` field of a swap operation returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.
- 🔴 Sending an indexUid more than once in the request payload returns a [duplicate_index_found](0061-error-format-and-definitions.md#duplicate_index_found) error.

## 4. Technical Details

### 4.1. Swapping Data

When indexes are swapped their data is exchanged. It concerns:

- The documents
- The settings
- The tasks history
    - `indexUid` field of tasks having been created before the `indexSwap` task `created_at` are updated to keep a practicable history. That is, it's possible to know why an index is in a given point in time state even if it has been swapped because it keeps the task history.

## 5. Future Possibilities

- Introduce a way to delete one of the swapped indexes when the swap operation occurs.