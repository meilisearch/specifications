- Title: Task Resource Lists Pagination
- Start Date: 2021-09-13
- Specification PR: []()
- Discovery Issue: []()

# Task Resource Lists Pagination

## 1. Functional Specification

### I. Summary

`task` resource lists now support cursor pagination, allowing users to browse multiple sets of `task` items.

#### Summary Key points

- Add a `pagination` metadata object to task list response.
- It concerns `/tasks` and `/indexes/:uid/tasks`.
-

### II. Motivation

Following the specification aiming to stabilize the `task` API resource, we want to give MeiliSearch capabilities to adapt to many use-cases, thus involving unpredictable orders of magnitude regarding `task` growth over time.

This feature should be usable by both solo developers and technical teams.

The objectives set during the exploration phase for the pagination of the task lists are:

- Performance should remain nearly constant, regardless of task volume.
- The results must be browsable in a consistent way between each call to the API, no matter how much the number of tasks is increased or reduced while the navigation is happening.
- Do not reinvent the wheel and therefore facilitate its implementation by selecting a proven paging interface.
- A subsidiary, but nonetheless important objective, is to feed our API guideline to be able to respond quickly to such needs in the future within the company. By proposing a proven solution we will be able to build faster while keeping a state of the art.

### III. Technical Explanations

#### Why a cursor based pagination?

As seen in the [Rest API Format Convention](https://github.com/meilisearch/product/issues/44#issuecomment-895888679), a cursor pagination is more appropriate when the data can grow or shrink quickly in terms of magnitude.

####  Pros

The performance is better than the not so good  but old pagination with `offset`/`limit`.

Cursor pagination keeps the results consistent between each page as the data evolves. It avoids the [Page Drift effect](https://use-the-index-luke.com/sql/partial-results/fetch-next-page) especially when the data are displayed from the most recent to the oldest.

Moreover, the performance is superior to more traditional pagination since the complexity can be constant e.g. `O(1)` to reach the identifier marking the beginning of the new slice to be returned in the hash table.

####  Cons

The main drawback of this type of pagination is that it does not allow one to navigate within a finite number of pages. It is also limited to a precise sorting criterion on unique identifiers ordered sequentially.

---

#### Cursor pagination definition

Rather than injecting the pagination metadata into the root of the response, we can describe a `pagination` object that allows clients to find this information more clearly in responses. It's a common practice to guide consumers by providing metadata objects through a Rest API.

Being consistent in our design not only makes our API easier to use but also makes its design simpler.

> 💡 This `pagination` metadata object can also contain the necessary attributes for other types of pagination that could be implemented for specific endpoints, such as per-page pagination.

### Cursor based `pagination` object definition for the `task` lists.

| field | type | description                         |
|------|------|--------------------------------------|
| limit | int  | Default `30`. A limit on the number of tasks to be returned, between `1` and `100`. |
| after  | int - nullable  | Represents the query parameter to send to fetch the next slice of the results. The next results will start at `after+1`. When the returned value is null, it means that all the data have been browsed in the given order. |

###  Usages examples

This specification demonstrate cursor paging on `/tasks` but it is also equivalent for `indexes/:uid/tasks`. Although the ids can be "holey" within an index in case several indexes are managed within the instance. The `uid` remains sorted sequentially and can be used to navigate in a list of `tasks` objects. In case the query parameter is sent with an `uid` value that does not exist in an index, the server takes the next closest to select the `tasks` to return.

---

**Request intiial default slice of `tasks`**

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
    "pagination": {
        "limit": 20,
        "after": 1330
    }
}
```

**Request the next slice of `tasks` with a size of `50` tasks items**

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
    "pagination": {
        "limit": 50,
        "after": 1279
    }
}
```

**End of cursor pagination demonstration**

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
    "pagination": {
        "limit": 10,
        "after": null
    }
}
```

- 💡 `after` response parameter is null because there are no more `tasks` to fetch.

### IV. Finalized Key Changes
tbd

## 2. Technical Aspects
tbd

## 3. Future Possibilities
tbd