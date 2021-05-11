- Title: Pagination
- Start Date: 2021-05-10
- Specification PR: [#40](https://github.com/meilisearch/specifications/pull/40)
- MeiliSearch Tracking-Issues:

# Pagination

## 1. Feature Description and Interaction

### I. Summary

Pagination allows the content to be viewed in the form of a sequence in order to facilitate the browsing of its content by not overloading the number of elements to be viewed for the user. Pagination also has the advantage of easily providing the total number of items that remain to be traversed by not overloading the network with all the content that can be loaded.

### II. Motivation

This specification will do two things.

Firstly, the specification will also serve as a convention for simple and standard paginations to be set up on API routes such as the list of updates for example.

Finally, it will make it possible to rule on the subject of the use and behavior of the engine with regard to the pagination of search results for which we have a lot of user feedback and questions about its behavior.

The behavior of the search results pagination is intrinsically linked to technical implementations of the search engine. It is important to determine what can and cannot be done.

### III. Additional Materials

#### Standard Pagination Case

Nothing special to analyze here because in this common case, it seems to be quite simple to implement.

#### Search results Pagination Case

##### Algolia

Algolia offers two ways to find segmented documents at query time.

The first method is to use a standard pagination. The `page` parameter is given to request the results page from the search engine at search time.

The second method use `offset` and `length` parameters to request specific record subsets outside of a page window.

At indexing time, Algolia allows to set up 2 default parameters:

`hitsPerPage` - the number of hits returned per page. Note that it can be set a query time to tweak page size. By default, the engine set it to `20`. The maximum authorized value is `1000`.

`paginationLimitedTo` - controls the overall number of hits you can potentially retrieve for a given query. By default, `paginationLimitedTo` is set to `1000` but can be overrided with a custom value.

###### About `paginationLimitedTo`

- It control the maximum number of hits accessible via pagination. E.g. if set to 1000, the user can't retrieve more results than 1000 using pagination.
- It works with the `page` and `hitsPerPage` settings to establish the full pagination logic.
- The `1000` default value is set  to guarantee good performance.
- Algolia recommend to keep the default value to guarantee excellent performance. Note that, they also warn users on the fact that increasing the pagination limit will have a direct impact on the performance of search queries. Thus, a too high value will also make it very easy for anyone to scrappe an entire dataset so it so it also poses security problems.

E.g. Sample results

```
{
  [...],
  "page": 0,
  "nbHits": 40,
  "nbPages": 2,
  "hitsPerPage": 20,
  "exhaustiveNbHits": true
}
```

###### About `exhaustiveNbHits`

As the Algolia documentation say : "If possible, Algolia will return an accurate number of total hits. However, there are some cases where performance needs to be favored over exhaustivity. If a query returns a huge number of results, the engine will approximate the hits count to avoid having to scan the full results set. This approximation has been put in place to protect other search and indexing operations. As an alternative, you can leverage the boolean exhaustiveNbHits to either hide or tweak the display of the hits count in this case. You can also use the frequent occurrence of this value to consider fine-tuning your data and index settings to improve performance."

##### TypeSense
TBF

##### ElasticSearch
TBF


### IV. Explanation

#### Standard Pagination Case

We will take as an example the API route giving the list of updates and specify a standard pagination from it to allow users to paginate through updates list. [See route documentation](https://docs.meilisearch.com/reference/api/updates.html#get-all-update-status).

##### Requirements

- Each paginable route must return the total number of items it can render to the user at the end and the current number of items returned in the response. E.g. `total` and `size`. In the case of multiples pages, all other pages except the last one will have `size = limit`.
- Each paginable route should be able to handle `offset` and `limit` to return a specific page.
- Each paginable route should handle a default `limit`. E.g. `20`
- Each paginable route should handle a default `offset` parameter with a value of `0`.
- `limit` and `offset` are number parameters.
- If `limit` and `offset` can't return any results, the API should not return an error. It's preferable to return empty results in this case.

##### Ideas

- Provide a `_links` object with `current`, `next` and `prev` elements to facilitate pagination on client side by pre-computing `next` and `prev` urls parameters given the pagination context.


E.g. Result from `GET /indexes/myindex/updates/` at `page 1`.

```json
{
    "hits": [
        {...},
        {...},
        {...},
        {...},
        {...}
    ],
    "total": 50,
    "size": 5,
    "offset": 5,
    "limit": 5,
    "_links": {
        "current": "https://0.0.0.0:7070/indexes/myindex/updates?offset=0&limit=5",
        "next": "https://0.0.0.0:7070/indexes/myindex/updates?offset=5&limit=5",
        "prev": null
    }
}
```

E.g. Result from `GET /indexes/myindex/updates/` at `page 2`.
```json
{
    "hits": [
        {...},
        {...},
        {...},
        {...},
        {...}
    ],
    "total": 50,
    "size": 5,
    "offset": 5,
    "limit": 5,
    "_links": {
        "current": "https://0.0.0.0:7070/indexes/myindex/updates?offset=5&limit=5",
        "next": "https://0.0.0.0:7070/indexes/myindex/updates?offset=10&limit=5",
        "prev": "https://0.0.0.0:7070/indexes/myindex/updates?offset=0&limit=5"
    }
}
```
E.g. Result from `GET /indexes/myindex/updates/` at `page 10`.
```json
{
    "hits": [
        {...},
        {...},
        {...},
        {...},
        {...}
    ],
    "total": 50,
    "size": 5,
    "offset": 5,
    "limit": 5,
    "_links": {
        "current": "https://0.0.0.0:7070/indexes/myindex/updates?offset=45&limit=5",
        "next": null,
        "prev": "https://0.0.0.0:7070/indexes/myindex/updates?offset=40&limit=5"
    }
}
```

#### Search results Pagination Case

Currently, we have some issues regarding pagination. More recently on laravel scout sdk, where people want to do strict pagination.

MeiliSearch is not able to provide an exhaustive hits number in most of the cases regarding to keep a high performant search experience. MeiliSearch is aimed to put relevancy at first and does not want the searching user to go to page 2 at all.

The problem is that we have some developers in our community who expect strict pagination to navigate through results. They use the `nbHits` parameter thinking that it returns the exact count to compute the pagination.

We decided to do an internal working group to discover more paths to solve this case.

Working Group Questions (05-11-2021):
- Use page parameter instead of limit offset? Can we keep it for a specific use case? If we stand on that, standard pagination should also implement that.
- Does `nbHits` should be exhaustive most of the time? Algolia exhaustivity is false when search > 50ms to keep a fast response.
- Can the user ask the engine to enforce exhaustivity on his side to get a strict number of total results?

Working Group Answers (05-11-2021):
- We need to decide what to do about pagination. Is it a feature provided by the engine for the search? Can we state about it and evangelize another way to paginate like an infinite scroll or next/prev only. Thus, not having finite pagination?

If it is a feature that MeiliSearch judge as very important to be handle at engine level, the engine could have many possibilities that we need to discuss.

- Have a parameter to force the engine to degrade performance and return exhaustive hits count for finite pagination use cases.
- Put a convenient `limit` by default on the engine level instead of the integration team does that workaround on SDK levels. E.g. A limit of 1000 or 500 documents maximum. This limit could also be overridden by the user at his own risk regarding performance.

If we don't decide to support strict pagination on search engine level, we also have many possibilities that we need to discuss.

- Rename `nbHits` to `approximateCount` for example. By assuming that the users will therefore understand that this field cannot be used to generate a strict pagination.
- Write and explain why we don't propose a way to have a strict pagination. What and why is our vision on that?
- Keep the workaround on the official libraries and make a decicated page to guide users.

### V. Impact on Documentation
TBD

#### Standard Pagination Case
- Create a reference pagination document page to explain the behavior of pagination.
- Each route handling a standard pagination should link to this page and mention the default value for `limit`.

### VI. Impact on SDKs
TBD

## 2. Technical Aspects
TBD

## 3. Future Possibilities
TBD
