# Tasks API

## 1. Functional Specification

### I. Summary

This specification describes the API endpoints for handling asynchronous tasks.

### II. Motivation

As writing is asynchronous for most of Meilisearch's operations, this API allows you to track the progress of asynchronous tasks, know and understand why a task has failed, and cancel specific tasks being enqueued or processing by consulting the history of operations that happened.

### III. Explanation

#### 1. `task` object definition

##### **Fully Qualified `task` object**

> This fully qualified version appears as a response object on `task` dedicated endpoints.

| field      | type    | description                                                                                                                                                                                                                   |
|------------|---------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| uid        | integer | Unique sequential identifier                                                                                                                                                                                                  |
| indexUid   | string  | Unique index identifier. This field is `null` when the task type is `dumpCreation`.                                                                                                                                                                                                                                        |
| status     | string  | Status of the task. Possible values are `enqueued`, `processing`, `succeeded`, `failed`, `canceled`                                                                                                                          |
| type       | string  | Type of the task. Possible values are `indexCreation`, `indexUpdate`, `indexDeletion`, `documentAdditionOrUpdate`, `documentDeletion`, `settingsUpdate`, `dumpCreation`, `taskCancelation`                                                    |
| details    | object  | Details information for a task payload. See Task Details part.                                                                                                                                                                |
| error      | object  | Error object containing error details and context when a task has a `failed` status. See [0061-error-format-and-definitions.md](0061-error-format-and-definitions.md)                                                         |
| duration   | string  | Total elapsed time the engine was in processing state expressed as an `ISO-8601` duration format. Times below the second can be expressed with the `.` notation, e.g., `PT0.5S` to express `500ms`. Default is set to `null`. |
| enqueuedAt | string  | Represent the date and time as `RFC 3339` format when the task has been enqueued                                                                                                                                              |
| startedAt  | string  | Represent the date and time as `RFC 3339` format when the task has been dequeued and started to be processed. Default is set to `null`                                                                                        |
| finishedAt | string  | Represent the date and time as `RFC 3339` format when the task has `failed` or `succeeded`. Default is set to `null`                                                                                                          |

> 💡 The order of the fields must be returned in this order.

##### Summarized `task` Object for `202 Accepted`

| field      | type    | description                     |
|------------|---------|---------------------------------|
| taskUid    | integer | Unique sequential identifier           |
| indexUid   | string  | Unique index identifier. This field is `null` when the task type is `dumpCreation`. |
| status     | string  | Status of the task. Value is `enqueued` |
| type       | string  | Type of the task. |
| enqueuedAt | string  | Represent the date and time as `RFC 3339` format when the task has been enqueued |


> 💡 The order of the fields must be returned in this order.
>
> 💡 This summarized version appears only in `202 Accepted` responses.

#### 2. `status` field enum

| label      |
|------------|
| enqueued   |
| processing |
| succeeded  |
| failed     |
| canceled   |

#### 3. `type` field enum

| label                    |
|--------------------------|
| indexCreation            |
| indexUpdate              |
| indexDeletion            |
| documentAdditionOrUpdate |
| documentDeletion         |
| settingsUpdate           |
| dumpCreation |
| taskCancelation          |

> 👍 Type values follow a `camelCase` naming convention.

#### 4. `details` field object

##### documentAdditionOrUpdate

| name              | description                          |
|-------------------|--------------------------------------|
| receivedDocuments | Number of documents received.        |
| indexedDocuments  | Number of documents finally indexed. |

##### documentDeletion

| name                | description                          |
|---------------------|--------------------------------------|
| receivedDocumentIds | Number of document ids received.     |
| deletedDocuments    | Number of documents finally deleted. |

##### indexCreation

| name       | description                                                                                                                                   |
|------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
| primaryKey | Value for the `primaryKey` field into the POST index payload. `null` if no `primaryKey` has been specified at the time of the index creation. |


##### indexUpdate

| name       | description                                                                                                                                |
|------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| primaryKey | Value for the `primaryKey` field into the PUT index payload. `null` if no `primaryKey` has been specified at the time of the index update. |

##### indexDeletion

| name             | description                                                                          |
|------------------|--------------------------------------------------------------------------------------|
| deletedDocuments | Number of deleted documents. Should be all documents contained in the deleted index. |

