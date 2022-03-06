- Title: Filterable Attributes Setting API

# Filterable Attributes Setting API

## 1. Summary

This specification describes the `filterableAttributes` setting API endpoints.

## 2. Motivation
N/A

## 3. Functional Specification

### 3.1. Explanations

`filterableAttributes` setting allows to configure the document fields usable as filter criteria and as facets.

Filters have several use-cases, such as restricting the results a specific user has access to or creating faceted search interfaces. Faceted search interfaces are particularly efficient in helping users navigate a great number of results across many broad categories.

`filterableAttributes` need to be properly processed and prepared by Meilisearch before they can be used. Fields defined as `filterableAttributes` are usable in the [`filter`](0118-search-api.md#1212-filter) and [facetsDistribution](0118-search-api.md#1214-facetsdistribution) search API parameters.

By default, Meilisearch has no filterable attributes defined.

### 3.2. Global Settings API Endpoints Definition

`filterableAttributes` is a sub-resource of `/indexes/:index_uid/settings`.

See [Settings API](0000-settings-api.md).

### 3.3. API Endpoints Definition

Manipulate the `filterableAttributes` setting of a Meilisearch index.

#### 3.3.1. `GET` - `indexes/:index_uid/settings/filterable-attributes`

Fetch the `filterableAttributes` setting of a Meilisearch index.

##### 3.3.1.1. Response Definition

- Type: Array of String
- Default: `[]`

##### 3.3.1.2. Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.3.2. `POST` - `indexes/:index_uid/settings/filterable-attributes`

Modify the `filterableAttributes` setting of a Meilisearch index.

##### 3.3.2.1. Request Payload Definition

- Type: Array of String / `null`

Setting `null` is equivalent to using the [3.3.3. `DELETE` - `indexes/:index_uid/settings/filterable-attributes`](#333-delete---indexesindexuidsettingsfilterable-attributes) API endpoint.

Specifying a document attribute that does not exist as a `filterableAttributes` index setting returns no error.

##### 3.3.2.2. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.3.2.3. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending a request payload value type different of `Array of String`, `[]`,  or `null` returns a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

###### 3.3.2.3.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.3.2.2. Response Definition](#3222-response-definition).

> Otherwise, Meilisearch will create the index in a lazy way. See [3.2.2.4. Lazy Index Creation](#3224-lazy-index-creation).

##### 3.3.2.4. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.3.2.3.1. Async Errors](#33231-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.3.2.2. Response Definition](#3322-response-definition).

#### 3.3.3. `DELETE` - `indexes/:index_uid/settings/filterable-attributes`

Reset the `filterableAttributes` setting of a Meilisearch index to the default value `[]`.

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

### 4.1. Indexing `filterableAttributes`

Changing the `filterableAttributes` setting of an index causes documents to be re-indexed.

## 5. Future Possibilities
- Replace `POST` HTTP verb with `PATCH`
- Add dedicated error to avoid using generic `bad_request` error code