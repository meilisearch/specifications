- Title: Task Resource Lists Filtering
- Start Date: 2021-09-13
- Specification PR: [#73](https://github.com/meilisearch/specifications/pull/73)

# Task Resource Lists Filtering

## 1. Functional Specification

### I. Summary

Add filtering capabilities to the `tasks` endpoints to facilitate the management of an instance and its indexes, or a specific index.

This first iteration adds filters on the `status` and `type` attributes of the `task` API resource.

#### Summary Key Points

- Add filtering capabilities on `type` and `status` for `GET` `task` lists endpoints.

### II. Motivation

Following the specification aiming to stabilize the `task` API resource, we want to give users the capability to refine the lists of task according to several criteria to find precise information more quickly.

### III. Technical Explanations

#### Query parameters definition

| parameter | type | required | description                         |
|------|------|----------|----------------------------|
| status | string | No | Possible values are all the status of a task. By default, when `status` query parameter is not set, all task statuses are returned. |
| type  | string | No | Possible values are all the types of a task. By default, when `type` is not set in, all task types are returned.  |

### Usages examples

This specification demonstrates filtering on `/tasks`, it should be equivalent for `indexes/:uid/tasks`.

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
            "type": "documentsAddition",
            ...,
        },
        ...,
        {
            "uid": 1330,
            "indexUid": "movies_reviews",
            "status": "succeeded",
            "type": "documentsDeletion",
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
            "type": "documentsAddition",
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

**Filter `tasks` that are of `documentsAddition` type**

`GET` - `/tasks?type=documentsAddition`

```json
{
    "results": [
        {
            "uid": 1350,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentsAddition",
            ...,
        },
        ...,
        {
            "uid": 1343,
            "indexUid": "movies",
            "type": "succeeded",
            "type": "documentsAddition",
            ...,
        }
    ],
    ...
}
```

- 💡 `status` and `type` can be used together. The two parameters are cumulated and a `AND` operation is performed between the two filters.

**Filter `tasks` that are of `documentsAddition` type and have a `failed` status**

`GET` - `/tasks?type=documentsAddition&status=failed`

```json
{
    "results": [
        {
            "uid": 1350,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentsAddition",
            ...,
        },
        ...,
        {
            "uid": 1346,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentsAddition",
            ...,
        }
    ],
    ...
}
```

- `type` and `status` query parameters can be read as is `type=documentsAddition AND status=failed`.

---

### Behaviors for `status` and `type` query parameters.

#### `status`

- 🔴 If the `status` value is not consistent with one of the task statuses, an `invalid_task_status` error is returned.

#### `invalid_task_status` Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": ":status is invalid. Available task statuses are: :taskStatuses.",
    "code": "invalid_task_status",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#invalid_task_status"
}
```

- The `:status` is inferred when the message is generated.
- The `:taskStatuses` is inferred when the message is generated.

#### `type`

- 🔴 If the `type` value is not consistent with one of the task types, an `invalid_task_type` error is returned.

#### `invalid_task_type` Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": ":type is invalid. Available task types are: :taskTypes.",
    "code": "invalid_task_type",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#invalid_task_type"
}
```

- The `:type` is inferred when the message is generated.
- The `:taskTypes` is inferred when the message is generated.

#### Empty `results`

💡 If no results match the filters. A response is returned with an empty `results` array.

## 2. Technical Aspects
n/a

## 3. Future Possibilities

- Filter `task` lists according to multiple types or statuses by separating several values with the `,` character.  This character would be interpreted as an `OR`. e.g. `?status=documentsAddition,settingsUpdate&type=failed,enqueued`