##### settingsUpdate

| name                 | description                          |
|----------------------|--------------------------------------|
| rankingRules         | `rankingRules` payload array         |
| searchableAttributes | `searchableAttributes` payload array |
| filterableAttributes | `filterableAttributes` payload array |
| sortableAttributes   | `sortableAttributes` payload array   |
| stopWords            | `stopWords` payload array            |
| synonyms             | `synonyms` payload object            |
| distinctAttribute    | `distrinctAttribute` payload string  |
| displayedAttributes  | `displayedAttributes` payload array  |
| typoTolerance        | `typoTolerance` payload object       |
| pagination           | `pagination` payload object          |
| faceting             | `faceting` payload object            |

##### dumpCreation

| name    | description  |
| -----   | ------------ |
| dumpUid | The generated uid of the dump |

Since the creation of a dump is not a task associated with a particular index, it is only present on the `GET` - `/tasks` and `GET` - `tasks/:task_uid` endpoints.

Fully qualified and summarized task objects related to a dump creation display a `null` `indexUid` field.

##### taskCancelation

| Name | Description |
| --- | --- |
| matchedTasks | The number of tasks that can be canceled based on the request. If the API key doesn’t have access to any of the indexes specified in the request, those tasks will not be included in matchedTasks.  |
| canceledTasks | The number of tasks successfully canceled. If the task fails, this will be 0. |
| originalQuery | The filter query used in the /tasks/cancel request. |

#### 5. Examples

e.g. A fully qualified `task` object in an `enqueued` state.

```json
{
    "uid": 0,
    "indexUid": "movies",
    "status": "enqueued",
    "type": "settingsUpdate",
    "details": {
        "rankingRules": [
            "typo",
            "ranking:desc",
            "words",
            "proximity",
            "attribute",
            "exactness"
        ]
    },
    "duration": null,
    "enqueuedAt": "2021-08-10T14:29:17.000000Z",
    "startedAt": null,
    "finishedAt": null
}
```

e.g. A fully qualified `task` object in a `processing` state.

```json
{
    "uid": 0,
    "indexUid": "movies",
    "status": "processing",
    "type": "settingsUpdate",
    "details": {
        "rankingRules": [
            "typo",
            "ranking:desc",
            "words",
            "proximity",
            "attribute",
            "exactness"
        ]
    },
    "duration": null,
    "enqueuedAt": "2021-08-10T14:29:17.000000Z",
    "startedAt": "2021-08-10T14:29:18.000000Z",
    "finishedAt": null
}
```

e.g. A fully qualified `task` object in a `succeeded` state.

```json
{
    "uid": 0,
    "indexUid": "movies",
    "status": "succeeded",
    "type": "settingsUpdate",
    "details": {
        "rankingRules": [
            "typo",
            "ranking:desc",
            "words",
            "proximity",
            "attribute",
            "exactness"
        ]
    },
    "duration": "PT1S",
    "enqueuedAt": "2021-08-10T14:29:17.000000Z",
    "startedAt": "2021-08-10T14:29:18.000000Z",
    "finishedAt": "2021-08-10T14:29:19.000000Z"
}
```

e.g. A fully qualified `task` object in a `failed` state.

```json
{
    "uid": 0,
    "indexUid": "movies",
    "status": "failed",
    "type": "settingsUpdate",
    "details": {
        "rankingRules": [
            "typo",
            "ranking:desc",
            "words",
            "proximity",
            "attribute",
            "wordsPosition",
            "exactness"
        ]
    },
    "error": {
        "message": "invalid criterion wordsPosition",
        "code": "internal",
        "type": "internal_error",
        "link": "https://docs.meilisearch.com/errors#internal",
    },
    "duration": "PT1S",
    "enqueuedAt": "2021-08-10T14:29:17.000000Z",
    "startedAt": "2021-08-10T14:29:18.000000Z",
    "finishedAt": "2021-08-10T14:29:19.000000Z"
}
```

e.g. A fully qualified `task` object in a `canceled` state.

