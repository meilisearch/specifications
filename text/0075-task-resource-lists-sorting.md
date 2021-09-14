- Title: Task Resource Lists Sorting
- Start Date: 2021-09-13
- Specification PR: [#75](https://github.com/meilisearch/specifications/pull/75)

# Task Resource Lists Sorting

## 1. Functional Specification

### I. Summary

Add sorting capabilities by `desc`/`order` on `task` `uid` to the `tasks` endpoints to facilitate the management of an instance and its indexes, or a specific index.

#### Summary Key Points

- Add ordering capabilities on `task` `uid` for `GET` `tasks` endpoints.

### II. Motivation

Following the specification aiming to stabilize the `task` API resource, we want to give users the capability to browse lists of task according to the direction of their choice.

### III. Technical Explanations

#### Query parameter definition

| parameter | type | required | description                         |
|------|------|----------|----------------------------|
| sort | string | No | Possible values are `uid:asc` and `uid:desc`. By default, when `sort` query parameter is not set, `task` results are ordered in descending order. |

### Usages examples

This specification demonstrates sorting on `/tasks`, it should be equivalent for `indexes/:uid/tasks`.

---

**No sort query parameter - default descending order**

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

**Sorting `tasks` by `uid` in ascending order**

`GET` - `/tasks?sort=uid:asc`

```json
{
    "results": [
        {
            "uid": 1279,
            "indexUid": "movies",
            "status": "failed",
            "type": "documentsAddition",
            ...,
        },
        ...,
        {
            "uid": 1350,
            "indexUid": "movies",
            "status": "failed",
            "type": "settingsUpdate",
            ...,
        }
    ],
    ...
}
```

- 💡 This feature allows the user to change the direction of navigation of the cursor-based pagination.

---

### Behaviors for the `sort` query parameter

- 🔴 Sending an attribute different than `uid` lead to a 400 Bad Request - **invalid_sort** error.

```json
{
    "message": "Attribute :attribute is not available to sort the task resource, available attributes are: uid",
    "errorCode": "invalid_sort",
    "errorType": "invalid_request_error",
    "errorLink": "https://docs.meilisearch.com/errors#invalid_sort"
}
```
- `:attribute` is inferred when the message is generated.

- 🔴 Sending a wrong formatted value lead to a 400 Bad Request - **invalid_sort**.

```json
{
    "message": "Invalid syntax for the sort parameter: :syntaxErrorHelper.",
    "errorCode": "invalid_sort",
    "errorType": "invalid_request_error",
    "errorLink": "https://docs.meilisearch.com/errors#invalid_sort"
}
```
- `:syntaxErrorhelper` is inferred when the message is generated.

## 2. Technical Aspects
n/a

## 3. Future Possibilities
n/a