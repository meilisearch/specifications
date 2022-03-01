- Title: Search API
- Start Date: 2022-02-27

# Search API

## 1. Functional Specification

### 1.1. Summary

The search endpoints permit to retrieve documents within an index that are the most relevant given a set of parameters forming a search query.

### 1.2. Explanation

Meilisearch exposes 2 routes to perform searches:

- GET `indexes/:index_uid/search`
- POST `indexes/:index_uid/search`

- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

If the instance is secured by a master-key, the auth layer returns the following errors:

- 🔴 Accessing these routes without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

`POST` HTTP verb errors:

- 🔴 Omitting the Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.

#### 1.2.1. Search payload parameters

| Field                   | Type                      | Required |
|-------------------------|---------------------------|----------|
| q                       | String                    | False    |
| filter                  | Array of String - String  | False    |
| sort                    | Array of String - String  | False    |
| facetsDistribution      | Array of String - String  | False    |
| limit                   | Integer                   | False    |
| offset                  | Integer                   | False    |
| attributesToRetrieve    | Array of String - String  | False    |
| attributesToHighlight   | Array of String - String  | False    |
| attributesToCrop        | Array of String - String  | False    |
| cropLength              | Integer                   | False    |
| matches                 | Boolean                   | False    |
| typoTolerance           | Object[typoTolerance]     | False    |

##### 1.2.1.1 `q`

- Type: String
- Required: False
- Default: `null`

`q` contains the terms to search within the index documents.

- 🔴 Sending a value with a different type than `String` or `null` for `q` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

> When q isn't specified, Meilisearch performs a **placeholder search**. A placeholder search returns all searchable documents in an index, modified by any search parameters used and sorted by that index's custom ranking rules. If the index has no sort or custom ranking rules, the results are returned in the order of their internal database position.

> Meilisearch only considers the first ten words of any given search query to deliver a fast search-as-you-type experience.

> `q` supports [Phrase Query](0043-phrase-query.md) expression.

##### 1.2.1.2 `filter`

- Type: Array of String (POST) | String (POST/GET)
- Required: False
- Default: `[]|null`

`filter` contains a filter expression written as a string or an array of strings. It permits to refine search results.

