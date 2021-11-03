- Title: Task Resource Lists Pagination
- Start Date: 2021-09-13
- Specification PR: [#72](https://github.com/meilisearch/specifications/pull/72)
- Discovery Issue: [#190](https://github.com/meilisearch/product/issues/190)

# Task Resource Lists Pagination

## 1. Functional Specification

### I. Summary

`task` resource lists now support cursor pagination, allowing users to browse multiple sets of `task` items.

#### Summary Key Points

- Add `limit` and `after` response attributes to task list response.
- It concerns `/tasks` and `/indexes/:uid/tasks`.

### II. Motivation

Following the specification aiming to stabilize the `task` API resource, we want to give MeiliSearch capabilities to adapt to many use-cases, thus involving unpredictable orders of magnitude regarding `task` growth over time.

This feature should be usable by both solo developers and technical teams.

The objectives set during the discovery's exploration phase for the pagination of the task lists are:

- Performance should remain nearly constant, regardless of task volume.
- The results must be browsable in a consistent way between each call to the API, no matter how much the number of tasks is increased or reduced while the navigation is happening.
- Do not reinvent the wheel and therefore facilitate its implementation by selecting a proven paging interface.
- A subsidiary, but critical objective, is to feed our API guideline to respond quickly to such needs in the future within the company. By proposing a proven solution, we can build faster with confidence.

### III. Technical Explanations

#### Why cursor-based pagination?

As seen in the [Rest API Format Convention](https://github.com/meilisearch/product/issues/44#issuecomment-895888679), cursor-based pagination is more appropriate when the data can grow or shrink quickly in terms of magnitude.

#### Pros

The performance is better than the not-so-good but old pagination with `offset`/`limit`.

Cursor pagination keeps the results consistent between each page as the data evolves. It avoids the [Page Drift effect](https://use-the-index-luke.com/sql/partial-results/fetch-next-page), especially when the data is sorted from the most recent to the oldest.

Moreover, the performance is superior to traditional pagination since the computational complexity remains constant to reach the identifier marking the beginning of the new slice to be returned from a hash table.

#### Cons

The main drawback of this type of pagination is that it does not navigate within a finite number of pages. It is also limited to a precise sorting criterion on unique identifiers ordered sequentially.

---

### Response attributes.

| field | type | description                          |
|-------|------|--------------------------------------|
| limit | int  | Default `30`. |
| after | int - nullable  | Represents the query parameter to send to fetch the next slice of the results. The first item for the next slice starts at `after+1`. When the returned value is null, it means that all the data have been browsed in the given order. |

### GET query parameters for cursor-based pagination

| field | type | required | description  |
|-------|------|----------|--------------|
| limit | int  | No       | Default `30`. Limit on the number of tasks to be returned, between `1` and `100`. |
| after | int  | No       | Limit results to tasks with uids greater/lower than the specified uid. |

### Usages examples

This specification demonstrates cursor paging on `/tasks`, but it should be equivalent for `indexes/:uid/tasks`. Although the uids can be "holey" on the `/indexes/:uid/tasks` endpoint if several indexes are managed within the instance. The items uid remains sorted sequentially and can be used to navigate a list of `tasks` objects.

---

**Initial default slice of `tasks`**

`GET` - `/tasks`

```json
{
    "results": [
        {
            "uid": 1350,
            "indexUid": "movies",
            "type": "documentsAddition",
            ...,
        },
        ...,
        {
            "uid": 1330,
            "indexUid": "movies_reviews",
            "type": "documentsAddition",
            ...,
        }
    ],
    "limit": 20,
    "after": 1330
}
```

**Request the next slice of `tasks` items with a limit of `50` tasks**

`GET` - `/tasks?after=1330&limit=50`

```json
{
    "results": [
        {
            "uid": 1329,
            "indexUid": "movies",
            "type": "documentsAddition",
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
    "after": 1279
}
```

**End of cursor pagination**

`GET` - `/tasks?after=20`

```json
{
    "results": [
        {
            "uid": 19,
            "indexUid": "movies",
            "type": "documentsAddition",
            ...,
        },
        ...,
        {
            "uid": 0,
            "indexUid": "movies",
            "type": "documentsAddition",
            ...,
        }
    ],
    "limit": 10,
    "after": null
}
```

- 💡 `after` response parameter is null because there are no more `tasks` to fetch. It means that the response represents the last slice of results for the given resource list.

---

### Behaviors for `limit` and `after` query parameters

#### `limit`

- If `limit` is not set, the default value is chosen.
- If `limit` is set and it is not between the minimum and maximum values, the default value is chosen.
- If `limit` is not an integer, the default value is chosen.

#### `after`

- If `after` is set with an out of bounds task `uid`, the response returns an empty `results` array and `after` is set to `null`.

## 2. Technical Aspects
n/a

## 3. Future Possibilities

### Reverse ordering of tasks

We might decide to implement the ability to reverse the list traversal using the query parameter `sort` in the future. e.g. `&sort=uid:asc`.

The use of the cursor-based pagination would remain the same except that the first element of the results would start with the oldest `uid` for the given resource.

### Filtering capabilities

We might implement filter capabilities on specific attributes.

e.g. `...&type=documentsDeletion` to specifically browse the tasks dedicated to document deletion.

These filters could be cumulated as a `AND` operator to refine the context of the tasks to retrieve. e.g. `...&type=documentsDeletion&status=succeeded`.

### Link HTTP Headers

We could take advantage of the Link header to propose navigation links that are not displayed in the body of the response.

e.g. `Link: <https://api.meilisearch.com/tasks?after=10&limit=10>; rel="next"`

It has the advantage of not displaying the pagination object in the body of the response, and if the link for rel="next" doesn't exist, then the user has reached the end of his navigation for the given resource.