```json
{
    "uid": 0,
    "indexUid": "movies",
    "status": "canceled",
    "type": "settingsUpdate",
    "details": {
        "rankingRules": [
            "typo",
            "ranking:desc",
            "words",
            "proximity",
            "attribute",
            "exactness"
        ]
    },
    "canceledBy": 1,
    "enqueuedAt": "2021-08-10T14:29:17.000000Z",
    "startedAt": "2021-08-10T14:29:18.000000Z",
    "finishedAt": "2021-08-10T14:29:19.000000Z"
}
e.g. A summarized `task` object in a `202 Accepted` HTTP response returned at index creation.

```json
{
    "taskUid": 0,
    "indexUid": "movies",
    "status": "enqueued",
    "type": "indexCreation",
    "enqueuedAt": "2021-08-11T09:25:53.000000Z"
}
```

---

#### 6. APIs endpoints

##### 6.1. Get all tasks | `GET` - `/tasks`

##### 6.1.1. Goals

Allows users to list tasks globally regardless of the indexes involved. Particularly useful to visualize all the tasks.

`200` - Response body - `/tasks`

```json
{
    "results": [
        {
            "uid": 1,
            "indexUid": "movies_reviews",
            "status": "enqueued",
            "type": "documentAdditionOrUpdate",
            "duration": null,
            "enqueuedAt": "2021-08-12T10:00:00.000000Z",
            "startedProcessingAt": null,
            "finishedAt": null
        },
        {
            "uid": 0,
            "indexUid": "movies",
            "status": "succeeded",
            "type": "documentAdditionOrUpdate",
            "details": {
                "receivedDocuments": 100,
                "indexedDocuments": 100
            },
            "duration": "PT16S",
            "enqueuedAt": "2021-08-11T09:25:53.000000Z",
            "startedAt": "2021-08-11T10:03:00.000000Z",
            "finishedAt": "2021-08-11T10:03:16.000000Z"
        }
    ]
}
```

##### 6.1.2. Requirements

> 💡 `task` objects are contained in a `results` array.
>
> 💡 `task` uid is generated globally. The `uid` of the tasks are no longer scoped to an index.
>
> 💡 By default, objects are sorted by `desc` order on `uid` field. So the most recent tasks appear first.
>
> 💡 When an index is deleted, its tasks remain accessible on the global `/tasks` endpoint.

##### 6.1.3. Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have the required permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

---

##### 6.2. Get a task by uid | `GET` - `/tasks/{uid}`

##### 6.2.1. Goals

Allows users to get a detailed `task` object retrieved by the `uid` field regardless of the index involved.

`200` - Response body -  `/tasks/1`

```json
{
    "uid": 1,
    "indexUid": "movies",
    "status": "enqueued",
    "type": "documentAdditionOrUpdate",
    "duration": null,
    "enqueuedAt": "2021-08-12T10:00:00.000000Z",
    "startedAt": null,
    "finishedAt": null
}
```

##### 6.2.2. Errors

- 🔴 If the task does not exist, the API returns a `404 Not Found` - `task_not_found` error.

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have the required permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

---

##### 6.3. Cancel tasks | `POST` - `/tasks/cancel`

##### 6.3.1. Goals

Allows users to cancel an `enqueued` or `processing` task. Particularly useful if a long or heavy task blocks the queue.

`202` - Response body - `/tasks/cancel`

```json
{
    "taskUid": 0,
    "indexUid": "movies",
    "status": "enqueued",
    "type": "taskCancelation",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

##### 6.3.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code 202 Accepted. The response's content is the summarized representation of the received asynchronous task.

##### 6.3.3. Errors

If a user tries canceling a `succeeded`, `failed`, or `canceled` task, it won’t throw an error. Task cancelation is an atomic transaction; all tasks are successfully canceled, or none aren't.

- 🔴 Sending a task cancelation without filtering query parameters returns a `missing_filters` error.
    
    ```json
    {
        "message": "Query parameters to filter the tasks to cancel are missing. Available query parameters are: `uid`, `indexUid`, `status`, `type`, `beforeEnqueuedAt`, `afterEnqueudAt`, `beforeStartedAt`, afterStartedAt`",
        "code": "missing_filters",
        "type": "invalid_request",
        "link": "https://docs.meilisearch.com/errors#missing_filters"
    }
    ```

- 🔴 If the `type` parameter value is not consistent with one of the task types, an `[invalid_task_type](https://github.com/meilisearch/specifications/blob/main/text/0061-error-format-and-definitions.md#invalidtasktype)` error is returned.
- 🔴 If the `status` parameter value is not consistent with one of the task statuses, an `[invalid_task_status](https://github.com/meilisearch/specifications/blob/main/text/0061-error-format-and-definitions.md#invalidtaskstatus)` error is returned.

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have the required permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

---

#### 7. `task_not_found` error

##### Context

This error happens when a requested task can't be found.

##### Error Definition

HTTP Code: `404 Not Found`

```json
{
    "message": "Task :taskUid not found.",
    "code": "task_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#task_not_found"
}
```

- The `:taskUid` is inferred when the message is generated.

#### 8. `MEILI_MAX_TASK_DB_SIZE` env var and `--max-task-db-size` CLI option

See [0119-instance-options](0119-instance-options.md##3312-max-taskdb-size)

#### 9. Asynchronous Write Operations on Index resource

- 💡 Automatic index creation using the `/indexes/:indexToCreate/documents` route generates a `documentAdditionOrUpdate` task that also handles index creation.

#### 10. Paginate `task` resource lists

The API endpoint `/tasks` is browsable using a keyset-based pagination.

##### 10.1. Why a Seek/Keyset based pagination?

Keyset-based pagination is more appropriate when the data can grow or shrink quickly in terms of magnitude.

###### 10.1.1. Pros

The performance is better than the not-so-good but old pagination with `offset`/`limit`.

Seek/Keyset pagination keeps the results consistent between each page as the data evolves. It avoids the [Page Drift effect](https://use-the-index-luke.com/sql/partial-results/fetch-next-page), especially when the data is sorted from the most recent to the oldest.

Moreover, the performance is superior to traditional pagination since the computational complexity remains constant to reach the identifier marking the beginning of the new slice to be returned from a hash table.

###### 10.1.2. Cons

The main drawback of this type of pagination is that it does not navigate within a finite number of pages. It is also limited to a precise sorting criterion on unique identifiers ordered sequentially.

##### 10.2. Response attributes

| field | type | description                          |
|-------|------|--------------------------------------|
| limit | integer  | Default `20`. |
| from | integer | The first task uid returned |
| next | integer - nullable  | Represents the value to send in `from` to fetch the next slice of the results. The first item for the next slice starts at this exact number. When the returned value is null, it means that all the data have been browsed in the given order. |

##### 10.3. GET query parameters

| field | type | required | description  |
|-------|------|----------|--------------|
| limit | integer  | No       | Default `20`. Limit on the number of tasks to be returned. |
| from | integer  | No       | Limit results to tasks with uids equal to and lower than this uid. |

##### 10.4. Usage examples

This part demonstrates keyset paging in action on `/tasks`. The items `uid` remains sorted sequentially and can be used to navigate a list of `tasks` objects.

---

**Initial default slice of `tasks`**

`GET` - `/tasks`

```json
{
    "results": [
        {
            "uid": 1350,
            "indexUid": "movies",
            "type": "documentAdditionOrUpdate",
            ...,
        },
        ...,
        {
            "uid": 1330,
            "indexUid": "movies_reviews",
            "type": "documentAdditionOrUpdate",
            ...,
        }
    ],
    "limit": 20,
    "from": 1350,
    "next": 1329
}
```

**Request the next slice of `tasks` items with a limit of `50` tasks**

`GET` - `/tasks?from=1329&limit=50`

```json
{
    "results": [
        {
            "uid": 1329,
            "indexUid": "movies",
            "type": "documentAdditionOrUpdate",
            ...,
        },
        ...,
        {
            "uid": 1279,
            "indexUid": "movies",
            "type": "settingsUpdate",
            ...,
        }
    ],
    "limit": 50,
    "from": 1329,
    "next": 1278
}
```

**End of seek/keyset pagination**

`GET` - `/tasks?from=20`

```json
{
    "results": [
        {
            "uid": 19,
            "indexUid": "movies",
            "type": "documentsAdditionOrUdpdate",
            ...,
        },
        ...,
        {
            "uid": 0,
            "indexUid": "movies",
            "type": "documentsAdditionOrUpdate",
            ...,
        }
    ],
    "limit": 20,
    "from": 20,
    "next": null
}
```

- 💡 `next` response parameter is null because there are no more `tasks` to fetch. It means that the response represents the last slice of results for the given resource list.

##### 10.5. Behaviors for `limit` and `from` query parameters

###### 10.5.1. `limit`

- If `limit` is not set, the default value is chosen.

###### 10.5.2. `from`

- If `from` is set with an out of bounds task `uid`, the response returns the tasks that are the nearest to the specified uid, the `next` field is set to the next page. It will be equivalent to call the `/tasks` route without any parameter.

###### 10.5.3. Errors

- 🔴 Sending a value with a different type than `Integer` for `limit` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending a value with a different type than `Integer` for `from` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 11. Filtering task resources

##### 11.1. Filtering task to get

The `/tasks` endpoint is filterable by `indexUid`, `type` and `status` query parameters.

##### 11.1.1 Query parameters definition

| parameter | type   | required | description                                                                                                                                                                                                                             |
|-----------|--------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| uid  | string | No       | Permits to filter tasks by their related unique identifier. By default, when `uid` query parameter is not set, all the tasks are concerned. It is possible to specify several uid by separating them with the `,` character. |
| indexUid  | string | No       | Permits to filter tasks by their related index. By default, when `indexUid` query parameter is not set, the tasks of all the indexes are concerned. It is possible to specify several indexes by separating them with the `,` character. |
| status    | string | No       | Permits to filter tasks by their status. By default, when `status` query parameter is not set, all task statuses are concerned. It's possible to specify several types by separating them with the `,` character.                        |
| type      | string | No       | Permits to filter tasks by their related type. By default, when `type` query parameter is not set, all task types are concerned. It's possible to specify several types by separating them with the `,` character.                       |
| date | string | No       | Permits to filter tasks by their started or launched time. By default, when `afterXAt` or `beforeXAt` query parameter is not set, all task types are concerned. It's possible to specify several types by separating them with the `,` character.                       |

##### 11.1.2. Usages examples

This part demonstrates filtering on `/tasks`.

---

**No filtering**

`GET` - `/tasks`

```json
{
    "results": [
        {
            "uid": 1350,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentAdditionOrUpdate",
            ...,
        },
        ...,
        {
            "uid": 1330,
            "indexUid": "movies_reviews",
            "status": "succeeded",
            "type": "documentDeletion",
            ...
        }
    ],
    ...
}
```

**Filter `tasks` that have a `failed` `status`**

`GET` - `/tasks?status=failed`

```json
{
    "results": [
        {
            "uid": 1350,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentAdditionOrUpdate",
            ...,
        },
        ...,
        {
            "uid": 1279,
            "indexUid": "movies",
            "status": "failed",
            "type": "settingsUpdate",
            ...,
        }
    ],
    ...
}
```

**Filter `tasks` that are of `documentAdditionOrUpdate` type**

`GET` - `/tasks?type=documentAdditionOrUpdate`

```json
{
    "results": [
        {
            "uid": 1350,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentAdditionOrUpdate",
            ...,
        },
        ...,
        {
            "uid": 1343,
            "indexUid": "movies",
            "type": "succeeded",
            "type": "documentAdditionOrUpdate",
            ...,
        }
    ],
    ...
}
```

**Filter `canceled` tasks by `canceledBy` parameter**

`GET` `tasks?canceledBy=1`

```json
{
    "results": [
        {
            "uid": 1350,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentAdditionOrUpdate",
            "canceledBy": 1,
            ...,
        },
        ...,
        {
            "uid": 1343,
            "indexUid": "movies",
            "type": "succeeded",
            "type": "documentAdditionOrUpdate",
            "canceledBy": 1,
            ...,
        }
    ],
    ...
}
```

**Filter `tasks` that are of `documentAdditionOrUpdate` type and have a `failed` status**

`GET` - `/tasks?type=documentAdditionOrUpdate&status=failed`

```json
{
    "results": [
        {
            "uid": 1350,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentAdditionOrUpdate",
            ...,
        },
        ...,
        {
            "uid": 1346,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentAdditionOrUpdate",
            ...,
        }
    ],
    ...
}
```

- 💡 Filters can be used together. The two parameters are cumulated and a `AND` operation is performed between the two filters. An OR operation between filters is not supported.
- `type` and `status` query parameters can be read as is `type=documentsAdditionOrUpdate AND status=failed`.

**Filter `tasks` by an non-existent `indexUid`**

`GET` - `/tasks?indexUid=aaaaa`

```json
{
    "results": [],
    ...
}
```

- If the `indexUid` parameter value contains an inexistent index, it returns an empty `results` array.

---

##### 11.1.3. Behaviors for `indexUid`, `status`and `type` query parameters.

###### 11.1.3.1. `indexUid`

- Type: String
- Required: False
- Default: `*`

`indexUid` is **case-sensitive**.

###### 11.1.3.2. `status`

- Type: String
- Required: False
- Default: `*`

- 🔴 If the `status` parameter value is not consistent with one of the task statuses, an [`invalid_task_status`](0061-error-format-and-definitions.md#invalidtaskstatus) error is returned.

###### 11.1.3.3. `type`

- Type: String
- Required: False
- Default: `*`

`type` is **case-insensitive**.

- 🔴 If the `type` parameter value is not consistent with one of the task types, an [`invalid_task_type`](0061-error-format-and-definitions.md#invalidtasktype) error is returned.

###### 11.1.3.4. `date`

You can cancel tasks using `afterXAt` and `beforeXAt`. You can use the following fields:

- `enqueuedAt` → `afterenqueuedAt` or `beforeanqueuedAt`
- `startedAt` → `afterstartedAt` or `beforestartedAt`
- `finishedAt` → `afterFinishedAt` or `beforeFinishedAt`

The filter accepts the RFC 3339 format. The following syntaxes are valid:

- `Y-M-D`
- `Y-M-DTH:M:SZ`
- `Y-M-DTH:M:S+01:00`

- Type: String
- Required: False
- Default: `*`

- 🔴 The date filters are exclusive. You can cancel tasks before or after a specified date, meaning it will not be included, `>=` and `<=` are not valid.

###### 11.1.3.5. Select multiple values for the same filter

It is possible to specify multiple values for a filter using the `,` character.

For example, to select the `enqueued` and `processing` tasks of the `movies` and `movie_reviews` indexes, it is possible to express it like this: `/tasks?indexUid=movies,movie_reviews&status=enqueued,processing`

##### 11.1.4. Empty `results`

If no results match the filters. A response is returned with an empty `results` array.

##### 11.2. Filtering task to cancel

The `/tasks/cancel` endpoint is filterable by `uid`, `indexUid`, `type` and `status` query parameters.

##### 11.2.1 Query parameters definition

| parameter | type   | required | description                                                                                                                                                                                                                             |
|-----------|--------|----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| uid  | string | No       | Permits to filter tasks by their related unique identifier. By default, when `uid` query parameter is not set, all the tasks are concerned. It is possible to specify several uid by separating them with the `,` character. |
| indexUid  | string | No       | Permits to filter tasks by their related index. By default, when `indexUid` query parameter is not set, the tasks of all the indexes are concerned. It is possible to specify several indexes by separating them with the `,` character. |
| status    | string | No       | Permits to filter tasks by their status. By default, when `status` query parameter is not set, all task statuses are concerned. It's possible to specify several types by separating them with the `,` character.                        |
| type      | string | No       | Permits to filter tasks by their related type. By default, when `type` query parameter is not set, all task types are concerned. It's possible to specify several types by separating them with the `,` character.                       |
| date | string | No       | Permits to filter tasks by their started or launched time. By default, when `afterXAt` or `beforeXAt` query parameter is not set, all task types are concerned. It's possible to specify several types by separating them with the `,` character.                       |

##### 11.2.2. Usages examples

This part demonstrates filtering on `/tasks/cancel`.

---

**No filtering**

Users will not be allowed to use this route without any filters (POST /tasks/cancel), as it may result in canceling everything by mistake. If the request contains no filters, the user will get an error asking them to be more specific.

**Filter `tasks` to cancel based on `uid`**

`POST` `/tasks/cancel?uid=1,2`

```json
{
    "taskUid": 3,
    "indexUid": null,
    "status": "enqueued",
    "type": "taskCancelation",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

**Filter `tasks` to cancel based on `indexUid`**

`POST` `/tasks/cancel?indexUid=movies`

```json
{
    "taskUid": 1,
    "indexUid": null,
    "status": "enqueued",
    "type": "taskCancelation",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

**Filter `tasks` to cancel based on `status`**

`POST` - `/tasks/cancel?status=processing`

```json
{
    "taskUid": 1,
    "indexUid": null,
    "status": "enqueued",
    "type": "taskCancelation",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

**Filter `tasks` to cancel based on `type`**

`POST` - `/tasks/cancel?type=documentAdditionOrUpdate`

```json
{
    "taskUid": 1,
    "indexUid": null,
    "status": "enqueued",
    "type": "taskCancelation",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

**Filter `tasks` to cancel using date filters**

`POST` - `/tasks/cancel?afterenqueuedAt>2020-10-11T11:49:53.000Z`

```json
{
    "taskUid": 1,
    "indexUid": null,
    "status": "enqueued",
    "type": "taskCancelation",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```
**Filter `tasks` to cancel that are of multiple type and have a `indexUid`**

`POST` - `/tasks/cancel?indexUid=movies&(AND)type=documentAdditionOrUpdate,(OR)documentDeletion`

```json
{
    "taskUid": 1,
    "indexUid": null,
    "status": "enqueued",
    "type": "taskCancelation",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

- 💡 Filters can be used together. Both parameters are accumulated and `AND` and `OR` operations are performed between the filters.
- `type` and `indexUid` query parameters can be read as is `type=documentsAdditionOrUpdate OR documentDeletion AND indexUid=movies`.

**Cancel all the tasks with filter**

`POST` - `/taskscancel?status=processing,enqueued`

```json
{
    "taskUid": 1,
    "indexUid": null,
    "status": "enqueued",
    "type": "taskCancelation",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

---

##### 11.2.3. Behaviors for `uid`, `indexUid`, `status`, `type` and `date` query parameters.

###### 11.2.3.1. `uid`

- Type: String
- Required: False
- Default: `*`

`uid` is **case-sensitive**.

###### 11.2.3.2. `indexUid`

- Type: String
- Required: False
- Default: `*`

`indexUid` is **case-sensitive**.

###### 11.2.3.3. `status`

- Type: String
- Required: False
- Default: `*`

- 🔴 If the `status` parameter value is not consistent with one of the task statuses, an [`invalid_task_status`](0061-error-format-and-definitions.md#invalidtaskstatus) error is returned.

###### 11.2.3.4. `type`

- Type: String
- Required: False
- Default: `*`

`type` is **case-insensitive**.

- 🔴 If the `type` parameter value is not consistent with one of the task types, an [`invalid_task_type`](0061-error-format-and-definitions.md#invalidtasktype) error is returned.

###### 11.2.3.5. `date`

You can cancel tasks using `afterXAt` and `beforeXAt`. You can use the following fields:

- `enqueuedAt` → `afterenqueuedAt` or `beforeanqueuedAt`
- `startedAt` → `afterstartedAt` or `beforestartedAt`

The filter accepts the RFC 3339 format. The following syntaxes are valid:

- `Y-M-D`
- `Y-M-DTH:M:SZ`
- `Y-M-DTH:M:S+01:00`

- Type: String
- Required: False
- Default: `*`

- 🔴 The date filters are exclusive. You can cancel tasks before or after a specified date, meaning it will not be included, `>=` and `<=` are not valid.

###### 11.2.3.6. Select multiple values for the same filter

It is possible to specify multiple values for a filter using the `,` character.

For example, to select the `enqueued` and `processing` tasks of the `movies` and `movie_reviews` indexes, it is possible to express it like this: `/tasks/cancel?indexUid=movies,movie_reviews&status=enqueued,processing`.

## 2. Technical details

n/a

## 3. Future Possibilities

- Use Hateoas capability to give direct access to a `task` resource.
- Add dedicated task type names modifying a sub-setting. e.g. `SearchableAttributesUpdate`.
- Add an archived state for old `tasks`.
- Add the `API Key` identity that added a `task`.