Attributes used as filter criteria must be added to the `filterableAttributes` list of an index settings.

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `filter` returns an [invalid_filter](0061-error-format-and-definitions.md#invalid_filter) error.
- 🔴 Sending an invalid syntax for `filter` returs an [invalid_filter](0061-error-format-and-definitions.md#invalid_filter) error.
- 🔴 Sending a field not defined as a `filterableAttributes` for `filter` returns an [invalid_filter](0061-error-format-and-definitions.md#invalid_filter) error.

> See [Filter And Facet Behavior](0027-filter-and-facet-behavior.md)

##### 1.2.1.3 `sort`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

`sort` contains a sort expression written as a string or an array of strings. It permits to sorts search results at query time according to the specified attributes and indicated order.

Attributes used as sort criteria must be added to the `sortableAttributes list of index settings.

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `sort` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending an invalid syntax for `sort` returns an [invalid_sort](0061-error-format-and-definitions.md#invalid_sort) error.
- 🔴 Sending a field not defined as a `sortableAttributes` for `sort` returns an [invalid_sort](0061-error-format-and-definitions.md#invalid_sort) error.

> See [Sort](0055-sort.md)

##### 1.2.1.4 `facetsDistribution`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

`facetsDistribution` permits to specify facets to be computed for the current search query.

It returns the number of documents matching the current search query for each specified facet.

This parameter can take two values:

- An array of attributes: `facetsDistribution=["attributeA", "attributeB", …]`
- An asterisk `"*"` — this returns a count for all facets present in `filterableAttributes`

Attributes used in `facetsDistribution` must be added to the `filterableAttributes` list of an index settings.

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `facetsDistribution` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending a field not defined as a `filterableAttributes` for `facetsDistribution` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

> See [Filter And Facet Behavior](0027-filter-and-facet-behavior.md)

##### 1.2.1.5 `limit`

- Type: Integer
- Required: False
- Default: `20`

Sets the maximum number of documents to be returned by the current search query.

- 🔴 Sending a value with a different type than `Integer` or `null` for `limit` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 1.2.1.6 `offset`

- Type: Integer
- Required: False
- Default: `0`

Sets the starting point in the search results, effectively skipping over a given number of documents.

- 🔴 Sending a value with a different type than `Integer` or `null` for `offset` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 1.2.1.7 `attributesToRetrieve`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

Configures which attributes will be retrieved in the returned documents.

If no value is specified, `attributesToRetrieve` uses the `displayedAttributes` list, which by default contains all attributes found in the documents.

> If an attribute has been removed from `displayedAttributes` index settings, `attributesToRetrieve` will silently ignore it and the field will not appear in the returned documents.

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `attributesToRetrieve` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 1.2.1.8 `attributesToHighlight`

- Type: Array[String](POST)|String(GET)
- Required: False
- Default: `[]|null`

Highlights matching query terms in the specified attributes by enclosing them in `<em>` tags.

When this parameter is set, returned documents include a `_formatted` object containing the highlighted terms.

If `"*"` is provided as a value: `attributesToHighlight=["*"]` all the attributes present in `attributesToRetrieve` will be assigned to `attributesToHighlight`.

- 🔴 Sending a value with a different type than `Array[String]`(POST), `String`(GET) or `null` for `attributesToHighlight` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

> See [_Formatted Field Behavior](0039-_formatted-field-behavior_.md)

##### 1.2.1.9 `attributesToCrop`

- Type: Array[String]|String
- Required: False
- Default: `[]|null`

Crops the selected attributes' values in the returned results to the length indicated by the `cropLength` parameter.

When this parameter is set, returned documents include a `_formatted` object containing the cropped terms.

Optionally, indicating a custom crop length for any of the listed attributes is possible: `attributesToCrop=["attributeNameA:25", "attributeNameB:150"]`. The custom crop length set in this way has priority over the `cropLength` parameter.

Instead of supplying individual attributes, it is possible to provide `["*"]` as a value: `attributesToCrop=["*"]`. This will crop the values of all attributes present in `attributesToRetrieve`.

- 🔴 Sending a value with a different type than `Array[String]`(POST), `String`(GET) or `null` for `attributesToCrop` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

> See [_Formatted Field Behavior](0039-_formatted-field-behavior_.md)

##### 1.2.1.10 `cropLength`

- Type: Integer
- Required: False
- Default: `200`

Configures the number of characters to keep on each side of the matching query term when using the `attributesToCrop` parameter.

If `attributesToCrop` is not configured, `cropLength` has no effect on the returned results.

- 🔴 Sending a value with a different type than `Integer` or `null` for `cropLength` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 1.2.1.11 `matches`

- Type: Boolean
- Required: False
- Default: `false`

Adds a `_matchesInfo` object to the search response that contains the location of each occurrence of queried terms across all fields. This is useful when more control is needed than offered by the built-in highlighting/cropping features.

- 🔴 Sending a value with a different type than `Boolean` or `null` for `matches` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 1.2.1.12 `typoTolerance`

- Type: Object[typoTolerance]
- Required: False
- Default: `null`

Override typo tolerance settings at search time.

The attributes of the `typoTolerance` object are not mandatory at search time.

> See [Typo Tolerance](0117-typo-tolerance-api.md)

#### 1.2.2. Search response

| Field                   | Type                         | Required |
|-------------------------|------------------------------|----------|
| hits                    | Array[Hit]                   | True     |
| limit                   | Integer                      | True     |
| offset                  | Integer                      | True     |
| nbHits                  | Integer                      | True     |
| exhaustiveNbHits        | Boolean                      | True     |
| facetsDistribution      | Object                       | False    |
| exhaustiveFacetsCount   | Boolean                      | False    |
| processingTimeMs        | Integer                      | True     |
| query                   | String                       | True     |

##### 1.2.2.1 `hits`

- Type: Array[Hit]
- Required: True

Results of the query as an array of documents.

> The search parameters `attributesToRetrieve` influence the returned payload for a document as a search result. See 1.2.1.7 `attributesToRetrieve` section.

> A Hit object that represents a document within the search results can host special attributes. See 1.2.2.9 `hits` special fields section.

##### 1.2.2.2 `limit`

- Type: Integer
- Required: True

Gives the `limit` search parameter used for the query.

> See 1.2.1.5 `limit` section.

##### 1.2.2.3 `offset`

- Type: Integer
- Required: True

Gives the `offset` search parameter used for the query.

> See 1.2.1.6 `offset` section.

##### 1.2.2.4 `nbHits`

- Type: Integer
- Required: True

Returns the total number of candidates for the search query.

##### 1.2.2.5 `exhaustiveNbHits`

- Type: Boolean
- Required: True

Whether `nbHits` is exhaustive.

> Always return `false`.

##### 1.2.2.6 `facetsDistribution`

- Type: Object
- Required: False

Added to the search response when `facetsDistribution` is set for a search query. It contains the number of remaining candidates for each specified facet in the `facetsDistribution` search parameter.

> See 1.2.1.4 `facetsDistribution` section.
> See [Filter And Facet Behavior](0027-filter-and-facet-behavior.md)

##### 1.2.2.7 `exhaustiveFacetsCount`

- Type: Boolean
- Required: False

Whether `facetsDistribution` count is exhaustive. The field `exhaustiveFacetsCount` is added when `facetsDistribution` is set as a search parameter.

> Always returns `false`.

##### 1.2.2.7 `processingTimeMs`

- Type: Integer
- Required: True

Processing time of the search query in milliseconds.

##### 1.2.2.8 `query`

- Type: String
- Required: True
- Default: `""`

Query originating the response. Equals to the `q` search parameter.

> See 1.2.1.1 `q` section.

##### 1.2.2.9 `hits` special fields

| Field                   | Type        | Required |
|-------------------------|-------------|----------|
| _geoDistance            | Integer     | False    |
| _formatted              | Object      | False    |
| _matchesInfo            | Object      | False    |

###### 1.2.2.9.1 `_geoDistance`

- Type: Integer
- Required: False

Search queries using `_geoPoint` will always include a `_geoDistance` field containing the distance in meters between the document location and the `_geoPoint`.

> See [GeoSearch](0059-geo-search.md)

###### 1.2.2.9.2 `_formatted`

- Type: Object
- Required: False

Object containing the cropped/highlighted values of the fields specified in `attributesToHighlight` or/and `attributesToCrop`.

> See 1.2.1.8 `attributesToHighlight` section and 1.2.1.9 `attributesToCrop` section.

###### 1.2.2.9.3 `_matchesInfo`

- Type: Object
- Required: False

Contains the location of each occurrence of queried terms across all fields. The `_matchesInfo` object is added to a document when `matches` search parameter is specified to true.

The beginning of a matching term within a field is indicated by start, and its length by length.

> See 1.2.1.11 `matches` section.

## 2. Technical Details
n/a

## 3. Future Possibilities
- Add dedicated errors to replace `bad_request` error.