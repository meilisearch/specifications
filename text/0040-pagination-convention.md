- Title: Pagination Convention
- Start Date: 2021-05-10
- Specification PR: [#40](https://github.com/meilisearch/specifications/pull/40)
- MeiliSearch Tracking-Issues:

# Pagination Convention

## 1. Feature Description and Interaction

### I. Summary

Pagination allows the content to be viewed in the form of a sequence in order to facilitate the browsing of its content by not overloading the number of elements to be viewed for the user. Pagination also has the advantage of easily providing the total number of items that remain to be traversed by not overloading the network with all the content that can be loaded.

### II. Motivation

This specification will be used as a convention to implement simple and standard paginations to be set up on API routes such as `/updates`, `/documents` or `/dumps` for example. Since the pagination of search results is intrinsically linked to technical implementations of the search engine, this specific case is detailed in its own specification. [Paginated Search specification](https://github.com/meilisearch/specifications/blob/main/text/0042-paginated-search.md).Thus this specification does not deal with this specific case.

### III. Additional Materials
N/A

### IV. Explanation

#### Examples

E.g. [GET `/indexes/:index_uid/updates`](https://docs.meilisearch.com/reference/api/updates.html#get-all-update-status)

##### Default response example

###### Request

`/indexes/:index_uid/updates`

> For the example, let's imagine that the default limit is `20`.

###### Response
```
[
    "data": [
        {
            "status": "processed",
            "updateId": 0,
            "type": {
                "name": "DocumentsAddition",
                "number": 285
            },
            "duration": 0.255927261,
            "enqueuedAt": "2021-05-07T11:23:08.366372554Z",
            "processedAt": "2021-05-07T11:23:08.645330603Z"
        },
        {
            "status": "processed",
            "updateId": 1,
            "type": {
                "name": "Settings",
                "settings": {
                    "ranking_rules": {
                        "Update": [
                            "Typo",
                            "Words",
                            "Proximity",
                            "Attribute",
                            "WordsPosition",
                            "Exactness"
                        ]
                    },
                    "distinct_attribute": "Nothing",
                    "primary_key": "Nothing",
                    "searchable_attributes": "Nothing",
                    "displayed_attributes": "Nothing",
                    "stop_words": "Nothing",
                    "synonyms": "Nothing",
                    "attributes_for_faceting": "Nothing"
                }
            },
            "duration": 0.26251344,
            "enqueuedAt": "2021-05-07T11:25:21.172902488Z",
            "processedAt": "2021-05-07T11:25:21.450619532Z"
        },
        {
            "status": "processed",
            "updateId": 2,
            "type": {
                "name": "Settings",
                "settings": {
                    "ranking_rules": "Nothing",
                    "distinct_attribute": "Nothing",
                    "primary_key": "Nothing",
                    "searchable_attributes": {
                        "Update": [
                            "citation"
                        ]
                    },
                    "displayed_attributes": "Nothing",
                    "stop_words": "Nothing",
                    "synonyms": "Nothing",
                    "attributes_for_faceting": "Nothing"
                }
            },
            "duration": 0.238216885,
            "enqueuedAt": "2021-05-07T11:25:43.073716593Z",
            "processedAt": "2021-05-07T11:25:43.325663019Z"
        },
        {
            "status": "processed",
            "updateId": 3,
            "type": {
                "name": "ClearAll"
            },
            "duration": 0.001705567,
            "enqueuedAt": "2021-05-13T09:59:14.406552884Z",
            "processedAt": "2021-05-13T09:59:14.420332760Z"
        },
        ...
    ],
    "offset": 0,
    "limit": 20,
    "total": 198,
    "size": 20
]
```

##### Get page 1 of updates with a page size of 2 items

###### Request

`/indexes/:index_uid/updates?offset=0&limit=2`

###### Response
```
[
    "data": [
        {
            "status": "processed",
            "updateId": 0,
            "type": {
                "name": "DocumentsAddition",
                "number": 285
            },
            "duration": 0.255927261,
            "enqueuedAt": "2021-05-07T11:23:08.366372554Z",
            "processedAt": "2021-05-07T11:23:08.645330603Z"
        },
        {
            "status": "processed",
            "updateId": 1,
            "type": {
                "name": "Settings",
                "settings": {
                    "ranking_rules": {
                        "Update": [
                            "Typo",
                            "Words",
                            "Proximity",
                            "Attribute",
                            "WordsPosition",
                            "Exactness"
                        ]
                    },
                    "distinct_attribute": "Nothing",
                    "primary_key": "Nothing",
                    "searchable_attributes": "Nothing",
                    "displayed_attributes": "Nothing",
                    "stop_words": "Nothing",
                    "synonyms": "Nothing",
                    "attributes_for_faceting": "Nothing"
                }
            },
            "duration": 0.26251344,
            "enqueuedAt": "2021-05-07T11:25:21.172902488Z",
            "processedAt": "2021-05-07T11:25:21.450619532Z"
        }
    ],
    "offset": 0,
    "limit": 2,
    "total": 198,
    "size": 2
]
```

##### Get page 2 of updates with a page size of 2 items

###### Request

`/indexes/:index_uid/updates?offset=2&limit=2`

###### Response

```
[
    "data": [
        {
            "status": "processed",
            "updateId": 2,
            "type": {
                "name": "Settings",
                "settings": {
                    "ranking_rules": "Nothing",
                    "distinct_attribute": "Nothing",
                    "primary_key": "Nothing",
                    "searchable_attributes": {
                        "Update": [
                            "citation"
                        ]
                    },
                    "displayed_attributes": "Nothing",
                    "stop_words": "Nothing",
                    "synonyms": "Nothing",
                    "attributes_for_faceting": "Nothing"
                }
            },
            "duration": 0.238216885,
            "enqueuedAt": "2021-05-07T11:25:43.073716593Z",
            "processedAt": "2021-05-07T11:25:43.325663019Z"
        },
        {
            "status": "processed",
            "updateId": 3,
            "type": {
                "name": "ClearAll"
            },
            "duration": 0.001705567,
            "enqueuedAt": "2021-05-13T09:59:14.406552884Z",
            "processedAt": "2021-05-13T09:59:14.420332760Z"
        }
    ],
    "offset": 2,
    "limit": 2,
    "total": 198,
    "size": 2
]
```

##### Get last page of processed updates

###### Request

`/indexes/:index_uid/updates?status=processed&offset=50&limit=10`

> For the example, let's imagine the total number of processed updates is `60` and we want page of `10` items.

###### Response
```
[
    "data": [
        {
            "status": "processed",
            "updateId": 49,
            "type": {
                "name": "Settings",
                "settings": {
                    "ranking_rules": "Nothing",
                    "distinct_attribute": "Nothing",
                    "primary_key": "Nothing",
                    "searchable_attributes": {
                        "Update": [
                            "citation"
                        ]
                    },
                    "displayed_attributes": "Nothing",
                    "stop_words": "Nothing",
                    "synonyms": "Nothing",
                    "attributes_for_faceting": "Nothing"
                }
            },
            "duration": 0.238216885,
            "enqueuedAt": "2021-05-07T11:25:43.073716593Z",
            "processedAt": "2021-05-07T11:25:43.325663019Z"
        },
        ...
        {
            "status": "processed",
            "updateId": 59,
            "type": {
                "name": "ClearAll"
            },
            "duration": 0.001705567,
            "enqueuedAt": "2021-05-13T09:59:14.406552884Z",
            "processedAt": "2021-05-13T09:59:14.420332760Z"
        }
    ],
    "offset": 50,
    "limit": 10,
    "total": 60,
    "size": 10
]
```

#### Requirements

- ✅ Each paginable route must return the `total` number of items that match the query and the number of items returned in the response as `size`.
- ✅ Each paginable route should be able to handle `offset` and `limit` to return a specific page.
- ✅ Each paginable route should handle a default `limit`. E.g. `20`
- ✅ Each paginable route should handle a default `offset` parameter with a value of `0`.
- ✅ `limit` and `offset` are number parameters.
- ✅ If `limit` and `offset` can't return any results, the API should not return an error. It's preferable to return empty results in this case.

### V. Impact on Documentation

- Create a reference pagination document page to explain the behavior of standard pagination as a central references.
- Create a pagination component to easily branch pagination parameters and explanation on each concerned routes ?
- Each route handling a standard pagination should link to this page and mention the default value for `limit`.

### VI. Impact on SDKs

- Implement a method to paginate easily on standard routes.

## 2. Technical Aspects
N/A

## 3. Future Possibilities
- Develop and branch a `_links` component  with `prev`, `next` parameters.
