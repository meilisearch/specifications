- Title: Search API

# Search API

## 1. Summary

The search endpoints permit to retrieve documents within an index that are the most relevant given a set of parameters forming a search request.

## 2. Motivation
N/A

## 3. Functional Specification

Meilisearch exposes 2 routes to perform search requests:

- GET `indexes/:index_uid/search`
- POST `indexes/:index_uid/search`

- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

If a master key secures the Meilisearch instance, the auth layer returns the following errors:

- 🔴 Accessing these routes without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

`POST` HTTP verb errors:

- 🔴 Omitting the Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.

### 3.1. Search Payload Parameters

| Field                                                 | Type                      | Required |
|-------------------------------------------------------|---------------------------|----------|
| [`q`](#311-q)                                         | String                    | False    |
| [`filter`](#312-filter)                               | Array of String - String  | False    |
| [`sort`](#313-sort)                                   | Array of String - String  | False    |
| [`facetsDistribution`](#314-facetsdistribution)       | Array of String - String  | False    |
| [`limit`](#315-limit)                                 | Integer                   | False    |
| [`offset`](#316-offset)                               | Integer                   | False    |
| [`attributesToRetrieve`](#317-attributestoretrieve)   | Array of String - String  | False    |
| [`attributesToHighlight`](#318-attributestohighlight) | Array of String - String  | False    |
| [`highlightPreTag`](#319-highlightpretag)             | String                    | False    |
| [`highlightPostTag`](#3110-highlightposttag)          | String                    | False    |
| [`attributesToCrop`](#3111-attributestocrop)          | Array of String - String  | False    |
| [`cropLength`](#31112-croplength)                     | Integer                   | False    |
| [`cropMarker`](#31113-cropmarker)                     | String                    | False    |
| [`matches`](#31114-matches)                           | Boolean                   | False    |

#### 3.1.1. `q`

- Type: String
- Required: False
- Default: `null`

`q` contains the terms to search within the index documents.

- 🔴 Sending a value with a different type than `String` or `null` for `q` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

> When q isn't specified, Meilisearch performs a **placeholder search**. A placeholder search returns all searchable documents in an index, modified by any search parameters used and sorted by that index's custom ranking rules. If the index has no sort search parameter or custom ranking rules, the results are returned in the order of their internal database position.

> Meilisearch only considers the first ten words of any given search query to deliver a fast search-as-you-type experience.

> `q` supports the [Phrase Query](0043-phrase-query.md) expression.

#### 3.1.2. `filter`

- Type: Array of String (POST) | String (POST/GET)
- Required: False
- Default: `[]|null`

`filter` contains a filter expression written as a string or an array of strings. It permits to refine search results.

Attributes used as filter criteria must be added to the `filterableAttributes` list of an index settings. See [Filterable Attributes Setting API](0123-filterable-attributes-setting-api.md).

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `filter` returns an [invalid_filter](0061-error-format-and-definitions.md#invalid_filter) error.
- 🔴 Sending an invalid syntax for `filter` returns an [invalid_filter](0061-error-format-and-definitions.md#invalid_filter) error.
- 🔴 Sending a field not defined as a `filterableAttributes` for `filter` returns an [invalid_filter](0061-error-format-and-definitions.md#invalid_filter) error.

> See [Filter And Facet Behavior](0027-filter-and-facet-behavior.md).

#### 3.1.3. `sort`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

`sort` contains a sort expression written as a string or an array of strings. It sorts the search esults at query time according to the specified attributes and indicated order.

Attributes used as sort criteria must be added to the `sortableAttributes list of an index settings. See [Sortable Attributes Setting API](0123-sortable-attributes-setting-api.md).

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `sort` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending an invalid syntax for `sort` returns an [invalid_sort](0061-error-format-and-definitions.md#invalid_sort) error.
- 🔴 Sending a field not defined as a `sortableAttributes` for `sort` returns an [invalid_sort](0061-error-format-and-definitions.md#invalid_sort) error.

> See [Sort](0055-sort.md)

#### 3.1.4. `facetsDistribution`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

`facetsDistribution` permits to specify facets to be computed for the current search query.

It returns the number of documents matching the current search query for each specified facet.

This parameter can take two values:

- An array of attributes: `facetsDistribution=["attributeA", "attributeB", …]`
- An asterisk `"*"` — this returns a count for all facets present in `filterableAttributes`

Attributes used in `facetsDistribution` must be added to the `filterableAttributes` list of an index settings. See [Filterable Attributes Setting API](0123-filterable-attributes-setting-api.md).

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `facetsDistribution` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.
- 🔴 Sending a field not defined as a `filterableAttributes` for `facetsDistribution` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

> See [Filter And Facet Behavior](0027-filter-and-facet-behavior.md)

#### 3.1.5. `limit`

- Type: Integer
- Required: False
- Default: `20`

Sets the maximum number of documents to be returned for the search query.

- 🔴 Sending a value with a different type than `Integer` or `null` for `limit` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.6. `offset`

- Type: Integer
- Required: False
- Default: `0`

Sets the starting point in the search results, effectively skipping over a given number of documents.

- 🔴 Sending a value with a different type than `Integer` or `null` for `offset` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.7. `attributesToRetrieve`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

Configures which attributes will be retrieved in the returned documents.

If no value is specified, `attributesToRetrieve` uses the `displayedAttributes` index setting, which by default contains all attributes found in the documents.

> If an attribute is missing from `displayedAttributes` index setting, `attributesToRetrieve` silently ignore it, and the field doesn't appear in the returned search results.

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `attributesToRetrieve` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.8. `attributesToHighlight`

- Type: Array[String](POST)|String(GET)
- Required: False
- Default: `[]|null`

Highlights document parts matching query terms in the `q` search parameter for the specified attributes.

Search results include a `_formatted` object containing the highlighted terms when this parameter is defined. See [3.2.1.1.2. `_formatted`](#32112-formatted) section.

Highlighted parts are surrounded by the [`highlightPreTag`](#319-highlightpretag) and [`highlightPostTag`](#3110-highlightposttag) parameter.

If `"*"` is provided as a value: e.g. `"attributesToHighlight":["*"]` all the attributes present in `attributesToRetrieve` will be assigned to `attributesToHighlight`.

`attributesToHighlight` only works on values of the following types: `string`, `number`, `array`, `object`.

- 🔴 Sending a value with a different type than `Array[String]`(POST), `String`(GET) or `null` for `attributesToHighlight` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 3.1.8.1. searchableAttributes

Attributes not defined in the `searchableAttributes` index setting are also highlighted if assigned to `attributesToHighlight`.

##### 3.1.8.2. stopWords

Attributes defined in the `stopWords` index setting are also highlighted if matched.

#### 3.1.9. `highlightPreTag`

- Type: String
- Required: False
- Default: `"<em>"`

Specify the tag to put **before** the matched query terms.

This parameter is taken into account when `attributesToHighlight` is specified. See [3.1.8. `attributesToHighlight`](#318-attributestohighlight) section.

- 🔴 Sending a value with a different type than `String` for `highlightPreTag` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.10. `highlightPostTag`

- Type: String
- Required: False
- Default: `"</em>"`

Specify the tag to put **after** the matched query terms.

This parameter is taken into account when `attributesToHighlight` is specified. See [3.1.8. `attributesToHighlight`](#318-attributestohighlight) section.

- 🔴 Sending a value with a different type than `String` for `highlightPostTag` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.11. `attributesToCrop`

- Type: Array[String]|String
- Required: False
- Default: `[]|null`

Defines document attributes to be cropped. Cropped attributes have their values shortened around query terms.

The number of words contained in the cropped value is defined by the `cropLength` parameter. See [3.1.1.12. `cropLength`](#31112-croplength) section.

The value of `cropLength` can be customized per attribute. See [3.1.11.2. Custom `cropLength` Defined Per Cropped Attribute](#31112-custom-croplength-defined-per-attribute) section.

The engine adds a marker by default in front of and/or behind the part selected by the cropper. This marker is customizable. See [3.1.1.13. `cropMarker`](#31113-cropmarker) section.

Search results include a `_formatted` object containing the cropped attributes representation when this parameter is defined. See [3.2.1.1.2. `_formatted`](#32112-formatted) section.

If `"*"` is provided as a value: `attributesToCrop=["*"]` all the attributes present in `attributesToRetrieve` will be automatically assigned to `attributesToCrop`.

- 🔴 Sending a value with a different type than `Array[String]`(POST), `String`(GET) or `null` for `attributesToCrop` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 3.1.11.2. Custom `cropLength` Defined Per Attribute.

Optionally, indicating a custom crop length for any of the listed attributes is possible:

`"attributesToCrop":["attributeNameA:15", "attributeNameB:30"]`

A custom crop length set in this way has priority over the `cropLength` parameter.

##### 3.1.11.3. searchableAttributes

As for `attributesToHighlight`, attributes not defined in the `searchableAttributes` index setting are also cropped if assigned to `attributesToCrop`.

#### 3.1.1.12. `cropLength`

- Type: Integer
- Required: False
- Default: `10`

Sets the total number of **words** to keep around the matched part of an attribute specified in the `attributesToCrop` parameter.

If `attributesToCrop` is not configured, `cropLength` has no effect on the returned results.

- 🔴 Sending a value with a different type than `Integer` or `null` for `cropLength` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

#### 3.1.1.13. `cropMarker`

- Type: String
- Required: False
- Default: `"…"` (U+2026)

Sets the crop marker to apply before and/or after the query terms that have matched within an attribute defined in `attributesToCrop`. See [3.1.11. `attributesToCrop`](#3111-attributestocrop) section.

The specified crop marker is applied by following rules. See [3.1.1.13.1. Applying `cropMarker`](#311131-applying-cropmarker) section.

Specifying `cropMarker` to `""` or `null` implies that no marker will be applied to the cropped attribute.

- 🔴 Sending a value with a different type than `String` or `null` for `cropMarker` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

##### 3.1.1.13.1. Applying `cropMarker`

###### 3.1.1.13.1.1. Single Crop

The cropping algorithm only keeps a single cropped part. It means that if any query terms could have been matched outside the number of words specified by `cropLength`. They will not be included into the resulting cropped part.

Given a document

```json
{
    "id": 0,
    "description": "Every year, food waste is about a third of our food. A lot of this food waste is cereals, fruit and vegetables. In the UK, more than 97% of food waste ends up in a landfill site. That's a lot!"
}
```

If a search query is defined by `"attributesToCrop": ["description:15"]` and `"q": "food waste"`.

The cropped result is

`"Every year, food waste if about a third of our food. A lot of this…"`

###### 3.1.1.13.1.2. Matched Part To Be Cropped

The cropping algorithm tries to match the window with the highest density of query terms within the `cropLength` limit. Then it will pick the window that contains the more ordered query terms.

If two window have the same density, it chooses the first one within the attribute to be cropped.

###### 3.1.1.13.1.3. Positioning Markers

If the cropped part contains the beginning of the attribute to be cropped, the `cropMarker` is not placed to the left of the cropped part.

If the cropped part contains the end of the attribute to be cropped, the `cropMarker`is not placed to the right of the cropped part.

#### 3.1.1.14. `matches`

- Type: Boolean
- Required: False
- Default: `false`

Adds a `_matchesInfo` object to the search response that contains the location of each occurrence of queried terms across all fields. This is useful when more control is needed than offered by the built-in highlighting/cropping features.

- 🔴 Sending a value with a different type than `Boolean` or `null` for `matches` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

### 3.2. Search Response Properties

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

#### 3.2.1. `hits`

- Type: Array[Hit]
- Required: True

Results of the query as an array of documents.

> The search parameters `attributesToRetrieve` influence the returned payload for a document as a search result. See 1.2.1.7 `attributesToRetrieve` section.

> Hit object represents a matched document as a search result. It can host special properties, see next section.

##### 3.2.1.1. `hits` Special Properties

| Field                   | Type        | Required |
|-------------------------|-------------|----------|
| _geoDistance            | Integer     | False    |
| _formatted              | Object      | False    |
| _matchesInfo            | Object      | False    |

###### 3.2.1.1.1. `_geoDistance`

- Type: Integer
- Required: False

Search queries using `_geoPoint` will always include a `_geoDistance` field containing the distance in meters between the document location and the `_geoPoint`.

> See [GeoSearch](0059-geo-search.md)

###### 3.2.1.1.2. `_formatted`

- Type: Object
- Required: False

Object containing the cropped/highlighted values of the fields specified in `attributesToHighlight` or/and `attributesToCrop`.

###### 3.2.1.1.2.1 Example

Given the following search query payload

```json
{
    "q": "Summer T-Shirt",
    "attributesToRetrieve": ["*"],
    "attributesToHighlight": ["*"],
    "attributesToCrop": ["description:10"]
}
```

It returns

```json
{
    "hits": [
        {
            "name": "T-shirt",
            "id": 4,
            "description": "Super cool T-Shirt for the summer. Soft, breathable jersey T-shirt fabric. Body: 100% Cotton.",
            "_formatted": {
                "name": "<em>T</em>-<em>shirt</em>",
                "id": "4",
                "description": "Super cool <em>T</em>-<em>Shirt</em> for the <em>summer</em>. Soft, breathable…"
            }
        }
    ],
    "nbHits": 1,
    "exhaustiveNbHits": false,
    "query": "Summer T-Shirt",
    "limit": 20,
    "offset": 0,
    "processingTimeMs": 6
}
```

###### 3.2.1.1.3. `_matchesInfo`

- Type: Object
- Required: False

Contains the location of each occurrence of queried terms across all fields. The `_matchesInfo` object is added to a document when `matches` search parameter is specified to true.

The beginning of a matching term within a field is indicated by `start`, and its `length` by length.

`start` and `length` are measured in bytes and not the number of characters. For example, `ü` represents two bytes but one character.

> See 1.2.1.11 `matches` section.

#### 3.2.2. `limit`

- Type: Integer
- Required: True

Returns the `limit` search parameter used for the query.

> See 1.2.1.5 `limit` section.

#### 3.2.3. `offset`

- Type: Integer
- Required: True

Returns the `offset` search parameter used for the query.

> See 1.2.1.6 `offset` section.

#### 3.2.4. `nbHits`

- Type: Integer
- Required: True

Returns the total number of candidates for the search query.

#### 3.2.5. `exhaustiveNbHits`

- Type: Boolean
- Required: True

Whether `nbHits` is exhaustive.

> Always return `false`.

#### 3.2.6. `facetsDistribution`

- Type: Object
- Required: False

Added to the search response when `facetsDistribution` is set for a search query. It contains the number of remaining candidates for each specified facet in the `facetsDistribution` search parameter.

> See 1.2.1.4 `facetsDistribution` section.
> See [Filter And Facet Behavior](0027-filter-and-facet-behavior.md)

#### 3.2.7. `exhaustiveFacetsCount`

- Type: Boolean
- Required: False

Whether `facetsDistribution` count is exhaustive. The field `exhaustiveFacetsCount` is added when `facetsDistribution` is set as a search parameter.

> Always returns `false`.

#### 3.2.8. `processingTimeMs`

- Type: Integer
- Required: True

Processing time of the search query in milliseconds.

#### 3.2.9. `query`

- Type: String
- Required: True
- Default: `""`

Query originating the response. Equals to the `q` search parameter.

> See 1.2.1.1 `q` section.

## 2. Technical Details
n/a

## 3. Future Possibilities
- Add dedicated errors to replace `bad_request` error.
- Move `attributesToHighlight`, `highlightPreTag`, `highlightPostTag`, `attributesToCrop`, `cropLength` and `cropMarker` into a `formatter` objet.
- Expose the `formatter` resource as an index setting.