# Searchable Attributes Setting API

## 1. Summary

This specification describes the `searchableAttributes` index setting API endpoints.

## 2. Motivation
N/A

## 3. Functional Specification

### 3.1. Explanations

Documents in Meilisearch are composed of multiple fields. A field can either be searchable or non-searchable.

When a search query is performed, all searchable fields are checked for matching query words and used to assess document relevancy, while non-searchable fields are ignored entirely.

By default, Meilisearch looks for matches in every field, but the `searchableAttributes` setting object allow to customize that behavior.

#### 3.1.1. Usage Example

Suppose a database of movies with the following fields: `id`, `overview`, `genres`, `title`, `release_date`. These fields all contain useful information; however, some are more useful to search than others. To make the `id` and `release_date` fields non-searchable and re-order the remaining fields by importance, it can be specified in the following way.

***Request payload `PUT`- `/indexes/movies/settings/searchable-attributes`***
```json
["overview", "genres", "title"]
```

#### 3.1.2. Field Ordering

Meilisearch uses an ordered list to determine which attributes are searchable. The order in which attributes appear in this list also determines their impact on relevancy, from most impactful to least.

In other words, the `searchableAttributes` setting serves two purposes:

- It designates the fields that are searchable.
- It dictates the attribute ranking order.

There are two possible modes for the `searchableAttributes` setting.

##### 3.1.1.1. Default case: Automatic

By default, all attributes are automatically added to the `searchableAttributes` list in their order of appearance.

This means that the initial order will be based on the order of attributes in the first document indexed, with each new attribute found in subsequent documents added at the end of this list.

This default behavior is indicated by a `searchableAttributes` value of `["*"]`.

##### 3.1.1.2. Manual

To make some attributes non-searchable, or change the attribute ranking order. Attributes must be described from most important to least important.

***Request payload `PUT`- `/indexes/movies/settings/searchable-attributes`***
```json
["title", "overview", "genres"]
```

`title` is considered more important than `overview` while `overview` is considered more important than `genres`.

The [Attribute Ranking Rule]() ranks search results by the order defined in the `searchableAttributes` setting. Documents that contain query terms in the more important searchable attribute will be returned first.

Manually updating `searchableAttributes` will change the displayed order of document fields in the JSON response.

#### 3.1.3. Relationship With Ranking Rules

A document field that is not defined in the list of `searchableAttributes` will not be considered by the following ranking rules to match and rank search results.

1. [Words](0123-settings-api.md#3111-words-ranking-rule)
2. [Typo](0123-settings-api.md#3112-typo-ranking-rule)
3. [Proximity](0123-settings-api.md#3113-proximity-ranking-rule)
4. [Attribute](0123-settings-api.md#3114-attribute-ranking-rule)
6. [Exactness](0123-settings-api.md#3116-exactness-ranking-rule)

### 3.2. Global Settings API Endpoints Definition

`searchableAttributes` is a sub-resource of `/indexes/:index_uid/settings`.

See [Settings API](0123-settings-api.md).

### 3.3. API Endpoints Definition

Manipulate the `searchableAttributes` setting of a Meilisearch index.

#### 3.3.1. `GET` - `indexes/:index_uid/settings/searchable-attributes`

Fetch the `searchableAttributes` setting of a Meilisearch index.

##### 3.3.1.1. Response Definition

- Type: Array of String
- Default: `["*"]`

##### 3.3.1.2. Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.3.2. `PUT` - `indexes/:index_uid/settings/searchable-attributes`

Modify the `searchableAttributes` setting of a Meilisearch index.

##### 3.3.2.1. Request Payload Definition

- Type: Array of String / `null`

Setting `null` is equivalent to using the [3.3.3. `DELETE` - `indexes/:index_uid/settings/searchable-attributes`](#333-delete---indexesindexuidsettingssearchable-attributes) API endpoint.

Specifying a document attribute that does not exist as a `searchableAttributes` index setting returns no error.

Specifying `[]` for the `searchableAttributes` index setting allows to specify that all fields are non-searchable.

##### 3.3.2.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.2.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending a request payload value type different of `Array of String`, `[]` or `null` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

###### 3.3.2.3.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.3.2.2. Response Definition](#3222-response-definition).

> Otherwise, Meilisearch will create the index in a lazy way. See [3.2.2.4. Lazy Index Creation](#3224-lazy-index-creation).

##### 3.3.2.4. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.3.2.3.1. Async Errors](#33231-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.3.2.2. Response Definition](#3322-response-definition).

#### 3.3.3. `DELETE` - `indexes/:index_uid/settings/searchable-attributes`

Reset the `searchableAttributes` setting of a Meilisearch index to the default value `["*"]`.

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

### 4.1. Triggering Documents Re-Indexing

Meilisearch favors search speed and makes a trade-off on indexing speed by computing internal data structures to get search results as fast as possible.

Modifying this index setting cause documents to be re-indexed.

## 5. Future Possibilities

- Add dedicated error to avoid using generic `bad_request` error code
- Return an error when `searchableAttributes` is defined as an empty array
- Fix the reordering issue of document representation when `searchableAttributes` is specified.