- Title: Distinct Attribute
- Start Date: 2021-04-16
- Specification PR: [#32](https://github.com/meilisearch/specifications/pull/32)
- MeiliSearch Tracking-Issues: [milli#168](https://github.com/meilisearch/milli/issues/168)

# Distinct Attribute

## 1. Feature Description and Interaction

### I. Summary

The distinct attribute is usefull to discard other occurences of document having the same value as the field setted as a distinct attribute.

The value of a field whose attribute is set as a distinct attribute will always be unique in the returned documents.

### II. Motivation

The new search engine called Milli no longer processes the distinct attribute as the current MeilliSearch. This specification makes it possible to state that Milli will be as backwards compatible as possible with the v0.20 in the usage of API interfaces but also in the expected search results.

### III. Additional Materials

#### Algolia

Algolia distinct feature is based on one attribute, as defined in `attributeForDistinct`. It is possible to limit the number of returned records that contains the same attribute value. It is done at indexing time.

Algolia distinct functionality enables deduplication and aggregation by allowing a numeric value for the defined distinct attribute.

| Behavior       | Distinct value | Description                                                                                       |
|----------------|----------------|---------------------------------------------------------------------------------------------------|
| de-duplication | N = 1          | Used to remove similar records from the search result. Only the most relevant record is returned. |
| grouping       | N > 1          | N records containing the same value for the distinct attribute will be returned.                  |

> `distinct` is silently ignored at query time if `attibuteForDistinct` is not defined. It is not mandatory but possible to set `distinct` at indexing time via the index settings endpoint.
>
> It is possible to disable distinct feature at query time, by giving a falsy value (`false` or `O`) for the distinct parameter. Giving `true` is equivalent to `1`.

#### TypeSense

TypeSense distinct feature uses `group_by` and `group_limit` to achieve de-duplication and grouping at query time.

> A field that is setted as a parameter of group_by must be previously faceted.

| Parameter   | Description                                                                                                                                                                                                                                   |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| group_by    | It is possible to aggregate records into groups by setting multiple fields separated by a comma. E.g. group_by=country,company_name                                                                                                           |
| group_limit | Control the maximum number of top records returned for groups. By default, TypeSense `group_limit` parameter is set to 3.                                                                                                                     |

Using group_by add a nested structure in the search result. Buckets are returned in `grouped_hits` field.

#### ElasticSearch

Elasticsearch does not provide this functionality directly. Nor a keyword or an operator exists to get de-duplicated or grouped results.

It is difficult to find the information in the official documentation. A lot of people are asking about the distinct feature on StackOverflow or ElasticSearch forums.

By specifying the search with terms or composite aggregations it is possible to have buckets fed by documents that have the same values on a specified field.

E.g Terms aggregation
```json
{
    "aggs": {
        "distinct_colors": {
            "terms": {
                "field": "color",
                "size": 1000
            }
        }
    }
}
```
The size parameter can be set to define how many term buckets should be returned out of the overall terms list. Term aggregations by default return 10 buckets only.

Composite aggregation allows to paginate over buckets containing a lot of values.

> Cardinality aggregation can calculates the count of distinct values for a field.

It is also possible to use the keyword `DISTINCT` from the SQL access feature from X-Pack. However the functionality only allows to return tabular data.

### IV.Explanation

Let's say that we have 2 documents with the same `product_id`. Each document exists to materialize the color variation.

```json
{
    "hits": [
        {
            "colors": "red",
            "id": 1,
            "label": "t-shirt",
            "product_id": 1
        },
        {
            "colors": "black",
            "id": 2,
            "label": "t-shirt",
            "product_id": 1
        }
    ],
    "offset": 0,
    "limit": 20,
    "nbHits": 2,
    "exhaustiveNbHits": false,
    "processingTimeMs": 1,
    "query": "t-shirt"
}
```

Without setting `product_id` as a distinct attribute, a search with `t-shirt` as a query will return the two documents.

It can be useful to display one product per color variation as a search result for example. But as your number of products variations grows over time, you might want to display only one result as a top search result, mostly for UI concerns.

It's in this case that the distinct attribute finds all its interest.

Setting `product_id` as a distinct attribute will discard all others documents having the same value for `product_id` from the search result.

#### Distinct Attribute in v0.20

> MeiliSearch accepts only one value which is the document attribute that needs to be de-duplicated from the other document sharing the same attribute value.

It is possible to configure the distinct attribute using two endpoints. [Update All Settings](https://docs.meilisearch.com/reference/api/settings.html#update-settings) and the [distinct attribute](https://docs.meilisearch.com/reference/api/distinct_attribute.html#update-distinct-attribute) endpoint.

Given this setting :
```json
{
    "distinctAttribute": "product_id"
}
```

MeiliSearch will return de-duplicated hits by `product_id`.

```json
{
    "hits": [
        {
            "colors": "red",
            "id": 1,
            "label": "t-shirt",
            "product_id": 1
        }
    ],
    "offset": 0,
    "limit": 20,
    "nbHits": 1,
    "exhaustiveNbHits": false,
    "processingTimeMs": 1,
    "query": "\"t-shirt\""
}
```

It returns the top most relevant document by discarding all the others.

For a search with `"q": "black"` as parameter, MeiliSearch returns:

```json
{
    "hits": [
        {
            "colors": "black",
            "id": 2,
            "label": "t-shirt",
            "product_id": 1
        }
    ],
    "offset": 0,
    "limit": 20,
    "nbHits": 1,
    "exhaustiveNbHits": false,
    "processingTimeMs": 0,
    "query": "\"black\""
}
```

#### Distinct Attribute in 0.21 (Milli)

Milli shoud be identical as v.0.20 concerning API endpoints and returned results.

### V. Impact on documentation
N/A

### VI. Impact on SDKs
N/A

## 2. Technical Aspects

### Abstract

To apply the distinct behavior, the search engine needs to create a database. This database is the same as we create to apply facets and filters on an attribute. However, the users will not pass the attribute into `attributesForFaceting` when setting a distinct attribute. It means the search engine must create this database for the related attribute.

Following this example, it also means the search engine would be technically able to apply filters and facet distribution on the distinct attribute, however, we should prevent this. To avoid confusion, the search engine should prevent the users to execute a filter or get facet distribution on the distinct attribute. Only the distinct capability should be available for this field.

If the user wants to filter on that attribute, he will have to add it in `attributesForFaceting` as well.

Using this new data structure allows interesting future possibilities.

## 3. Future possibilities

> These ideas will be materialized as feature-proposals in the product team backlog. Keep in mind that this specification is related to v0.21.

### Grouping
#### Distinct count to enable grouping feature

Since MeiliSearch can only de-duplicate documents matching the distinct attribute, it would be interesting to be able to set the number of topmost relevant documents before discarding the others.

`groupLimit` could be applied at search time to fit users needs.

#### Distinct on multiple fields

> It probably require to add a nested structure to return hits for each groups with clarity.

##### Indexing time

Given this distinct setting:

```json
{
    "distinctAttributes": ["colors", "size"]
}
```

##### Search time

We could add a `groupBy` like parameter to let the user choose a specific grouping behavior:

```json
{
    "groupBy": ["colors"]
}
```

or

```json
{
    "groupBy": ["size", "colors"]
}
```

> It can be used in combination with groupLimit to define the topmost relevant documents before discarding the others for each group.
