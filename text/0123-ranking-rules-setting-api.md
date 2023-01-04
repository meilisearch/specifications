# Ranking Rules Setting API

## 1. Summary

This specification describes the `rankingRules` index setting API endpoints.

## 2. Motivation
N/A

## 3. Functional Specification

### 3.1. Explanations

To ensure relevant results at search time, documents are sorted based on consecutive rules called ranking rules. The order in which ranking rules are applied matters.

The first rule in the `rankingRules` list has the most impact, and the last rule has the least.

This setting is fully customizable, meaning existing rules can be removed, new ones added and reordered as needed.

#### 3.1.1. Built-in Ranking Rules

By default, Meilisearch contains six built-in ranking rules in the following order:

1. [Words](#3111-words-ranking-rule)
2. [Typo](#3112-typo-ranking-rule)
3. [Proximity](#3113-proximity-ranking-rule)
4. [Attribute](#3114-attribute-ranking-rule)
5. [Sort](#3115-sort-ranking-rule)
6. [Exactness](#3116-exactness-ranking-rule)

##### 3.1.1.1. Words Ranking Rule

Results are sorted by decreasing the number of matched query terms. `words` ranks documents that contain all query terms first.

The `words` ranking rule works from right to left. Therefore, the order of the query string impacts the order of results.

For example, if someone were to search `batman dark knight`, then the words rule would rank documents containing all three terms first, documents containing only `batman` and `dark` second, and documents containing only `batman` third.

##### 3.1.1.2. Typo Ranking Rule

Results are sorted by increasing the number of typos. `typo` ranks documents that match query terms with fewer typos first.

Meiliseach tolerates a maximum of `2` typos for a query term.

##### 3.1.1.3. Proximity Ranking Rule

Results are sorted by increasing distance between matched query terms. `proximity` ranks documents where query terms occur close together and in the same order as the query terms first.

##### 3.1.1.4. Attribute Ranking Rule

Results are sorted according to the attribute ranking order. `attribute` ranks documents that contain query terms in more important attributes first. See [searchableAttributes setting API](0123-searchable-attributes-setting-API).

Also, note that the documents with attributes containing the query terms at the beginning of an attribute will be considered more relevant than documents containing the query terms at the end of an attribute.

##### 3.1.1.5. Sort Ranking Rule

Results are sorted according to parameters decided at query time.

When the `sort` ranking rule is in a **higher** position, sorting is exhaustive: results will be less relevant, but follow the user-defined sorting order more closely.

When the `sort` ranking rule is in a **lower** position, sorting is relevant: results will be very relevant, but might not always follow the order defined by the user.

Unlike other ranking rules, `sort` is only active for search queries containing the `sort` search parameter ([See Sort Search Parameter](0118-search-api.md#1213-sort)). If a search request does not contain `sort`, this ranking rule will be ignored.

If a field has values of different types across documents, Meilisearch will give precedence to numbers over strings. It means documents with numeric field values will be ranked higher than those with string values.

##### 3.1.1.6. Exactness Ranking Rule

Results are sorted by the similarity of the matched words with the query words. `exactness` ranks documents that contain exactly the same terms as the ones queried first.

### 3.1.2. Custom Ranking Rules

Meilisearch supports two custom rules expression that can be added to the ranking rules setting: one for ascending sort and one for descending sort.

To add a custom ranking rule, the attribute name must be specified and followed by a colon (`:`) and either `asc` for ascending order or `desc` for descending order.

To apply an ascending sort (results sorted by increasing value of the attribute): `"attribute_name:asc"`

To apply a descending sort (results sorted by decreasing value of the attribute): `"attribute_name:desc"`

The attribute must have either a numeric or a string value.

Contrary to the `sort` ranking rule, custom ranking rules are always active after configured and can be helpful to promote certain types of results. The `sort` ranking rule is most useful when end-users define what type of results they want to see first.

#### 3.1.2.1. Example

Suppose a movie dataset. The documents contain a field `release_date` with a timestamp as a value.

The following example creates a custom ranking rule that makes recent movies more relevant than older ones. A movie released in 2020 will appear before a movie released in 1999.

***Request payload `PUT`- `/indexes/products/settings/ranking-rules`***
```json
[
    "words",
    "typo",
    "proximity",
    "sort",
    "exactness",
    "release_date:desc"
]
```

### 3.2. Global Settings API Endpoints Definition

`rankingRules` is a sub-resource of `/indexes/:index_uid/settings`.

See [Settings API](0000-settings-api.md).

### 3.3. API Endpoints Definition

Manipulate the `rankingRules` setting of a Meilisearch index.

#### 3.3.1. `GET` - `indexes/:index_uid/settings/ranking-rules`

Fetch the `rankingRules` setting of a Meilisearch index.

##### 3.3.1.1. Response Definition

- Type: Array of String
- Default: `["words", "typo", "proximity", "attribute", "sort", "exactness"]`

##### 3.3.1.2. Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.3.2. `PUT` - `indexes/:index_uid/settings/ranking-rules`

Modify the `rankingRules` setting of a Meilisearch index.

##### 3.3.2.1. Request Payload Definition

- Type: Array of String / `null`

Setting `null` is equivalent to using the [3.3.3. `DELETE` - `indexes/:index_uid/settings/ranking-rules`](#333-delete---indexesindexuidsettingsranking-rules) API endpoint.

Specifying a document attribute that does not exist as a `rankingRules` index setting returns no error.

Specifying `[]` for the `rankingRules` index setting allows specifying that no ranking rules are used to rank results. Search results are sorted by their **internal id** which can be considered as **undefined order**.

##### 3.3.2.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.2.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending a request payload value type different of `Array of String`, `[]`,  or `null` returns an [invalid_settings_ranking_rules](0061-error-format-and-definitions.md#invalid_settings_ranking_rules) error.

###### 3.3.2.3.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.3.2.2. Response Definition](#3222-response-definition).

> Otherwise, Meilisearch will create the index in a lazy way. See [3.2.2.4. Lazy Index Creation](#3224-lazy-index-creation).

- 🔴 Sending an invalid ranking rule returns an [invalid_settings_ranking_rules](0061-error-format-and-definitions.md#invalid_settings_ranking_rules) error in the related asynchronous `task` resource. See [3.3.2.2. Response Definition](#3222-response-definition).

##### 3.3.2.4. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.3.2.3.1. Async Errors](#33231-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.3.2.2. Response Definition](#3322-response-definition).

#### 3.3.3. `DELETE` - `indexes/:index_uid/settings/ranking-rules`

Reset the `rankingRules` setting of a Meilisearch index to the default value `["*"]`.

##### 3.3.3.1. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.3.3. Errors

###### 3.3.3.3.1. Asynchronous Index Not Found Error

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related async `task` resource. See [3.3.3.1. Response Definition](#3331-response-definition).

#### 3.3.4. General Errors

These errors apply to all endpoints described here.

##### 3.3.4.1 Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 4. Technical Details

### 4.1. Bucket Sort

Whenever a search query is made, Meilisearch uses a [bucket sort](https://en.wikipedia.org/wiki/Bucket_sort) algorithm to rank documents.

The first ranking rule is applied to all documents, while each subsequent rule is only applied to documents that are considered equal under the previous rule (i.e. as a tiebreaker).

## 5. Future Possibilities

- Return an error when `rankingRules` is defined as an empty array