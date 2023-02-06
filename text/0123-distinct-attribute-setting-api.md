# Distinct Attribute Setting API

## 1. Summary

This specification describes the `distinctAttribute` index setting API endpoints.

## 2. Motivation
N/A

## 3. Functional Specification

### 3.1. Explanations

The value of a field whose attribute is set as a distinct attribute will always be unique in the returned documents.

#### 3.1.1. Usage Example

Suppose an e-commerce dataset. For an index that contains information about jackets, a products index may have several identical items in different variations (color or size).

As shown below, 2 documents containing the same jacket are defined. One of the jackets is brown and the other one is black.

```json
[
  {
    "id": 1,
    "description": "Leather jacket",
    "brand": "Lee jeans",
    "color": "brown",
    "product_id": "123456"
  },
  {
    "id": 2,
    "description": "Leather jacket",
    "brand": "Lee jeans",
    "color": "black",
    "product_id": "123456"
  }
]
```

By default, a search for `Leather jacket` would return all documents. This might not be desired, since displaying nearly identical variations of the same item can make results appear cluttered.

By setting `product_id` as the `distinctAttribute` setting, the different variations of an item will be ignored.

***Request payload `PUT`- `/indexes/products/settings/distinct-attribute`***

```json
"product_id"
```

After setting the distinct attribute as shown above, querying for `Leather jacket` would only return the **first document** found. The search response would look like this:

```json
{
  "hits": [
    {
      "id": 1,
      "description": "Leather jacket",
      "brand": "Lee jeans",
      "color": "brown",
      "product_id": "123456"
    }
  ],
  "offset": 0,
  "limit": 20,
  "nbHits": 1,
  "exhaustiveNbHits": false,
  "processingTimeMs": 0,
  "query": "Leather jacket"
}
```

Search requests will never return more than one document with the same `product_id`.

### 3.2. Global Settings API Endpoints Definition

`distinctAttribute` is a sub-resource of `/indexes/:index_uid/settings`.

See [Settings API](0123-settings-api.md).

### 3.3. API Endpoints Definition

Manipulate the `distinctAttribute` setting of a Meilisearch index.

#### 3.3.1. `GET` - `indexes/:index_uid/settings/distinct-attribute`

Fetch the `distinctAttribute` setting of a Meilisearch index.

##### 3.3.1.1. Response Definition

- Type: String / `null`
- Default: `null`

##### 3.3.1.2. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.3.2. `PUT` - `indexes/:index_uid/settings/distinct-attribute`

Modify the `distinctAttribute` setting of a Meilisearch index.

##### 3.3.2.1. Request Payload Definition

- Type: String / `null`

Setting `null` is equivalent to using the [3.3.3. `DELETE` - `indexes/:index_uid/settings/distinct-attribute`](#333-delete---indexesindexuidsettingsdistinct-attribute) API endpoint.

Specifying a document attribute that does not exist as a `distinctAttribute` index setting returns no error.

##### 3.3.2.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.2.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a request payload value type different of `String` or `null` returns an [invalid_settings_distinct_attribute](0061-error-format-and-definitions.md#invalid_settings_distinct_attribute) error.

###### 3.3.2.3.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.3.2.2. Response Definition](#3222-response-definition).

> Otherwise, Meilisearch will create the index in a lazy way. See [3.2.2.4. Lazy Index Creation](#3224-lazy-index-creation).

##### 3.3.2.4. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.3.2.3.1. Async Errors](#33231-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.3.2.2. Response Definition](#3322-response-definition).

#### 3.3.3. `DELETE` - `indexes/:index_uid/settings/distinct-attribute`

Reset the `distinctAttribute` setting of a Meilisearch index to the default value `null`.

##### 3.3.3.1. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.3.3. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.

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

- Return an error when `distinctAttribute` is specified as an empty string