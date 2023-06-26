# Faceting Settings API

## 1. Summary

This specification describes the customizable options for the faceting settings.

## 2. Motivation

Despite the default values that work out-of-the-box for most users, some need to go further in customization.

This settings will host the parameters to configure the faceting behavior for an index.

## 3. Functional Specification

### 3.1. `faceting` API resource definition

| Field                                            | Type            | Required |
|--------------------------------------------------|-----------------|----------|
| [maxValuesPerFacet](#311-maxValuesPerFacet)      | Integer         | False    |
| [sortFacetValuesBy](#312-sortFacetValuesBy)      | Object          | False    |

#### 3.1.1. `maxValuesPerFacet`

- Type: Integer
- Required: False
- Default: `100`

Define maximum number of value returned for all distributed facets.

e.g. It means that with the default value of `100`, it is not possible to have `101` different colors if the `color` field is defined as a facet at search time.

The value of 100 ensures good performance and prevents malicious users from scraping data from a Meilisearch instance.

Increasing this value can degrade performance as well as expose the data of an instance to scrapping.

The facets that are returned are sorted in ascending lexicographical order.

#### 3.1.2 `sortFacetValuesBy`

- Type: Object
- Required: False
- Default:

```json
{
    "*": "alpha"
}
```

Defines how facet values are sorted. By default, all facets (`*`) are sorted by name, alphanumerically in ascending order (`alpha`).

It is possible to sort them by the number of documents containing a facet value in descending order using `count`.

It is possible to specify a particular order for a facet.

```json
{
    "*": "alpha",
    "genre": "count"
}
```

In this example, values from facets other than `genres` will be displayed sorted by their name in ascending alphanumeric order, while values from the `genres` facet will be sorted in descending order by the count of the number of documents containing each value.

## 3.2. API Endpoints Definition

### 3.2.1. Global Settings API Endpoints Definition

`faceting` is a sub-resource of `/indexes/:index_uid/settings`.

See [Settings API](0123-settings-api.md).

### 3.2.2. `indexes/:index_uid/settings/faceting`

Manage the faceting configuration for an index.

#### 3.2.2.1. `GET` - `indexes/:index_uid/settings/faceting`

Allow fetching the current definition of the faceting setting for an index.

`200` - Response body

```json
    {
        "maxValuesPerFacet": 100,
        "sortFacetValuesBy": {
            "*": "alpha"
        }
    }
```

All properties must be returned when the resource is retrieved.

##### 3.2.2.1.2. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 If the index does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.2.2.2. `PATCH` - `indexes/:index_uid/settings/faceting`

Allow customizing partially the settings of an index for faceting.

Request payload

```json
{
    "maxValuesPerFacet": 10
}
```

`202 Accepted` - Example Response

```json
{
    "uid": 42,
    "indexUid": "books",
    "status": "enqueued",
    "type": "settingsUpdate",
    "enqueuedAt": "2022-03-01T18:39:29.228155Z"
}
```

##### 3.2.2.2.1. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.2.2.2.2. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid JSON payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a value different from `null` or with a different type than `Integer` for the `maxValuesPerFacet` field returns
an [invalid_settings_faceting](0061-error-format-and-definitions.md#invalid_settings_faceting) error.
- TODO: ERRORS DEFINITION

###### 3.2.2.2.2.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.2.2.2.1. Response Definition](#32221-response-definition).

> Otherwise, Meilisearch will create the index in a lazy way. See [3.2.2.2.3. Lazy Index Creation](#32223-lazy-index-creation).

##### 3.2.2.2.3. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.2.2.2.2.1. Async Errors](#322221-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.2.2.2.1. Response Definition](#32221-response-definition).

#### 3.2.2.3. `DELETE`- `indexes/:index_uid/settings/faceting`

Allow resetting the faceting setting to the default for an index.

##### 3.2.2.3.1. Response Definition

When the request is in a successful state, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.2.2.3.2. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.

###### 3.2.2.3.2.1. Asynchronous Index Not Found Error

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related async `task` resource. See [3.2.2.3.1. Response Definition](#32231-response-definition).

### 3.2.3. General Errors

These errors apply to all endpoints described here.

#### 3.2.3.1. Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 2. Technical Details
n/a

## 3. Future Possibilities

- Introduces a field to define the sorting of facets. e.g `sort` by `count`/`alphanumeric`