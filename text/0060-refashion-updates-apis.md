- Title: Refashion Updates APIs
- Start Date: 2021-08-13
- Specification PR: [#55](https://github.com/meilisearch/specifications/pull/60)
- Discovery Issue: [#48](https://github.com/meilisearch/product/issues/48)

# Refashion Updates APIs

## 1. Functional Specification

### I. Summary

The term `update` is not the best choice as it can be confused with document updates or `settings`. We have chosen to replace this term with `task`, which is more generic and better reflects the meaning of this API resource.

As an additional change, we have reworked the format of an update to make it more in line with our expectations of an API that is supposed to be easily understandable and developer-oriented.

The changes we will make to the response format of the `task` object lists will make it easier to add more functionality in a future iteration. See the Future Possibilities section for a brief overview.

Two new API endpoints are added. Although quite simple, they allow to consult the list of tasks or a specific task without being forced to know the related index.

#### Summary Key Points

-  The `update` resource is renamed `task`. The names of existing API routes are also changed to reflect this change.
-  The format of the `task` object is updated.
    - `updateId` becomes `uid`.
    - Attributes of an error appearing in a `failed` `task` are now contained in a dedicated `error` object.
    - `type` is no longer an object. It now becomes a string containing the values of its `name` field previously defined in the `type` object.
    - The possible values for the `type` field are reworked to be more clear and consistent with our naming rules.
    - A `details` object is added to contain specific information related to a `task` payload that was previously displayed in the `type` nested object.
    - An `indexUid` field is added to give information about the related index on which the task is performed.
    - `processed` status changes to `succeeded`.
    - `startedProcessingAt` is updated to `startedAt`.
    - `processedAt` is updated to `finishedAt`.
- `202 Accepted` requests previously returning an `updateId` are now returning a summarized `task` object.

### II. Motivation

The main motivation is to stabilize the current `update` resource to a version that conforms to our API convention and thus allow future evolutions on a more solid base. We would like to modify the name `update`, its format is also to be reviewed because some attributes are not immediately clear, either in the possible values or in the chosen names.

### III. Explanation

#### 1. `task` object definition

##### **Fully Qualified `task` object**

> This fully qualified version appears as response object on `task` dedicated endpoints.

| field   | type    | description                     |
|---------|---------|---------------------------------|
| uid      | integer | Unique sequential identifier           |
| indexUid | string | Unique index identifier |
| status  | string  | Status of the task. Possible values are `enqueued`, `processing`, `succeeded`, `failed`                                |
| type    | string  | Type of the task. Possible values are `documentsAddition`, `documentsPartial`, `documentsDeletion`, `settingsUpdate` |
| details    | object  |  Details information of the type payload. See examples. |
| error | object | Error object containing error details and context when a task has a `failed` status. See https://github.com/meilisearch/specifications/pull/61|
| duration | number | Total elasped seconds the engine was in `processing` state. Default is set to `null`  |
| enqueuedAt | string | Represent the date and time as `ISO-8601` format when the task has been enqueued |
| startedAt | string | Represent the date and time as `ISO-8601` format when the task has been dequeued and started to be processed. Default is set to `null`|
| finishedAt | string | Represent the date and time as `ISO-8601` format when the task has `failed` or `succeeded`. Only displayed when the status is `failed` or `succeeded`. Default is set to `null` |

> 💡 The order of the fields must be returned in this order.
>
> 🔮 Future consideration - `task` object contained in a list could have a `link` attribute allowing direct access to the resource.

##### Summarized `task` Object for `202 Accepted`

| field   | type    | description                     |
|---------|---------|---------------------------------|
| uid      | integer | Unique sequential identifier           |
| indexUid | string | Unique index identifier |
| status  | string  | Status of the task. Value is `enqueued` |
| enqueuedAt | string | Represent the date and time as `ISO-8601` format when the task has been enqueued |

> 💡 The order of the fields must be returned in this order.
>
> 💡 This summarized version appears only in `202 Accepted` responses.
>
> 🔮 Future consideration - `task` object contained in a list of `tasks` could have a `link` attribute allowing direct access to the resource.

#### 2. `status` field enum

| old        | new           |
|------------|---------------|
| enqueued   | -             |
| processing | -             |
| processed  | **succeeded** |
| failed     | -             |

> 👍 Better semantic differentiation than `processing` and `processed`. The final status of a processed task is `succeeded` or `failed`.

#### 3. `type` field Enum changes

| old        | new           |
|------------|---------------|
| DocumentsAddition | documentsAddition     |
| DocumentsPartial | documentsPartial   |
| DocumentsDeletion  | documentsDeletion |
| Settings     | settingsUpdate |

> 👍 Type values follow a `camelCase` naming convention.
>
> 💡 `Settings` is updated to be more precise with the name `settingsUpdate`.
>
> 🔮 Future consideration - Dedicated type name for sub-settings endpoints usage `SearchableAttributesUpdate`.

#### 4. Examples

e.g. A fully qualified `task` object in `enqueued` state.

```json=
{
        "uid": 0,
        "indexUid": "movies",
        "status": "enqueued",
        "type": "settingsUpdate",
        "details": {
            "rankingRules": [
                "typo",
                "desc(ranking)",
                "words",
                "proximity",
                "attribute",
                "wordsPosition",
                "exactness"
            ]
        },
        "duration": null,
        "enqueuedAt": "2021-08-10T14:29:17.000000Z",
        "startedAt": null,
        "finishedAt": null
}
```

e.g. A fully qualified `task` object in `processing` state.

```json=
{
        "uid": 0,
        "indexUid": "movies",
        "status": "processing",
        "type": "settingsUpdate",
        "details": {
            "rankingRules": [
                "typo",
                "desc(ranking)",
                "words",
                "proximity",
                "attribute",
                "wordsPosition",
                "exactness"
            ]
        },
        "duration": null,
        "enqueuedAt": "2021-08-10T14:29:17.000000Z",
        "startedAt": "2021-08-10T14:29:18.000000Z",
        "finishedAt": null,
}
```

e.g. A fully qualified `task` object in `succeeded` state.

```json=
{
        "uid": 0,
        "indexUid": "movies",
        "status": "succeeded",
        "type": "settingsUpdate",
        "details": {
            "rankingRules": [
                "typo",
                "desc(ranking)",
                "words",
                "proximity",
                "attribute",
                "wordsPosition",
                "exactness"
            ]
        },
        "duration": 1.0,
        "enqueuedAt": "2021-08-10T14:29:17.000000Z",
        "startedAt": "2021-08-10T14:29:18.000000Z",
        "finishedAt": "2021-08-10T14:29:19.000000Z"
}
```

e.g. A fully qualified `task` object in `failed` state.

```json=
{
        "uid": 0,
        "indexUid": "movies",
        "status": "failed",
        "type": "settingsUpdate",
        "details": {
            "rankingRules": [
                "typo",
                "desc(ranking)",
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
        "duration": 1.0,
        "enqueuedAt": "2021-08-10T14:29:17.000000Z",
        "startedAt": "2021-08-10T14:29:18.000000Z",
        "finishedAt": "2021-08-10T14:29:19.000000Z",
}
```

e.g. A summarized `task` object as a response for a `202 Accepted` HTTP code.

```json=
{
    "uid": 0,
    "indexUid": "movies",
    "status": "enqueued",
    "enqueuedAt": "2021-08-11T09:25:53.000000Z"
}
```

---

### GET `/tasks`

#### Goal

Allows users to list tasks globally regardless of the indexes involved. Particularly useful to visualize all the tasks.

#### Response Definition

- `task` objects are contained in a `data` array.
- By default, objects are sorted by desc order on `uid` field. So the most recent tasks appear first.

#### Response Example

```json=
{
    "data": [
        {
            "uid": 1,
            "status": "enqueued",
            "indexUid": "movies",
            "type": "documentsAddition",
            "duration": null,
            "enqueuedAt": "2021-08-12T10:00:00.000000Z",
            "startedProcessingAt": null,
            "finishedAt": null
        },
        {
            "uid": 0,
            "status": "succeeded",
            "indexUid": "movies",
            "type": "documentsAddition",
            "details": {
                "number": 100
            },
            "duration": 16.000,
            "enqueuedAt": "2021-08-11T09:25:53.000000Z",
            "startedAt": "2021-08-11T10:03:00.000000Z",
            "finishedAt": "2021-08-11T10:03:16.000000Z"
        }
    ]
}
```

> 🔮 Future consideration - Add a keyset pagination
>
> 🔮 Future consideration - Add dedicated query parameters for filtering

### GET `/tasks/{uid}`

#### Goal

Allows users to get a detailed `task` object retrieved by the `uid` field regardless of the index involved.

> 🔮 Future consideration -  Reconsider usage of `/indexes/indexUId/tasks/{uid}` route since the two routes are stricly similar.

#### Response Example

e.g. `/tasks/1`

```json=
{

    {
        "uid": 1,
        "status": "enqueued",
        "indexUid": "movies",
        "type": "documentsAddition",
        "duration": null,
        "enqueuedAt": "2021-08-12T10:00:00.000000Z",
        "startedProcessingAt": null,
        "finishedAt": null
    }
}
```

---

### GET `/indexes/{indexUid}/tasks`

TBD

### GET `/indexes/{indexUid}/tasks/{tasksUid}`

TBD

### IV. Finalized Key Changes

## 2. Technical details

### I. Measuring

TBD

## 3. Future Possibilities

- Add pagination on `task` lists.
- Add filtering capabilities.
