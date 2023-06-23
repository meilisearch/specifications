# Facet Search API

## 1. Summary

The facet search endpoint retrieves facet values from a query.

## 2. Motivation

When many values exist for a facet, end-users need to be able to discover non-show values they can select in order to refine their faceted search. Being able to search for values for a facet solves this need.

## 3. Functional Specification

Meilisearch exposes 1 route to perform facet search requests:

- POST `indexes/:index_uid/facet-search`

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

If a master key is used to secure a Meilisearch instance, the auth layer returns the following errors:

- 🔴 Accessing these routes without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing these routes with a key that does not have permissions (i.e. other than the master key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

`POST` HTTP verb errors:

- 🔴 Omitting the Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.

### 3.1. Facet Search Payload Parameters

| Field                                                 | Type                     | Required |
|-------------------------------------------------------|--------------------------|----------|
| [`facetName`](#311-facetName)                         | String                   | True     |
| [`facetQuery`](#312-facetQuery)                       | String                   | False    |

#### 3.1.1. `facetName`

- Type: String
- Required: True
- Default: `null`

`facetName` contains the facet name to search values on.

- 🔴 Omitting `facetName` returns a `missing_facet_search_facet_name`(0061-error-format-and-definitions.md#missing_facet_search_facet_name) error.
- 🔴 Sending a value with a different type than `String` for `facetName` returns a [missing_facet_search_facet_name](0061-error-format-and-definitions.md#minvalid_facet_search_facet_name) error.
- 🔴 Sending a field not defined as a `filterableAttributes` for `facetname` returns an [invalid_facet_search_facet_name](0061-error-format-and-definitions.md#invalid_facet_search_facet_name) error.

#### 3.1.2. `facetQuery`

- Type: String
- Required: False
- Default: `null`

`facetQuery` contains the terms to search within the facet values contained in the index documents.

> When `facetQuery` isn't specified, Meilisearch performs a **placeholder search**. A placeholder search returns all facet values for the searched facet, limited by the [maxValuesPerFacet index setting](157-faceting-setting-api.md#311-maxvaluesperfacet).
> It supports **prefix search**
> It supports **typo tolerance**. Modifying the [typo tolerance index setting](0117-typo-tolerance-setting-api.md) impacts the behavior of the facet search.

> Meilisearch only considers one term for `facetQuery`. e.g if "harry potter" is typed the facet search consider "harry potter" as one single term.

- 🔴 Sending a value with a different type than `String` or `null` for `facetQuery` returns an [invalid_facet_search_facet_query](0061-error-format-and-definitions.md#invalid_facet_search_facet_query) error.

### 3.2. Additional Search Parameters

Additional search parameters can be injected to the facet search query. If additional search parameters are set, the method will return facet values that both:

- Match the facet query
- Are contained in the records matching the additional search parameters (`q`, `filter`, `matchingStrategy`)

| Field                                                          | Type                     | Required |
|----------------------------------------------------------------|--------------------------|----------|
| [`q`](0118-search-api.md#311-q)                                | String                   | False    |
| [`filter`](0118-search-api.md#312-filter)                      | String                   | False    |
| [`matchingStrategy`](0118-search-api.md#3117-matchingstrategy) | String                   | False    |

- 🔴 These properties are validated as stipulated in their original specification and return the associated errors.

> Non-listed search parameters are ignored if specified for the API call; no error is raised.

### 3.3. Search Response Properties

| Field                                           | Type            | Required |
|-------------------------------------------------|-----------------|----------|
| [`facetHits`](#331-facetHits)                   | Array[FacetHit] | True     |
| [`facetQuery`](#332-facetQuery)                 | String          | True     |
| [`processingTimeMs`](#333-processingtimems)     | Integer         | True     |

#### 3.3.1. `facetHits`

- Type: Array[FacetHit]
- Required: True

Results of the facet search query as an array of facet hit.

> FacetHit object represents a matched facet value for a facet search.

##### 3.3.1.1. `facetHit` Object Properties

`facetHit` sorting is controlled by the [sortFacetValuesBy index setting](157-faceting-setting-api.md#312-sortfacetvaluesby).

| Field                                        | Type    | Required |
|----------------------------------------------|---------|----------|
| [`value`](#32112-formatted)                  | String  | True     |
| [`count`](#32111-geodistance)                | Integer | True     |

###### 3.3.1.1.1. `value`

- Type: String
- Required: True

The facet value being matched.

###### 3.3.1.1.2. `count`

- Type: Integer
- Required: True

The number of document containing the matched facet value.

#### 3.3.2. `facetQuery`

- Type: Integer
- Required: True

Facet query originating the response.

#### 3.3.3. `processingTimeMs`

- Type: Integer
- Required: True

Processing time of the facet search query in **milliseconds**.

## 2. Technical Details

### 2.1 Facet Containing Numbers Limitation

- Numbers on the Meilisearch side are represented as [float64](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) internally. When a user sends a number in a JSON document, it can be written in different ways but end up being the same number i.e. 12e4, 120000, 120000.0. There is an issue as to how to let the user find it if it is not written in the same way.
- Another issue is that float64 numbers sometimes lack precision. You can write a number in your document i.e. 12.6758493, search for this exact same number and don’t find it. The reason is that numbers are stored in a different way and an exact search will not find them.

## 3. Future Possibilities

- Add a property to customize the sort of returned facet values
- Highlight parts matching the facet value from the facet query
- Support for number search