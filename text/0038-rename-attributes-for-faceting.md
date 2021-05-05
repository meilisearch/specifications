- Title: Rename attributesForFaceting
- Start Date: 2021-04-16
- Specification PR: [#38](https://github.com/meilisearch/specifications/pull/38)
- MeiliSearch Tracking-Issues: TBD

# Rename attributesForFaceting

## 1. Feature Description and Interaction

### I. Summary

### II. Motivation

As the new engine requires filterable fields to be declared in attributesForFaceting, the name of this field is not as clear as we would like it to be. Since it’s now concerning filter and possible wanted facets on search result by the user.

### III. Additional Materials
N/A

### IV.Explanation

#### I. attributesForFaceting

`attributesForFaceting` is now used for declaring fields that could be filtered and given as facets.

Since faceting is also used to filter the search result by navigation, the idea is to highlight the filtering aspect in the name of the field.

> Any fields declared in `attributesForFaceting` for filtering can be faceted. Any fields declared in `attributesForFaceting` for faceting can be filtered.

attributesForFaceting is lacking precision because it is now used to authorize the usage of `filter` and `facetsDistribution` on those fields.

### II. filterableAttributes

We are proposing `filterableAttributes` as the new parameter name. This will be clearer since all the fields declared within it now allow you to refine the search results. The declared fields allow you to refine the query in two ways. The first allows you to use these in the `filter` parameter. The second also enables the possibility of distributing the fields as a facet by using these in the `facetsDistribution` parameter.

This way of naming this field is clearly saying that the expected parameter is an array of fields. Keeping the term faceting in the field name is something to be avoided as this field is not related only to faceting. The expected action behind that will remain the fact of activating filtering for these fields.

> Faceting is a UI method of displaying document fields in a distributed fashion. Generally the final aim involves proposing a method of refining the research by interacting with the facets previously generated. Thus, filtering the search result.

On a side note, the term `facetDistribution` is a term used in e-commerce and we will therefore probably deal with this in a future specification.

### V. Impact on documentation

The documentation needs to replace occurences of `attributesForFaceting` by `filterableAttributes` [here](https://docs.meilisearch.com/reference/features/faceted_search.html#filters-or-facets). Also, the documentation should mention that fields needs to be declared in `filterableAttributes` to be used in the `filter` query parameter [here](https://docs.meilisearch.com/reference/features/filtering.html#filtering)


### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future possibilities
N/A