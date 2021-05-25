- Title: Paginated Search
- Start Date: 2021-05-25
- Specification PR: [#42](https://github.com/meilisearch/specifications/pull/42)
- MeiliSearch Tracking-Issues:

# Paginated Search

## 1. Feature Description and Interaction

### I. Summary

Pagination allows the content to be viewed in the form of a sequence in order to facilitate the browsing of its content by not overloading the number of elements to be viewed for the user. Pagination also has the advantage of easily providing the total number of items that remain to be traversed by not overloading the network with all the content that can be loaded.

### II. Motivation

It will make it possible to rule on the subject of the use and behavior of the engine with regard to the pagination of search results for which we have a lot of user feedback and questions about its behavior.

Naturally, our users rely on `nbHits` to determine the number of possible pages. However, in the context of the search endpoint and the internal functioning of the engine dedicated to a fast and accurate search, this number of results represents a number of possible candidates for the search result. This number is not exhaustive to avoid a loss in performance.

Developers who want to implement a finite paged search are therefore faced with a hazardous situation. For example, it is possible that by clicking directly on the last page the search will not return any results. This can be explained by the behavior of the search engine which finally finds no relevant candidate to return for this page.

The integration team regularly implements a workaround to propose a finite search by calculating on the client side the number of pages by scanning a maximum of documents on a first API call. This is not the most efficient method. In addition, we often have developers who do not use our SDKs for various reasons who do not understand the behavior of the engine when it comes to getting paged results.

### III. Additional Materials

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
- The `1000` default value is set to guarantee good performance.
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

### IV. Explanation

We don't necessarily want to push and support finite pagination for the search endpoint. However, we have determined that many developers naturally implement this type of paging and we want to limit support requests for it. We have therefore made some choices that will be implemented in our SDKs to facilitate their use on this use-case and avoid the implementation of the current workaround.

Our solution will not be officially documented at first, it will evolve in the future depending on its viability.

The solution consists in having a hidden parameter that will ask the engine to force the exhaustiveness of the number of results in order to be able to build a finite pagination on a limited set of results. This limit will be implemented within the engine and will be brought to evolve according to the returns, performances. We could imagine one day to open an option to increase or decrease this limit to the users in the way of Algolia.

To make it short, the finished pagination will be an experimental feature that will evolve.

##### Requirements

###### [GET/POST Search API endpoint](https://docs.meilisearch.com/reference/api/search.html)

- ✅ Accept a `forceExhaustivity` boolean parameter to strictly ensure that the engine will return an exhaustive `nbHits` value.
- ✅ Add a constant in the core to have a limited number of documents when `forceExhaustivity` is set to `true`. We will start with a value of `1000`. This value will evolve according to our analysis. In the future, if we decide to support this feature officially, we could open it via the API so that the user can change the behavior according to his needs with the consequences that it implies on the performances. `exhaustivityLimit` is a proposal for this constant.
- ✅ `exhaustiveNbHits` should be set to `true` when `forceExhaustivity` is set to `true`. `exhaustiveNbHits` being all the time at false currently, this will be the way to make it dynamic and keep it consistent.

### V. Impact on Documentation

- The `paginateSearch` will be documented in SDKS docs (Code Sample, methods behavior etc). The limit of `1000` exhaustives number should be mentionned. We should also explain that it is due to performance reasons to keep fast search performance.
- Besides that we will write a blog post about our vision for a search experience. It should be understood that we prefer to push a non-finite search like an infinite scroll rather than a finite search.

### VI. Impact on SDKs

- The official SDKs will implement a new method `paginateSearch` that will internally send this `forceExhaustivity` parameter to `true`.

## 2. Technical Aspects

### I. Analytics
- We will track search endpoint to distribute usage between a paged search and a non-paged search. It is sufficient to track the usage of `forceExhaustivity` to do this analysis.

## 3. Future Possibilities

- Open `forceExhaustivity` in official API documentation.
- Allow users to configure the `exhaustivityLimit` in settings to be used when `forceExhaustivity` is set to `true`.

