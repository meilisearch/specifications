# Search API

## 1. Summary

The search endpoints retrieve documents from an index. Their returned documents are considered relevant based on the settings of the index and the provided search parameters.

## 2. Motivation
N/A

## 3. Functional Specification

Meilisearch exposes 2 routes to perform search requests:

- GET `indexes/:index_uid/search`
- POST `indexes/:index_uid/search`

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

### 3.1. Search Payload Parameters

| Field                                                 | Type                     | Required |
|-------------------------------------------------------|--------------------------|----------|
| [`q`](#311-q)                                         | String                   | False    |
| [`filter`](#312-filter)                               | Array of String - String | False    |
| [`sort`](#313-sort)                                   | Array of String - String | False    |
| [`facets`](#314-facets)                               | Array of String - String | False    |
| [`limit`](#315-limit)                                 | Integer                  | False    |
| [`offset`](#316-offset)                               | Integer                  | False    |
| [`page`](#317-page)                                   | Integer                  | False    |
| [`hitsPerPage`](#318-hitsperpage)                     | Integer                  | False    |
| [`attributesToRetrieve`](#319-attributestoretrieve)   | Array of String - String | False    |
| [`attributesToHighlight`](#3110-attributestohighlight)| Array of String - String | False    |
| [`highlightPreTag`](#3111-highlightpretag)            | String                   | False    |
| [`highlightPostTag`](#3112-highlightposttag)          | String                   | False    |
| [`attributesToCrop`](#3113-attributestocrop)          | Array of String - String | False    |
| [`cropLength`](#3114-croplength)                      | Integer                  | False    |
| [`cropMarker`](#3115-cropmarker)                      | String                   | False    |
| [`showMatchesPosition`](#3116-showmatchesposition)    | Boolean                  | False    |
| [`matchingStrategy`](#3117-matchingStrategy)          | String                   | False    |


#### 3.1.1. `q`

- Type: String
- Required: False
- Default: `null`

`q` contains the terms to search within the index documents.

> When q isn't specified, Meilisearch performs a **placeholder search**. A placeholder search returns all searchable documents in an index, modified by any search parameters used and sorted by that index's custom ranking rules. If the index has no sort search parameter or custom ranking rules, the results are returned in the order of their internal database position.

> Meilisearch only considers the first ten words of any given search query to deliver a fast search-as-you-type experience.

> `q` supports the [Phrase Query](0043-phrase-query.md) expression.

- 🔴 Sending a value with a different type than `String` or `null` for `q` returns an [invalid_search_q](0061-error-format-and-definitions.md#invalid_search_q) error.

#### 3.1.2. `filter`

- Type: String (POST/GET) | Array of (String, Array of String) (POST)
- Required: False
- Default: `[]|null`

`filter` contains a filter expression written as a string or an array of (strings and array of strings). Its purpose is to refine search results by selecting documents that match the given filter and running the search query only on those documents.

Attributes used as filter criteria must be added to the `filterableAttributes` list of an index settings. See [Filterable Attributes Setting API](0123-filterable-attributes-setting-api.md).

- 🔴 Sending a value with a different type than `String`(GET/POST), `Array of (Array of String, String) (POST)`, or `null` for `filter` returns an [invalid_search_filter](0061-error-format-and-definitions.md#invalid_search_filter) error.
- 🔴 Sending an invalid syntax for `filter` returns an [invalid_search_filter](0061-error-format-and-definitions.md#invalid_search_filter) error.
- 🔴 Sending a field not defined as a `filterableAttributes` for `filter` returns an [invalid_search_filter](0061-error-format-and-definitions.md#invalid_search_filter) error.

##### 3.1.2.1. String Syntax

###### 3.1.2.1.1 Grammar

The grammar of the filter syntax is given below in pseudo-BNF form:
```
filter         = expression EOF
expression     = or
or             = and ("OR" WS+ and)*
and            = not ("AND" WS+ not)*
not            = ("NOT" WS+ not) | primary
primary        = "(" WS* expression WS* ")" | geoRadius | geoBoundingBox | in | condition | exists | not_exists | to
in             = attribute "IN" WS* "[" value_list "]"
condition      = attribute ("=" | "!=" | ">" | ">=" | "<" | "<=") value
exists         = attribute "EXISTS"
not_exists     = attribute "NOT" WS+ "EXISTS"
to             = attribute value "TO" WS+ value
value          = WS* ( word | singleQuoted | doubleQuoted) WS+
attribute      = value
value_list     = (value ("," value)* ","?)?
singleQuoted   = "'" single_quoted_string "'"
doubleQuoted   = "\"" double_quoted_string "\""
word           = ([a-zA-Z0-9] | "_" | "-" | ".")+
geoRadius      = "_geoRadius(" WS* float WS* "," WS* float WS* "," float WS* ")"
float          = [+-]?[0-9]*("."[0-9]+)?(("E"|"e") [+-]?[0-9]+)?
WS             = ' ' | '\t' | '\r' | '\n'
```
where `single_quoted_string` and `double_quoted_string` can contain anything except unescaped single quotes `'` and unescaped double quotes `"`, respectively. Quotes are escaped by a preceding backslash. For example: `"escaped \" double quote" "` and `escaped \' single quote`. If a backslash is not followed by the correct quote, it is kept in the string.

###### 3.1.2.1.2 Naming a filterable attribute

A filterable attribute can appear in a filter by its unquoted name if it only contains ascii alphanumeric characters, dots, hyphens, and underscores.

For example, each filter below selects the documents where the given filterable attribute (on the left side of the equal) is equal to a specific value (on the right side):

```
genres = "film"
genre.subgenre = "adventure"
_geo.lat = 1.23
101 = "abc"
```

If the filterable attribute is composed of multiple words or contains other characters, it must be quoted, either using single quotes or double quotes:

```
"place of birth" = Berlin
"∆" = 2.1
"Friend's name" = Albus
'opinion on "the best search engine"' = "meilisearch"
```

If the filterable attribute contains the same quote character that surrounds the attribute, then this quote character must be escaped by a preceding backslash:
```
'Friend\'s name' = Albus
```

##### 3.1.2.1.3 Naming the value of a filterable attribute

The grammar for the value of a filterable attribute is the same as the grammar for filterable attributes themselves.

##### 3.1.2.1.4 List of supported operators

- Equality: `attribute = value`
- Inequality: `attribute != value`
- Comparison:
    * `attribute < value`
    * `attribute <= value`
    * `attribute > value`
    * `attribute >= value`
    * `attribute value TO value`
- Exists:
    * `attribute EXISTS`
    * `attribute NOT EXISTS`
- In:
    * `attribute IN[value, value, etc.]`
    * `attribute NOT IN[value, value, etc.]`
- AND: `filter AND filter`
- OR: `filter OR filter`
- NOT: `NOT filter`
- GeoSearch: `_geoRadius(lat, lng, distance)`
- GeoSearch: `_geoBoundingBox([lat, lng], [lat, lng])`

###### 3.1.2.1.5 Equality

The equality operator, `=`, selects the documents for which:
1. the given filterable attribute exists; and
2. the attribute contains a value that is equal to a specific value

It is an infix operator that takes an attribute name on the left hand side and a value on the right hand side.

For example, given the documents:
```json
[{
    "id": 0,
    "size": 1
},
{
    "id": 1,
    "size": ["1", "L"]
},
{
    "id": 2,
}
{
    "id": 3,
    "size": "small"
    "shop_distance": 1.2e+5
}]
```
then the filter:
```
size = 1
```
will select the documents with ids `0` and `1`.

Note that there is no way to specify whether the value on the right hand side of the equality should be interpreted as a string or as a number. Meilisearch will always try to match both. And since unquoted values cannot contain the `+` character, it is in fact necessary to quote floating point numbers that have positive exponents:
```
shop_distance = "1.2e+5"
```
will select the document with id `3`.

Furthermore, there is no way to check whether an attribute has a value that is `null` or an array. An attribute whose value is `null` or an empty array is considered not to have any value and will therefore never be matched by an equality operator.

###### 3.1.2.1.6 Inequality

The inequality operator selects all documents that are not selected by the equality operator.
With the same documents given as examples to the equality operator, the following filter:
```
size != 1
```
will select the documents with ids `2` and `3`.

Note that `attribute != value` is equivalent to `NOT attribute = value`.

Furthermore, there is no way to write a filter to select documents which contain a value that is different than a given string or number. In the example above, `size != 1` did not select the document with id `1`, even though its `size` attribute contains the value `"L"`, which is different than `1`.

###### 3.1.2.1.7 Comparison

The comparison operators select the documents for which:
1. the filterable attribute exists; and
2. the attribute contains a number that satisfies the comparison

Note that the right hand side of the comparison must be a valid floating point number.

For example, with the documents:
```json
[{
    "id": 0,
    "size": [0, "small"]
    "colour": "blue"
},
{
    "id": 1,
    "size": 1
},
{
    "id": 2,
    "size": [2, 20]
}]
```
Then the following filters will select these documents:
```
size > 1      -> selects [1]
size >= 1     -> selects [1,2]
size < 2      -> selects [0,1]
size <= 2     -> selects [0,1,2]
size -1 TO 2  -> equivalent to size >= -1 AND size <= 2 -> selects [0,1,2]
```

And the following filters are invalid:
```
size > "small"
size "larga" TO "largz"
```

##### 3.1.2.1.8 Combining filter conditions

Multiple filters can be combined together using the operators `AND` and `OR`. These infix operators take two sub-filters as arguments.

The `AND` operator selects the documents that are selected by both subfilters at the same time. In other words, it is an intersection of the two sets of documents selected by the sub-filters.

The `OR` operator selects the documents that are selected by either operator. In other words, it is a union of the two sets of documents selected by the sub-filters.

Note that `AND` has a higher precedence than `OR`. Therefore, the following filter:
```
x = 1 AND y = 2 OR z = 3
```
will be interepreted as:
```
(x = 1 AND y = 2) OR (z = 3)
```

With the same documents given as examples for the comparison operators, the following filters will select these documents:
```
size = 0 OR size = 1                       -> selects [0,1]
size = 0 AND (size = 2 OR colour = "blue") -> selects []
size = 0 AND size = 2 OR colour = "blue"   -> selects [0]
size > 5 AND size < 5                      -> selects [2]
```

###### 3.1.2.1.9 Negating a filter

The negation operator, `NOT`, is used to select all documents that are not selected by a sub-filter. It is a prefix operator that takes one argument. Its precedence is higher than both `AND` and `OR`.

With the same documents given as examples for the comparison operators, the following filters will select these documents:
```
NOT size = 0                               -> selects [1,2]
NOT (size = 0 OR size = 1)                 -> selects [2]
NOT size = 0 OR size = 1                   -> selects [1,2]
NOT (size < 2 AND colour = "blue")         -> selects [1,2]
NOT size < 2 AND colour = "blue"           -> selects []
size = 0 OR NOT size = 2                   -> selects [0,1]
NOT (NOT size = 0)                         -> selects [0]
```

###### 3.1.2.1.10 Exists

The `EXISTS` operator selects the documents for which the filterable attribute exists, even if its value is `null` or an empty array. It is a postfix operator that takes an attribute name as argument.

The negated form of `EXISTS` can be written in two ways:
```
attribute NOT EXISTS
NOT attribute EXISTS
```
Both forms are equivalent. They select the documents for which the attribute does not exist.

For example, with the documents:
```json
[{
    "id": 0,
    "colour": []
},
{
    "id": 1,
    "colour": null
},
{
    "id": 2
}]
```
Then the filter `colour EXISTS` selects the document ids `[0,1]` while the filter `colour NOT EXISTS` or `NOT colour EXISTS` selects the document ids `[2]`.

###### 3.1.2.1.11 In

The `IN[..]` operator is a more concise way to combine equality operators. It is a postfix operator that takes an attribute name on the left hand side and an array of values on the right hand side. An array of value is a comma-separated list of values delimited by square brackets.

The two filters below are equivalent:
```
attribute IN[value1, value2, value3,]

attribute = value1 OR attribute = value2 OR attribute = value3
```
In short, `IN` selects the documents for which:
1. the filterable attribute exists; and
2. the attribute contains a value that is equal to any of the values in the array

The negated form of `IN` can be written in two different ways:
```
attribute NOT IN [value1, value2, etc.]
NOT attribute IN [value1, value2, etc.]
```
and it is equivalent to:
```
attribute != value1 AND attribute != value2 AND ...
```

###### 3.1.2.1.12 Geo Search

- The `_geoRadius` operator selects the documents whose geographical coordinates fall within a certain range of a given coordinate. See [GeoSearch](0059-geo-search.md) for more information.
- The `_geoBoundingBox` operator selects the documents whose geographical coordinates fall within a square described by the given coordinates. See [GeoSearch](0059-geo-search.md) for more information.

##### 3.1.2.2. Array Syntax

The array syntax is an alternative way to combine different filters with `OR` and `AND` operators.

- Elements in the outer array are connected by `AND` operators
- Elements in the inner arrays are connected by `OR` operators

Example:
```json
{
    "filter": [["genres = Comedy", "genres = Romance"], "director = 'Mati Diop'"]
}
```
is equivalent to:
```json
{
    "filter": "(genres = Comedy OR genres = Romance) AND (director = 'Mati Diop')"
}
```

#### 3.1.3. `sort`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

`sort` contains a sort expression written as a string or an array of strings. It sorts the search results at query time according to the specified attributes and indicated order.

Attributes used as sort criteria must be added to the `sortableAttributes list of an index settings. See [Sortable Attributes Setting API](0123-sortable-attributes-setting-api.md).

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `sort` returns an [invalid_search_sort](0061-error-format-and-definitions.md#invalid_search_sort) error.
- 🔴 Sending an invalid syntax for `sort` returns an [invalid_search_sort](0061-error-format-and-definitions.md#invalid_search_sort) error.
- 🔴 Sending a field not defined as a `sortableAttributes` for `sort` returns an [invalid_search_sort](0061-error-format-and-definitions.md#invalid_search_sort) error.

> See [Sort](0055-sort.md)

#### 3.1.4. `facets`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `[]|null`

`facets` permits to specify facets to be computed for the current search query.

It returns the number of documents matching the current search query for each specified facet.

This parameter can take two values:

- An array of attributes: `facets=["attributeA", "attributeB", …]`
- A wildcard `"*"` — this returns a count for all facets present in `filterableAttributes`

Attributes used in `facets` must be added to the `filterableAttributes` list of an index settings. See [Filterable Attributes Setting API](0123-filterable-attributes-setting-api.md).

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `facets` returns an [invalid_search_facets](0061-error-format-and-definitions.md#invalid_search_facets) error.
- 🔴 Sending a field not defined as a `filterableAttributes` for `facets` returns an [invalid_search_facets](0061-error-format-and-definitions.md#invalid_search_facets) error.

The distribution of the different facets is returned in the `facetDistribution` response parameter.

Statistics are computed and returned within the `facetStats` object for distributed facets. See [`facetStats`](#3210-facetstats) section.

#### 3.1.5. `limit`

- Type: Integer
- Required: False
- Default: `20`

Sets the maximum number of documents to be returned for the search query.

If in addition to either `page` and/or `hitsPerPage`, `limit` and/or `offset` are provided as well, `limit` and `offset` are ignored. See [explaination](#3181-navigating-search-results-by-page-selection).

- 🔴 Sending a value with a different type than `Integer` for `limit` returns an [invalid_search_limit](0061-error-format-and-definitions.md#invalid_search_limit) error.

#### 3.1.6. `offset`

- Type: Integer
- Required: False
- Default: `0`

Sets the starting point in the search results, effectively skipping over a given number of documents.

If in addition to either `page` and/or `hitsPerPage`, `limit` and/or `offset` are provided as well, `limit` and `offset` are ignored. See [details](#3181-navigating-search-results-by-page-selection).

- 🔴 Sending a value with a different type than `Integer` for `offset` returns an [invalid_search_offset](0061-error-format-and-definitions.md#invalid_search_offset) error.

#### 3.1.7. `page`

- Type: Integer
- Required: False
- Default: `null`

Sets the specific results page to fetch.

By default, page is `null`, or `1` if `hitsPerPage` is provided.
The first page has a value of `1`, the second `2`, etc... When `0` is provided as a value, no hits are returned.

When providing `page` or `hitsPerPage` in the query parameters, the `page selection` system is enabled, which makes it possible to navigate through the search results pages. See explanation on the [`page selection`](#3181-navigating-search-results-by-page-selection).

If in addition to either `page` and/or `hitsPerPage`, `limit` and/or `offset` are provided as well, `limit` and `offset` are ignored. See [explaination](#3181-navigating-search-results-by-page-selection).

- 🔴 Sending a value with a different type than `Integer` for `page` returns an [invalid_search_page](0061-error-format-and-definitions.md#invalid_search_page) error.

#### 3.1.8. `hitsPerPage`

- Type: Integer
- Required: False
- Default: `null`

Sets the number of results returned for a query.

By default, `hitsPerPage` is `null`, or `20` if `page` is provided. When `0` is provided as a value, no hits are returned.

When providing `page` or `hitsPerPage` in the query parameters, the `page selection` system is enabled, which makes it possible to navigate through the search results pages. See explanation on the [`page selection`](#3181-navigating-search-results-by-page-selection).

If in addition to either `page` and/or `hitsPerPage`, `limit` and/or `offset` are provided as well, `limit` and `offset` are ignored. See [explaination](#3181-navigating-search-results-by-page-selection).

- 🔴 Sending a value with a different type than `Integer` for `hitsPerPage` returns an [invalid_search_hits_per_page](0061-error-format-and-definitions.md#invalid_search_hits_per_page) error.

#### 3.1.8.1. Navigating search results by page selection

By default, `limit` and `offset` are used to navigate search results. While being performant, it lacks exhaustiveness to create a seamless page selection navigation. Upon using `limit`/`offset`, `estimatedTotalHits` is returned, which provides a rough estimation of how many hits may be candidates for a given request. See [`limit`/`offset` usage](#31811-limitoffset-usage) for further explanation.

The `page selection` system provides an alternative that tackles the above-mentioned issue when reliable information is needed to navigate results with a page selector. e.g. A page selector component `<< < 1, 2, 3, ...14 > >>`. Nonetheless, it's considered less performant on a larger number of results as the engine needs to compute the `totalHits` exhaustively.
With this page selection system, it is possible to jump from one page to another using the `page` parameter and decide how many results should be returned per page with the `hitsPerPage` parameter. See [`page`/`hitsPerPage` usage](#31812-pagehitsperpage-usage) for further explanation.

##### 3.1.8.1.1. Limit/offset usage

When either `limit` or `offset` is specified or when neither `limit`, `offset`, `page` and `hitsPerPage` are specified, the response object contains the related fields:
- [`estimatedTotalHits`](#324-estimatedtotalhits)
- [`limit`](#322-limit)
- [`offset`](#323-offset)

If in addition to `limit` and/or `offset`, either `page` or `hitsPerPage` is also provided, `limit` and `offset` parameters are ignored.

For example, given the following query parameters:

- limit: 10
- offset: 1

The response objects contain these specific fields:
```json
{
    "hits": [
        /// ... 10 hits
    ],
    /// ... other fields
    "limit": 10,
    "offset": 1,
    "estimatedTotalHits": 1345
}
```

For example, on a query with no query parameters:

The response objects contain these specific fields:
```json
{
    "hits": [
        /// ... 10 hits
    ],
    /// ... other fields
    "limit": 20,
    "offset": 0,
    "estimatedTotalHits": 1345
}
```

##### 3.1.8.1.2 page/hitsPerPage usage

As soon as either `page` or `hitsPerPage` is used as a query parameter, in the response object, `limit`, `offset`, and `estimatedTotalHits` are removed and page selection related fields are returned:

- [`hitsPerPage`](#326-hitsperpage): number of results per page.
- [`page`](#325-page): current search results page. The counting starts at `1`.
- [`totalPages`](#327-totalpages): total number of results pages. Calculated using `hitsPerPage` value.
- [`totalHits`](#328-totalhits): total number of search results.

Both `totalPages` and `totalHits` are computed until they reach the [`pagination.maxTotalHits`](#https://github.com/meilisearch/specifications/blob/main/text/157-pagination-setting-api.md#311-maxtotalhits) number from the settings.

If in addition to either `page` and/or `hitsPerPage`, `limit` and/or `offset` are provided as well, `limit` and `offset` are ignored.

For example, given the following query parameters:

- page: 2
- hitsPerPage: 10

The response objects contain these specific fields:
```json
{
    "hits": [
        /// ... 10 hits
    ],
    /// ... other fields
    "page": 2,
    "hitsPerPage": 10,
    "totalHits": 2100,
    "totalPages": 210
}
```

For example, given the following query parameters:

- page: 2
- hitsPerPage: 10
- limit: 1

The response objects contain these specific fields:
```json
{
    "hits": [
        /// ... 10 hits
    ],
    /// ... other fields
    "page": 2,
    "hitsPerPage": 10,
    "totalHits": 2100,
    "totalPages": 210
}
```

For example, given the following query parameters:

- page: 0
- hitsPerPage: 10

The response objects contain these specific fields:
```json
{
    "hits": [],
    /// ... other fields
    "page": 0,
    "hitsPerPage": 10,
    "totalHits": 2100,
    "totalPages": 210
}
```


#### 3.1.9. `attributesToRetrieve`

- Type: Array of String (POST) | String (GET)
- Required: False
- Default: `["*"]`, meaning all the attributes

Configures which attributes will be retrieved in the returned documents.

If no value is specified, the default value of `attributesToRetrieve` is used (`["*"]`). This corresponds to the `displayedAttributes` index setting, which by default contains all attributes found in the documents.

> If an attribute is missing from `displayedAttributes` index setting, `attributesToRetrieve` silently ignore it, and the field doesn't appear in the returned search results.

- 🔴 Sending a value with a different type than `Array of String`(POST), `String`(GET) or `null` for `attributesToRetrieve` returns an [invalid_search_attributes_to_retrieve](0061-error-format-and-definitions.md#invalid_search_attributes_to_retrieve) error.

#### 3.1.10. `attributesToHighlight`

- Type: Array of String (POST) | String(GET)
- Required: False
- Default: `[]|null`

Configures which fields may have highlighted parts, given that they match the requested query terms (i.e. the terms in the [`q`](#311-q) search parameter). Pre/post highlighting tags are applied around each word corresponding to a query term.

If `attributesToHighlight` is present in the search query, the search results will include a `_formatted` object containing the attributes and their highlighted parts. For more detailed regarding the `_formatted` behavior, see the [3.2.1.1.2. `_formatted`](#32112-formatted) section.

If `"*"` is provided as a value (`attributesToHighlight=["*"]`), all the attributes present in `displayedAttributes` setting will be highlighted.

Highlighted parts are surrounded by the [`highlightPreTag`](3111-highlightpretag) and [`highlightPostTag`](#3112-highlightposttag) parameters.

`attributesToHighlight` only works on values of the following types: `string`, `number`, `array`, `object`. When highlighted, number attributes are transformed to string.

- 🔴 Sending a value with a different type than `Array[String]`(POST), `String`(GET) or `null` for `attributesToHighlight` returns an [invalid_search_attributes_to_highlight](0061-error-format-and-definitions.md#invalid_search_attributes_to_highlight) error.

##### 3.1.10.1. searchableAttributes

Attributes not defined in the `searchableAttributes` index setting are also highlighted if assigned to `attributesToHighlight`.

##### 3.1.10.2. stopWords

Attributes defined in the `stopWords` index setting are also highlighted if matched.

##### 3.1.10.3. Tokenizer Separators

Tokenizer separators are not highlighted.

##### 3.1.10.4. synonyms

Synonyms are also highlighted.

#### 3.1.11. `highlightPreTag`

- Type: String
- Required: False
- Default: `"<em>"`

Specifies the string to put **before** every highlighted query terms.

This parameter is applied to the fields configured in `attributesToHighlight`. If there are none, this parameter has no effect. See [3.1.10. `attributesToHighlight`](3110-attributestohighlight) section.

- 🔴 Sending a value with a different type than `String` for `highlightPreTag` returns an [invalid_search_highlight_pre_tag](0061-error-format-and-definitions.md#invalid_search_highlight_pre_tag) error.

If `attributesToHighlight` is omitted while `highlightPreTag` is specified, there is no error.

#### 3.1.12. `highlightPostTag`

- Type: String
- Required: False
- Default: `"</em>"`

Specifies the string to put **after** the highlighted query terms.

This parameter is applied to the fields from `attributesToHighlight`. If there are none, this parameter has no effect. See [3.1.10. `attributesToHighlight`](3110-attributestohighlight) section.

- 🔴 Sending a value with a different type than `String` for `highlightPostTag` returns an [invalid_search_highlight_post_tag](0061-error-format-and-definitions.md#invalid_search_highlight_post_tag) error.

If `attributesToHighlight` is omitted while `highlightPostTag` is specified, there is no error.

#### 3.1.13. `attributesToCrop`

- Type: Array[String]|String
- Required: False
- Default: `[]|null`

Defines document attributes to be cropped. Cropped attributes have their values shortened around query terms.

If `attributesToCrop` is present in the search query, the search results will include a `_formatted` object containing the attributes and their cropped parts. For more detailed regarding the `_formatted` behavior, see the [3.2.1.1.2. `_formatted`](#32112-formatted) section.

If `"*"` is provided as a value (`attributesToCrop=["*"]`), all the attributes present in `displayedAttributes` setting will be cropped.

The number of words contained in the cropped value is defined by the `cropLength` parameter. See [3.1.1.14. `cropLength`](#3114-croplength) section.

The value of `cropLength` can be customized per attribute. See [3.1.12.1. Custom `cropLength` Defined Per Cropped Attribute](#31121-custom-croplength-defined-per-attribute) section.

The engine adds a marker by default in front of and/or behind the part selected by the cropper. This marker is customizable. See [3.1.13. `cropMarker`](#3115-cropmarker) section.

- 🔴 Sending a value with a different type than `Array[String]`(POST), `String`(GET) or `null` for `attributesToCrop` returns an [invalid_search_attributes_to_crop](0061-error-format-and-definitions.md#invalid_search_attributes_to_crop) error.

##### 3.1.13.2. searchableAttributes

Attributes configured in `attributesToCrop` are cropped even if not present in the `searchableAttributes` index setting.

##### 3.1.13.3. stopWords

Terms defined in the `stopWords` index setting are counted as words regarding `cropLength`.

##### 3.1.13.3. Tokenizer Separators

Tokenizer separators aren't counted as words regarding `cropLength`.

#### 3.1.14. `cropLength`

- Type: Integer
- Required: False
- Default: `10`

Sets the total number of **words** to keep for the cropped part of an attribute specified in the `attributesToCrop` parameter. It means that if `10` is set for `cropLength`, the cropped part returned in `_formatted` will only be 10 words long.

This parameter is applied to the fields from `attributesToCrop`. If there are none, this parameter has no effect. See [3.1.13. `attributesToCrop`](#3113-attributestocrop) section.

Sending a `0` value deactivates the cropping unless a custom crop length is defined for an attribute inside `attributesToCrop`.

- 🔴 Sending a value with a different type than `Integer` for `cropLength` returns an [invalid_search_crop_length](0061-error-format-and-definitions.md#invalid_search_crop_length) error.

##### 3.1.14.1. Custom `cropLength` Defined Per Attribute.

Optionally, indicating a custom crop length for any of the listed attributes is possible:

`"attributesToCrop":["attributeNameA:15", "attributeNameB:30"]`

A custom crop length set in this way has priority over the `cropLength` parameter.

##### 3.1.14.2 Examples

###### 3.1.14.1.1. Extending around

Given an attribute defined in `attributesToCrop` containing:

`"In his ravenous hatred he found no peace, and with boiling blood he scoured the umbral plains, seeking vengence afgainst the dark lords who had robbed him."`

With `croplength` defined as `5` and `q` defined as `boiling blood`, the cropped value will be:

`"…and with boiling blood he…"`

Cropped query terms are counted as a word regarding `cropLength`.

Sending more query terms than the `cropLength` value has no impact. The cropped part will contain the `cropLength` number.

###### 3.1.14.1.2. Keeping a phrase context

After Meilisearch has chosen the best possible match window (some number of words < `cropLength`), it will add words from before or after the match window until the total number is equal to `cropLength`. In doing so, it will attempt to add context to the match window by choosing words from the same sentence(s) where the match window occurs.

For instance, for the matching word `Split` the text:

`"Natalie risk her future. Split The World is a book written by Emily Henry. I never read it."`

will be cropped like:

`…Split The World is a book written by Emily Henry…`

and not like:

`Natalie risk her future. Split The World is a book…`

#### 3.1.15. `cropMarker`

- Type: String
- Required: False
- Default: `"…"` (U+2026)

Sets which string to add before and/or after the cropped text. See [3.1.13. `attributesToCrop`](#3113-attributestocrop) section.

The specified crop marker is applied by following rules outline in section [3.1.15.1. Applying `cropMarker`](#3115-cropmarker).

Specifying `cropMarker` to `""` or `null` implies that no marker will be applied to the cropped part.

This parameter is applied to the fields configured in `attributesToCrop`. If there are none, this parameter has no effect. See [3.1.13. `attributesToCrop`](#3113-attributestocrop) section.

- 🔴 Sending a value with a different type than `String` or `null` for `cropMarker` returns an [invalid_search_crop_marker](0061-error-format-and-definitions.md#invalid_search_crop_marker) error.

##### 3.1.15.1. Applying `cropMarker`

###### 3.1.15.1.1. Matched Part To Be Cropped

The cropping algorithm tries to match the window with the highest density of query terms within the `cropLength` limit.

The cropping algorithm tries to find the crop window that contains the most relevant matches.

1. That has the highest count of unique matches

For example, for the query terms `split the world`, then the interval `the split the split the` has `5` matches but only `2` unique matches (`1` for `split` and `1` for `the`) where the interval `split of the world` has `3` matches and `3` unique matches. So the interval `split of the world` is considered better.

2. That have the minimum distance between matches

For example, for the query terms `split the world`, then the interval `split of the world` has a distance of `3` (`2` between `split` and `the`, and `1` between `the` and `world`) where the interval `split the world` has a distance of `2`. So the interval `split the world` is considered better.

3. That have the highest count of ordered matches

For example, for the query terms `split the world`, then the interval `the world split` has `2` ordered words where the interval `split the world` has `3`. So the interval `split the world` is considered better.

Only one cropped part from an attribute is returned.

If no part is found when selecting a part to be cropped, the returned value in `_formatted` will start at the beginning of the attribute and include a number of words equal to `cropLength`.

###### 3.1.15.1.2. Positioning Markers

If the cropped part has been matched against query terms and contains the beginning of the attribute to be cropped, the `cropMarker` is not placed to the left of the cropped part.

If the cropped part has been matched against query terms and contains the end of the attribute to be cropped, the `cropMarker` is not placed to the right of the cropped part.

#### 3.1.16. `showMatchesPosition`

- Type: Boolean
- Required: False
- Default: `false`

Adds a `_matchesPosition` object to the search response that contains the location of each occurrence of queried terms across all fields. The given positions are in bytes.

It's useful when more control is needed than offered by the built-in highlighting/cropping features.

- 🔴 Sending a value with a different type than `Boolean` or `null` for `showMatchesPosition` returns an [invalid_search_show_matches_position](0061-error-format-and-definitions.md#invalid_search_show_matches_position) error.

#### 3.1.17. `matchingStrategy`

- Type: String
- Required: False
- Default: `last`

Defines which strategy to use to match the query terms within the documents as search results.

Two different strategies are available, `last` and `all`. By default, the `last` strategy is chosen.

- 🔴 Sending a value with a different type than `String` and other than `last` or `all` as a value for `matchingStrategy` returns an [invalid_search_matching_strategy](0061-error-format-and-definitions.md#invalid_search_matching_strategy) error.

##### 3.1.17.1. `last` strategy

The documents containing ALL the query words (i.e. in the `q` parameter) are returned first by Meilisearch. If Meilisearch doesn't have enough documents to fit the requested `limit`, it iteratively ignores the query words from the last typed word to the first typed word to match more documents.

##### 3.1.17.2. `all` strategy

Only the documents containing ALL the query words (i.e. in the `q` parameter) are returned by Meilisearch. If Meilisearch doesn't have enough documents to fit the requested `limit`, it returns the documents found without trying to match more documents.

### 3.2. Search Response Properties

| Field                                           | Type       | Required |
|-------------------------------------------------|------------|----------|
| [`hits`](#321-hits)                             | Array[Hit] | True     |
| [`limit`](#322-limit)                           | Integer    | False    |
| [`offset`](#323-offset)                         | Integer    | False    |
| [`estimatedTotalHits`](#324-estimatedTotalHits) | Integer    | False    |
| [`page`](#325-page)                             | Integer    | False    |
| [`hitsPerPage`](#326-hitsperpage)               | Integer    | False    |
| [`totalPages`](#327-totalpages)                 | Integer    | False    |
| [`totalHits`](#328-totalhits)                   | Integer    | False    |
| [`facetDistribution`](#329-facetdistribution)   | Object     | False    |
| [`facetStats`](#3210-facetstats)                | Object     | False    |
| [`processingTimeMs`](#3211-processingtimems)    | Integer    | True     |
| [`query`](#3212-query)                          | String     | True     |

#### 3.2.1. `hits`

- Type: Array[Hit]
- Required: True

Results of the search query as an array of documents.

> Hit object represents a matched document as a search result.

> The search parameters `attributesToRetrieve` influence the returned payload for a hit. See [3.1.7. `attributesToRetrieve`](#319-attributestoretrieve) section.

A search result can contain special properties. See [3.2.1.1. `hit` Special Properties](#3211-hits-special-properties) section.

##### 3.2.1.1. `hit` Special Properties

| Field                                        | Type    | Required |
|----------------------------------------------|---------|----------|
| [`_geoDistance`](#32111-geodistance)         | Integer | False    |
| [`_formatted`](#32112-formatted)             | Object  | False    |
| [`_matchesPosition`](#32113-matchesposition) | Object  | False    |

###### 3.2.1.1.1. `_geoDistance`

- Type: Integer
- Required: False

Search queries using `_geoPoint` returns a `_geoDistance` field containing the distance in meters between the document `_geo` coordinates and the specified `_geoPoint`.

> See [GeoSearch](0059-geo-search.md)

###### 3.2.1.1.2. `_formatted`

- Type: Object
- Required: False

`_formatted` is an object returned in the search response, only if at least one of the following paramaters has been set in the search query:
- `attributesToHighlight`
- `attributesToCrop`

If `attributesToHighlight` and `attributesToCrop` are not set, `_formatted` is not returned.

This `_formatted` object will be present in each returned document in the `hits` field.

Example:

```json
{
    "attributesToCrop": ["title"]
}
```

```json
{
    "hits": [
        {
            "id": 2,
            "title": "Pride and Prejudice",
            "_formatted": {
                "id": "2",
                "title": "Pride and Prejudice"
            }
        },
        {
            "id": 456,
            "title": "Le Petit Prince",
            "_formatted": {
                "id": "456",
                "title": "Le Petit Prince",
            }
        }
    ],
    ...
}
```

Which attributes are present in `_formatted`?

*Remember the main rule: `_formatted` is only present if `attributesToHighlight` or `attributesToCrop` is set.*

The `_formatted` object contains attributes coming from the original document, depending on the parameters the users set during the search query. Indeed, **`_formatted` contains all the attributes present in `attributesToRetrieve`, `attributesToHighlight`, and `attributesToCrop` combined**.

Knowing the default value of `attributesToRetrieve` is `["*"]` (so all the attributes present in `displayedAttributes`), if no `attributesToRetrieve` are set in the search query, `_formatted` will return all the `displayedAttributes`.

Returning attributes in the `_formatted` object does not mean these attributes will be necessarily highlighted or cropped, see the next point.

Which attributes are highlighted or cropped in `_formatted`?

No matter which attributes are retrieved in `_formatted` (according to the previous section "Which attributes are present in `_formatted`?"):
- Only the attributes present in `attributesToHighlight` are highlighted.
- Only the attributes present in `attributesToCrop` are cropped.
- Attributes present in both are cropped and highlighted at the same time.

Some edge cases:
- If cumulated fields in `attributesToHighlight` and `attributesToCrop` resolve to only having non-existent fields, `_formatted` is not returned.

Some examples:
*The examples work the same with `attributesToCrop`*

Example 1:

```json
{
    "q": "t",
    "attributesToHighlight": ["title"]
}
```

```json
{
    "hits": [
        {
            "id": 1,
            "title": "The Hobbit",
            "author": "J. R. R. Tolkien",
            "_formatted": {
                "id": "1",
                "title": "<em>T</em>he Hobbit",
                "author": "J. R. R. Tolkien"
            }
        }
    ],
    ...
}
```
-> All the attributes (so `id`, `title` and `author`) are returned in `_formatted` because by default `attributesToRetrieve` is set to `["*"]`.
-> Only `title` is highlighted.

Example 2:

```json
{
    "q": "t",
    "attributesToHighlight": ["*"]
}
```

```json
{
    "hits": [
        {
            "id": 1,
            "title": "The Hobbit",
            "author": "J. R. R. Tolkien",
            "_formatted": {
                "id": "1",
                "title": "<em>T</em>he Hobbit",
                "author": "J. R. R. <em>T</em>olkien"
            }
        }
    ],
    ...
}
```
-> `id`, `title` and `author` are returned in `_formatted` because`attributesToHighlight` is set to `["*"]` (but also `attributesToRetrieve` by default).
-> Both `title` and `author` are highlighted because `attributesToHighlight` is set to `["*"]`.

Example 3:

```json
{
    "q": "t",
    "attributesToRetrieve": ["author"],
    "attributesToHighlight": ["title"]
}
```

```json
{
    "hits": [
        {
            "author": "J. R. R. Tolkien",
            "_formatted": {
                "title": "<em>T</em>he Hobbit",
                "author": "J. R. R. Tolkien"
            }
        }
    ],
    ...
}
```
-> Only `author` is returned at the root of the document because defined in the `attributesToRetrieve`.
-> Only `author` and `title` are returned in `_formatted` because the addition of `attributesToRetrieve` and `attributesToHighlight`.
-> Only `title` is highlighted because the only one defined in `attributesToHighlight`.

Example 4:

```json
{
    "q": "t",
    "attributesToRetrieve": [],
    "attributesToHighlight": ["*"]
}
```

```json
{
    "hits": [
        {
            "_formatted": {
                "id": "1",
                "title": "<em>T</em>he Hobbit",
                "author": "J. R. R. <em>T</em>olkien"
            }
        }
    ],
    ...
}
```
-> No attributes are returned at the root of the document because `attributesToRetrieve` is set to `[]`.
-> All the attributes are returned in `_formatted` because `attributesToHighlight` is set to `["*"]`.
-> All the attributes are highlighted because `attributesToHighlight` is set to `["*"]`.


###### 3.2.1.1.3. `_matchesPosition`

- Type: Object
- Required: False

Contains the location of each occurrence of queried terms across all fields. The `_matchesPosition` object is added to a search result when the `showMatchesPosition` search parameter is specified to true.

The beginning of a matching term within a field is indicated by `start`, and its `length` by length.

`start` and `length` are measured in bytes and not the number of characters. For example, `ü` represents two bytes but one character.

> See [3.1.14. `showMatchesPosition`](#3116-showmatchesposition) section.

#### 3.2.2. `limit`

- Type: Integer
- Required: False

Returns the `limit` search parameter used for the query.
This field is returned only when:
- `limit` and/or `offset` are used as query parameters.
- None of `limit`, `offset`, `page`, `hitsPerPage` are used as a query parameter

See [details](#3181-navigating-search-results-by-page-selection) on the different ways of navigating search results.

> See [3.1.5. `limit`](#315-limit) section.

#### 3.2.3. `offset`

- Type: Integer
- Required: False

Returns the `offset` search parameter used for the query.
This field is returned only when none of `page` and `hitsPerPage` are used as a query parameter.

See [explanation](#3181-navigating-search-results-by-page-selection) on the different ways of navigating search results.

> See [3.1.6. `offset` section](#316-offset) section.

#### 3.2.4. `estimatedTotalHits`

- Type: Integer
- Required: False

Returns the estimated number of candidates for the search query. This field is returned only when `limit` or/and `offset` are used as a query parameter.

This field is returned only when:
- `limit` and/or `offset` are used as query parameters.
- None of `limit`, `offset`, `page`, `hitsPerPage` are used as a query parameter

See [details](#3181-navigating-search-results-by-page-selection) on the different ways of navigation search results.

#### 3.2.5. `page`

- Type: Integer
- Required: False

Returns the current search results page. This field is returned only when the `page selection` feature is enabled; see [details](#3181-navigating-search-results-by-page-selection).

> See [3.1.7. `page` section](#317-page) section.

#### 3.2.6. `hitsPerPage`

- Type: Integer
- Required: False

Returns the number of results per page. This field is returned only when the `page selection` feature is enabled; see [details](#3181-navigating-search-results-by-page-selection).

> See [3.1.7. `hitsPerPage` section](#318-hitsperpage) section.

#### 3.2.7. `totalPages`

- Type: Integer
- Required: False

Returns the total number of results pages. Calculated using [`hitsPerPage`]. Both `totalPages` and `totalHits` are computed until they reach the [`pagination.maxTotalHits`](#https://github.com/meilisearch/specifications/blob/main/text/157-pagination-setting-api.md#311-maxtotalhits) number from the settings.

This field is returned only when the `page selection` feature is enabled; see [details](#3181-navigating-search-results-by-page-selection).

#### 3.2.8. `totalHits`

- Type: Integer
- Required: False

Returns the total number of search results. Both `totalPages` and `totalHits` are computed until they reach the [`pagination.maxTotalHits`](#https://github.com/meilisearch/specifications/blob/main/text/157-pagination-setting-api.md#311-maxtotalhits) number from the settings.

This field is returned only when the `page selection` feature is enabled, see [details](#3181-navigating-search-results-by-page-selection).

#### 3.2.9. `facetDistribution`

- Type: Object
- Required: False

Added to the search response when `facets` is set for a search query. It contains the number of remaining candidates for each specified facet in the `facets` search parameter.

If a field distributed as a facet contains no value, it is returned as a `facetDistribution` field with an empty object as value.

> See [3.1.4. `facets`](#314-facets) section.

#### 3.2.10. `facetStats`

- Type: Object
- Required: False

When using the `facets` parameter, the distributed facets that contain some numeric values are displayed in a `facetStats` object that contains, per facet, the numeric `min` and `max` values for that facet of all documents matching the search query.

If none of the hits returned by the search query have a numeric value for a facet, this facet is not part of the `facetStats` object.

It ignores string values even if parseable. e.g `"21"` isn't considered by the engine when computing the `facetStats` `min` and `max`.

> See [3.1.4. `facets`](#314-facets) section.

#### 3.2.11. `processingTimeMs`

- Type: Integer
- Required: True

Processing time of the search query in **milliseconds**.

#### 3.2.12. `query`

- Type: String
- Required: True
- Default: `""`

Query originating the response. Equals to the `q` search parameter.

> See [3.1.1. `q`](#311-q) section.

## 2. Technical Details
n/a

## 3. Future Possibilities

- Add dedicated errors to replace `bad_request` error.

### 3.1. Formatting Search Results

- Replaces `_matchesPosition` with chars position instead of bytes. It could also be a `mode` to choose `byte` or `char`.
- Move `attributesToHighlight`, `highlightPreTag`, `highlightPostTag`, `attributesToCrop`, `cropLength` and `cropMarker` into a `formatter` objet.
- Add an option to only highlight complete query term.
- Expose the `formatter` resource as an index setting.
- Highlight a phrase search as a single highlighted section.
