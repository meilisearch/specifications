- Title: Pagination Convention
- Start Date: 2021-05-10
- Specification PR: [#40](https://github.com/meilisearch/specifications/pull/40)
- MeiliSearch Tracking-Issues:

# Pagination Convention

## 1. Feature Description and Interaction

### I. Summary

Pagination allows the content to be viewed in the form of a sequence in order to facilitate the browsing of its content by not overloading the number of elements to be viewed for the user. Pagination also has the advantage of easily providing the total number of items that remain to be traversed by not overloading the network with all the content that can be loaded.

### II. Motivation

This specification will be used as a convention to implement simple and standard paginations to be set up on API routes such as the list of updates for example. Since the pagination of search results is intrinsically linked to technical implementations of the search engine, this specific case is detailed in its own specification.

### III. Additional Materials
N/A

### IV. Explanation

##### Requirements

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
- Introduce a `page` parameter.
