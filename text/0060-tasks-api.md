- Title: Tasks API
- Start Date: 2021-08-13
- Specification PR: [#60](https://github.com/meilisearch/specifications/pull/60)
- Discovery Issue: [#48](https://github.com/meilisearch/product/issues/48)

# Refashion Updates APIs

## 1. Functional Specification

### I. Summary

The term `update` is not the best choice as it can be confused with document updates or `settings`. We have chosen to replace this term with `task`, which is more generic and better reflects the meaning of this API resource.

As an additional change, we have reworked the format of an update to make it more in line with our expectations of an API that is supposed to be easily understandable and developer-oriented.

Changing the format of `task` object lists makes adding more functionality in a future iteration easier. See the Future Possibilities section for a brief overview.

The specification adds two new API endpoints. Although quite simple, they allow consulting the list of tasks or a specific task without being forced to know the related index.

The specification makes any writing operation on an index asynchronous to serve consistency and facilitate future evolutions.

#### Summary Key Points

- The `update` resource is renamed `task`. The names of existing API routes are also changed to reflect this change.
- Tasks are now also accessible as an independent resource of an index. `GET - /tasks`; `GET - /tasks/:taskUid`
- The task `uid` is not incremented by index anymore. The sequence is generated globally.
- A `task_not_found` error is introduced.
- The format of the `task` object is updated.
    - `updateId` becomes `uid`.
    - Attributes of an error appearing in a `failed` `task` are now contained in a dedicated `error` object.
    - `type` is no longer an object. It now becomes a string containing the values of its `name` field previously defined in the `type` object.
    - The possible values for the `type` field are reworked to be more clear and consistent with our naming rules.
    - A `details` object is added to contain specific information related to a `task` payload that was previously displayed in the `type` nested object.
    - An `indexUid` field is added to give information about the related index on which the task is performed.
    - `duration` format has been updated to express an `ISO 8601` duration.
    - `processed` status changes to `succeeded`.
    - `startedProcessingAt` is updated to `startedAt`.
    - `processedAt` is updated to `finishedAt`.
- `202 Accepted` requests previously returning an `updateId` are now returning a summarized `task` object.
- `MEILI_MAX_UDB_SIZE` env var is updated `MEILI_MAX_TASK_DB_SIZE`.
- `--max-udb-size` cli option is updated to `--max-task-db-size`.
- `task` object lists are now returned under a `results` array.
- Each operation on an index (creation, update, deletion) is now asynchronous and represented by a `task`.


### II. Motivation

The motivation is to stabilize the current `update` resource to a version that conforms to our API convention, thus developing future evolutions on a more solid base. We want to modify the name `update`, the format is also changed because some attributes are not immediately explicit, either in the possible values or in the chosen names.

### III. Explanation

#### 1. `task` object definition

##### **Fully Qualified `task` object**

> This fully qualified version appears as a response object on `task` dedicated endpoints.

| field   | type    | description                     |
|---------|---------|---------------------------------|
| uid      | integer | Unique sequential identifier           |
| indexUid | string | Unique index identifier |
| batchUid | integer | Identify in which batch a task has been grouped by auto-batching. It corresponds to the first task uid grouped within a batch. See [auto-batching specification](0096-auto-batching.md) |
| status  | string  | Status of the task. Possible values are `enqueued`, `processing`, `succeeded`, `failed`                                |
| type    | string  | Type of the task. Possible values are `indexCreation`, `indexUpdate`, `indexDeletion`, `documentAddition`, `documentPartial`, `documentDeletion`, `settingsUpdate`, `clearAll` |
| details | object |  Details information for a task payload. See Task Details part. |
| error | object | Error object containing error details and context when a task has a `failed` status. See https://github.com/meilisearch/specifications/pull/61|
 | duration | string | Total elapsed time the engine was in processing state expressed as an `ISO-8601` duration format. Times below the second can be expressed with the `.` notation, e.g., `PT0.5S` to express `500ms`. Default is set to `null`.   |
| enqueuedAt | string | Represent the date and time as `RFC 3339` format when the task has been enqueued |
| startedAt | string | Represent the date and time as `RFC 3339` format when the task has been dequeued and started to be processed. Default is set to `null`|
| finishedAt | string | Represent the date and time as `RFC 3339` format when the task has `failed` or `succeeded`. Default is set to `null` |

> 💡 The order of the fields must be returned in this order.

##### Summarized `task` Object for `202 Accepted`

| field      | type    | description                     |
|------------|---------|---------------------------------|
| uid        | integer | Unique sequential identifier           |
| indexUid   | string | Unique index identifier |
| status     | string  | Status of the task. Value is `enqueued` |
| enqueuedAt | string | Represent the date and time as `RFC 3339` format when the task has been enqueued |


> 💡 The order of the fields must be returned in this order.
>
> 💡 This summarized version appears only in `202 Accepted` responses.

#### 2. `status` field enum

| old        | new           |
|------------|---------------|
| enqueued   | -             |
| processing | -             |
| processed  | **succeeded** |
| failed     | -             |

> 👍 Better semantic differentiation between `processing` and `processed`. The final status of a *processed* task is `succeeded` or `failed`.

#### 3. `type` field enum

| old        | new           |
|------------|---------------|
|  -         | indexCreation |
|  -         | indexUpdate   |
|  -         | indexDeletion |
| DocumentsAddition | documentAddition  |
| DocumentsPartial | documentPartial  |
| DocumentsDeletion  | documentDeletion |
| Settings     | settingsUpdate |
| ClearAll | clearAll |

> 👍 Type values follow a `camelCase` naming convention.
>
> 💡 `Settings` is updated to be more precise with the name `settingsUpdate`.

#### 4. `details` field object

##### documentAddition

| name     | description |
| -------- | --------    |
| receivedDocuments | Number of documents received. |
| indexedDocuments  | Number of documents finally indexed. |


##### documentPartial

| name     | description |
| -------- | --------    |
| receivedDocuments | Number of documents received. |
| indexedDocuments  | Number of documents finally indexed. |


##### documentDeletion

| name     | description |
| -------- | --------    |
| receivedDocumentIds | Number of document ids received.  |
| deletedDocuments    | Number of documents finally deleted. |

##### indexCreation

| name     | description |
| -------- | --------    |
| primaryKey  | Value for the `primaryKey` field into the POST index payload. `null` if no `primaryKey` has been specified at the time of the index creation. |


##### indexUpdate

| name     | description |
| -------- | --------    |
| primaryKey | Value for the `primaryKey` field into the PUT index payload. `null` if no `primaryKey` has been specified at the time of the index update. |

##### indexDeletion

| name     | description |
| -------- | --------    |
| deletedDocuments    | Number of deleted documents. Should be all documents contained in the deleted index. |

##### clearAll

| name     | description |
| -------- | --------    |
| deletedDocuments    | Number of deleted documents. Should be all documents contained in the cleared index. |

##### settingsUpdate

| name     | description |
| -------- | --------    |
| rankingRules     | `rankingRules` payload array |
| searchableAttributes | `searchableAttributes` payload array |
| filterableAttributes | `filterableAttributes` payload array |
| sortableAttributes | `sortableAttributes` payload array | 
| stopWords | `stopWords` payload array |
| synonyms  | `synonyms` payload object |
| distinctAttribute | `distrinctAttribute` payload string |
| displayedAttributes | `displayedAttributes` payload array |

#### 5. Examples

e.g. A fully qualified `task` object in an `enqueued` state.

```json
{
    "uid": 0,
    "indexUid": "movies",
    "batchUid": 0,
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
    "batchUid": 0,
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
    "batchUid": 0,
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
    "batchUid": 0,
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

e.g. A summarized `task` object in a `202 Accepted` HTTP response returned at index creation.

```json
{
    "uid": 0,
    "indexUid": "movies",
    "status": "enqueued",
    "type": "createIndex",
    "enqueuedAt": "2021-08-11T09:25:53.000000Z"
}
```

---

#### 6. APIs endpoints

**Get all tasks** | `GET` - `/tasks`

##### Goals

Allows users to list tasks globally regardless of the indexes involved. Particularly useful to visualize all the tasks.

`200` - Response body - `/tasks`

```json
{
    "results": [
        {
            "uid": 1,
            "indexUid": "movies_reviews",
            "batchUid": 1,
            "status": "enqueued",
            "type": "documentAddition",
            "duration": null,
            "enqueuedAt": "2021-08-12T10:00:00.000000Z",
            "startedProcessingAt": null,
            "finishedAt": null
        },
        {
            "uid": 0,
            "indexUid": "movies",
            "batchUid": 0,
            "status": "succeeded",
            "type": "documentAddition",
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

##### Requirements

> 💡 `task` objects are contained in a `results` array.
>
> 💡 `task` uid is generated globally. The `uid` of the tasks are no longer scoped to an index.
>
> 💡 By default, objects are sorted by `desc` order on `uid` field. So the most recent tasks appear first.
>
> 💡 When an index is deleted, its tasks remain accessible on the global `/tasks` endpoint.

##### Errors

- 🔴 If a master key is configured on the server-side but missing from the client in the `X-MEILI-API-KEY` header, the API returns a `401 Unauthorized` - `missing_authorization_header` error.
- 🔴 If a master key is sent by the client but does not match the value configured on the server-side, the API returns a `403 Forbidden` - `invalid_api_key`.

---

**Get a task by uid** | `GET` - `/tasks/{uid}`

##### Goals

Allows users to get a detailed `task` object retrieved by the `uid` field regardless of the index involved.

`200` - Response body -  `/tasks/1`

```json
{
    "uid": 1,
    "indexUid": "movies",
    "batchUid": 1,
    "status": "enqueued",
    "type": "documentAddition",
    "duration": null,
    "enqueuedAt": "2021-08-12T10:00:00.000000Z",
    "startedAt": null,
    "finishedAt": null
}
```

##### Errors

- 🔴 If a master key is configured on the server-side but missing from the client in the `X-MEILI-API-KEY` header, the API returns a `401 Unauthorized` - `missing_authorization_header` error.
- 🔴 If a master key is sent by the client but does not match the value configured on the server-side, the API returns a `403 Forbidden` - `invalid_api_key`.
- 🔴 If the task does not exist, the API returns a `404 Not Found` - `task_not_found` error.

---

**Get all tasks of an index** | `GET` - `/indexes/{indexUid}/tasks`

##### Goals

Allows users to list tasks of a particular index.

`200` - Response body - `/indexes/movies/tasks`

```json
{
    "results": [
        {
            "uid": 1,
            "indexUid": "movies",
            "batchUid": 1,
            "status": "enqueued",
            "type": "documentAddition",
            "duration": null,
            "enqueuedAt": "2021-08-12T10:00:00.000000Z",
            "startedAt": null,
            "finishedAt": null
        },
        {
            "uid": 0,
            "indexUid": "movies",
            "batchUid": 0,
            "status": "succeeded",
            "type": "documentAddition",
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

##### Errors

- 🔴 If a master key is configured on the server-side but missing from the client in the `X-MEILI-API-KEY` header, the API returns a `401 Unauthorized` - `missing_authorization_header` error.
- 🔴 If a master key is sent by the client but does not match the value configured on the server-side, the API returns a `403 Forbidden` - `invalid_api_key`.
- 🔴 If the index does not exist, the API returns a `404 Not Found` - `index_not_found` error.

---

**Get a task of an index** | `GET` - `/indexes/{indexUid}/tasks/{tasksUid}`

`200` - Response body - `/indexes/movies/tasks/1`

```json
{

    "uid": 1,
    "indexUid": "movies",
    "batchUid": 1,
    "status": "enqueued",
    "type": "documentAddition",
    "duration": null,
    "enqueuedAt": "2021-08-12T10:00:00.000000Z",
    "startedAt": null,
    "finishedAt": null
}
```

##### Errors

- 🔴 If a master key is configured on the server-side but missing from the client in the `X-MEILI-API-KEY` header, the API returns a `401 Unauthorized` - `missing_authorization_header` error.
- 🔴 If a master key is sent by the client but does not match the value configured on the server-side, the API returns a `403 Forbidden` - `invalid_api_key`.
- 🔴 If the index does not exist, the API returns a `404 Not Found` - `index_not_found` error.
- 🔴 If the task does not exist, the API returns a `404 Not Found` - `task_not_found` error.

#### 6. `task_not_found` error

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

#### 7. `MEILI_MAX_UDB_SIZE` env var and `--max-udb-size` cli option

As the notion of `update` no longer exists. The acronym `UDB` is changed by `TASK_DB`.

- `MEILI_MAX_UDB_SIZE` env var is updated to `MEILI_MAX_TASK_DB_SIZE`.
- `--max-udb-size` cli option is updated to `--max-task-db-size`.

#### 8. Asynchronous Write Operations on Index resource

To consolidate the writing behavior between an index and document modifications, configuration changes of settings, the creation, modification, and deletion of an index resource are now asynchronous.

Initially, the index creation, update, and deletion operations were synchronous, which could cause problems like race conditions.

For example, when updating documents on an index while an index is being deleted synchronously in parallel. It also improves the consistency for the understanding of writes within a MeiliSearch index among an identical behavior.

This structure allows us to facilitate communications and write propagations in a high availability context in the future.

The main change in the API is that the routes response described below now becomes a `202 Accepted` response with the summarized task response payload.

Errors are now part of the `task` as for other asynchronous operations.

New task types are also added for these operations. `indexCreation`, `indexUpdate`, and `indexDeletion`.

**Create an index** | `POST` - `/indexes`

```json
{
    "uid": "movies"
}
```

`202` - Accepted Response body

```json
{
    "uid": 0,
    "indexUid": "movies",
    "status": "enqueued",
    "type": "indexCreation",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

- 💡 Automatic index creation using the `/indexes/:indexToCreate/documents` route generates only one `documentAdditions` task that also handles index creation.

**Update an index** | `PUT` - `/indexes/:indexUid`

```json
{
    "primaryKey": "uid"
}
```

`202` - Accepted Response body

```json
{
    "uid": 1,
    "indexUid": "movies",
    "status": "enqueued",
    "type": "indexUpdate",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

**Delete an index** | `DELETE` - `/indexes/:indexUid`

`202` - Accepted Response body

```json
{
    "uid": 1,
    "indexUid": "movies",
    "status": "enqueued",
    "type": "indexDeletion",
    "enqueuedAt": "2021-08-12T10:00:00.000000Z"
}
```

## 2. Technical details

### I. Measuring

- Number of call on `indexes/:indexUid/tasks` per instance
- Number of call on `indexes/:indexUid/tasks/:taskUid` per instance
- Number of call on `tasks` per instance
- Number of call on `tasks/:taskUid` per instance

## 3. Future Possibilities

- Add a pagination system for on `/tasks` and `indexes/:indexUid/tasks` lists.
- Add enhanced filtering capabilities.
- Simplify `documentAddition` and `documentPartial` type and elaborate on `details` metadata.
- Use Hateoas capability to give direct access to a `task` resource.
- Add dedicated task type names modifying a sub-setting. e.g. `SearchableAttributesUpdate`.
- Reconsider existence of `/indexes/:indexUid/tasks/:taskUid` route since it is similar to `/tasks/:taskUid`.
- Add an archived state for old `tasks`.
- Indicate the `API Key` identity that added a `task`. It should not permits to