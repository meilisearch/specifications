# Documents API

## 1. Summary

This specification describes the documents API endpoints permitting to list, fetch, add/replace, and delete index documents.

It is an API dedicated to document management within the Meilisearch index.

## 2. Motivation
N/A

## 3. Functional Specification

Documents are objects composed of fields that can store any type of data.
Each field contains an attribute and its associated value.

Documents are stored inside indexes.

### 3.1. API Endpoints Definition

Manipulate documents of a Meilisearch index.

- [3.1.1. (Fetch Documents) - `GET` - `indexes/:index_uid/documents` and `POST` - `indexes/:index_uid/documents/fetch](#311-fetch-documents-get-indexesindexuiddocuments-and-post-indexesindexuiddocumentsfetch)
- [3.1.2. `GET` - `indexes/:index_uid/documents/:document_id`](#312-get---indexesindexuiddocumentsdocumentid)
- [3.1.3. `POST` - `indexes/:index_uid/documents`](#313-post---indexesindexuiddocuments)
- [3.1.4. `PUT` - `indexes/:index_uid/documents`](#314-put---indexesindexuiddocuments)
- [3.1.5. `DELETE` - `indexes/:index_uid/documents`](#315-delete---indexesindexuiddocuments)
- [3.1.6. `DELETE` - `indexes/:index_uid/documents/:document_id`](#316-delete---indexesindexuiddocumentsdocumentid)
- [3.1.7. `POST` - `indexes/:index_uid/documents/delete-batch`](#317-post---indexesindexuiddocumentsdelete-batch)

#### 3.1.1. (Fetch Documents) - `GET` - `indexes/:index_uid/documents` and `POST` - `indexes/:index_uid/documents/fetch

Meilisearch exposes 2 routes to get the documents:
- GET `indexes/:index_uid/documents`, which gets its parameters as query parameters.
- POST `indexes/:index_uid/documents/fetch`, which gets its parameters in a JSON payload.

List all documents of a Meilisearch index.

The query parameters `offset` and `limit` permit browsing through all documents of an index.

Meilisearch orders documents depending on the order they were inserted in the db.

##### 3.1.1.1. Path Parameters

| Parameters               | Type                     | Required |
|--------------------------|--------------------------|----------|
| `index_uid`              | String                   | true     |

###### 3.1.1.1.1. `index_uid`

- Type: String
- Required: True

Unique identifier of an index.

##### 3.1.1.2. Parameters

The following parameters need to be send as:
- Query parameter for the `GET` - `indexes/:index_uid/documents` route.
- JSON body for the `POST `indexes/:index_uid/documents/fetch` route.

| Field                    | Type                      | Required |
|--------------------------|---------------------------|----------|
| `offset`                 | Integer                   | false    |
| `limit`                  | Integer                   | false    |
| `fields`                 | Array of Strings / `null` | false    |
| `filter`                 | filter / `null`           | false    |

In the case of the query parameter, as always, the `filter` can only be a string, while it can be either a string, an array of strings, or an array of array of strings for the JSON body.

###### 3.1.1.2.1. `offset`

- Type: Integer
- Required: False
- Default: `0`

Sets the starting point in the results, effectively skipping over a given number of documents.

###### 3.1.1.2.2. `limit`

- Type: Integer
- Required: False
- Default: `20`

Sets the maximum number of documents to be returned by the current request.

###### 3.1.1.2.3. `fields`

- Type: Array of Strings
- Required: False
- Default: `*`

Configures which attributes will be retrieved in the returned documents.

If `fields` is not specified, all attributes from the documents are returned in the response. It's equivalent to `fields=*`.

- Sending `fields` without specifying a value, returns empty documents ressources. `fields=`.
- Sending `fields` with a non-existent field as part of the value will not return an error, the non-existent field will not be displayed.

> `fields` values are case-sensitive.

> Specified fields have to be separated by a comma. e.g. `&fields=title,description`

> The index setting `displayedAttributes` has no impact on this endpoint.

###### 3.1.1.2.4. `filter`

- Type: String | Array of array of Strings
- Required: False
- Default: null

Refine the results by selecting documents that match the given filter.
In the case of the POST route, it is possible to send the filter in the form of an array of array of strings akin to the search route.

Attributes used as filter criteria must be added to the `filterableAttributes` list of an index settings. See [Filterable Attributes Setting API](0123-filterable-attributes-setting-api.md).

##### 3.1.1.3. Response Definition

A `results` array representing documents as JSON objects.

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `results`                | Array[Document]          | true     |
| `offset`                 | Integer                  | true     |
| `limit`                  | Integer                  | true     |
| `total`                  | Integer                  | true     |

###### 3.1.1.3.1. `results`

- Type: Array[Document]
- Required: True

An array containing the fetched documents.

###### 3.1.1.3.2. `offset`

- Type: Integer
- Required: True

Gives the `offset` parameter used for the query.

> See [3.1.1.2.1. `offset`](#31121-offset) section.

###### 3.1.1.3.3. `limit`

- Type: Integer
- Required: True

Gives the `limit` parameter used for the query.

> See [3.1.1.2.2. `limit`](#31122-limit) section.

###### 3.1.1.3.3. `total`

- Type: Integer
- Required: True

Gives the total number of documents that can be browsed in the related index.

###### 3.1.1.3.4. Example

```json
{
  "results": [
    {
      "id": 25684,
      "title": "American Ninja 5"
    },
    {
      "id": 468219,
      "title": "Dead in a Week (Or Your Money Back)"
    }
  ],
  "offset": 0,
  "limit": 2,
  "total": 3 //The index contains 3 documents in total
}
```

##### 3.1.1.4. Errors

 - 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a value with a different type than `Integer` or `null` for `offset` will return a [invalid_document_offset](0061-error-format-and-definitions.md#invalid_document_offset) error.
- 🔴 Sending a value with a different type than `Integer` or `null` for `limit` will return a [invalid_document_limit](0061-error-format-and-definitions.md#invalid_document_limit) error.
- 🔴 Sending a value with a different type than `String` or `null` for `fields` will return a [invalid_document_fields](0061-error-format-and-definitions.md#invalid_document_fields) error.
- 🔴 Sending a value with a different type than `String`, `Array of strings`, `Array of array of strings` or `null` for `filter` will return a [invalid_document_filter](0061-error-format-and-definitions.md#invalid_document_filter) error.

#### 3.1.2. `GET` - `indexes/:index_uid/documents/:document_id`

Get a document using its unique id.

##### 3.1.2.1. Path Parameters

| Parameters               | Type                     | Required |
|--------------------------|--------------------------|----------|
| `index_uid`              | String                   | true     |
| `document_id`            | String / `null`          | true     |

###### 3.1.2.1.1. `index_uid`

- Type: String
- Required: True

Unique identifier of an index.

###### 3.1.2.1.2. `document_id`

- Type: Integer
- Required: True

Unique identifier of a document.

##### 3.1.2.2. Query Parameters

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `fields`                 | String / `null`          | false    |

###### 3.1.2.2.1. `fields`

- Type: String
- Required: False
- Default: `*`

Configures which attributes will be retrieved in the returned documents.

If `fields` is not specified, all attributes from the documents are returned in the response. It's equivalent to `fields=*`.

- Sending `fields` without specifying a value, returns empty documents ressources. `fields=`.
- Sending `fields` with a non-existent field as part of the value will not return an error, the non-existent field will not be displayed.

> `fields` values are case-sensitive.

> Specified fields have to be separated by a comma. e.g. `&fields=title,description`

> The index setting `displayedAttributes` has no impact on this endpoint.

##### 3.1.2.3. Request Payload Definition
N/A

##### 3.1.2.4. Response Definition

A document represented as a JSON object.

##### 3.1.2.4.1. Example

```json
{
  "id": 25684,
  "title": "American Ninja 5",
  "poster": "https://image.tmdb.org/t/p/w1280/iuAQVI4mvjI83wnirpD8GVNRVuY.jpg",
  "overview": "When a scientists daughter is kidnapped, American Ninja, attempts to find her, but this time he teams up with a youngster he has trained in the ways of the ninja.",
  "release_date": "1993-01-01"
}
```

##### 3.1.2.5. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.
- 🔴 If the requested `document_id` does not exist, the API returns an [document_not_found](0061-error-format-and-definitions.md#document_not_found) error.
- 🔴 Sending a value with a different type than `String` or `null` for `fields` will return a [invalid_document_fields](0061-error-format-and-definitions.md#invalid_document_fields) error.

#### 3.1.3. `POST` - `indexes/:index_uid/documents`

Add a list of documents or replace them if they already exist.

If the provided index does not exist, it will be created. See [3.1.3.5. Lazy Index Creation](#3145-lazy-index-creation)

If an already existing document (same document id) is sent, the whole existing document will be overwritten by the new document.

Fields that are no longer present in the new document are removed.

This endpoint accepts various content-type:

- [`application/json`](0135-indexing-json.md)
- [`text/csv`](0028-indexing-csv.md)
- [`application/x-ndjson`](0029-indexing-ndjson.md)

##### 3.1.3.1. Path Parameters

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `index_uid`              | String                   | True     |

###### 3.1.3.1.1 `index_uid`

- Type: String
- Required: True

Unique identifier of an index.

##### 3.1.3.2. Request Payload Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `primaryKey`             | String                   | False    |

###### 3.1.3.2.1 `primaryKey`

- Type: String
- Required: False
- Default: `null`

Allows to bypass the auto-inference mechanism of the document identifiers.

By default, the `primaryKey` will be chosen by the auto-inference mechanism by the engine when a first document is indexed.

Specifying this field tells the engine to use the document attribute specified in `primaryKey` and bypasses this mechanism.

When the index is empty, it is possible to modify the `primaryKey`.

If the index is not empty, the query parameter `primaryKey` is ignored.

##### 3.1.3.3. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.1.3.4. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json`, `text/csv`, or `application/x-ndjson` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a value with a different type than `String` or `null` for the `primaryKey` parameter will return a [invalid_index_primary_key](0061-error-format-and-definitions.md#invalid_index_primary_key) error.

###### 3.1.3.4.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.2.2.2. Response Definition](#3222-response-definition).
- 🔴 When updating the `primaryKey`, if the previous `primaryKey` value has already been used for a document, the API returns an [index_primary_key_already_exists](0061-error-format-and-definitions.md#index_primary_key_already_exists) error.

##### 3.1.3.5. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.1.3.4.1. Async Errors](#31341-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.1.3.3. Response Definition](#3133-response-definition).

#### 3.1.4. `PUT` - `indexes/:index_uid/documents`

Add a list of documents or update them if they already exist.

If the provided index does not exist, it will be created. See [3.1.4.5. Lazy Index Creation](#3145-lazy-index-creation)

If an already existing document (same identifier) is set, the old document will be only partially updated according to the fields of the new document. Thus, any fields not present in the new document are kept and remain unchanged.

This endpoint accepts various content-type:

- [`application/json`](0135-indexing-json.md)
- [`text/csv`](0028-indexing-csv.md)
- [`application/x-ndjson`](0029-indexing-ndjson.md)

##### 3.1.4.1. Path Parameters

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `index_uid`              | String                   | True     |

###### 3.1.4.1.1. `index_uid`

- Type: String
- Required: True

Unique identifier of an index.

##### 3.1.4.2. Query Parameters Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `primaryKey`             | String                   | False    |

###### 3.1.4.2.1. `primaryKey`

- Type: String
- Required: False
- Default: `null`

Allows to bypass the auto-inference mechanism of the document identifiers.

By default, the `primaryKey` will be chosen by the auto-inference mechanism by the engine when a first document is indexed.

Specifying this field tells the engine to use the document attribute specified in `primaryKey` and bypasses this mechanism.

When the index is empty, it is possible to modify the `primaryKey`.

If the index is not empty, the query parameter `primaryKey` is ignored.

##### 3.1.4.3. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.1.4.4. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json`, `text/csv`, or `application/x-ndjson` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a value with a different type than `String` or `null` for the `primaryKey` parameter will return a [invalid_index_primary_key](0061-error-format-and-definitions.md#invalid_index_primary_key) error.

###### 3.1.4.4.1. Async Errors

- 🔴 When Meilisearch is secured, if the API Key do not have the `indexes.create` action defined, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error in the related asynchronous `task` resource. See [3.2.2.2. Response Definition](#3222-response-definition).
- 🔴 When updating the `primaryKey`, if the previous `primaryKey` value has already been used for a document, the API returns an [index_primary_key_already_exists](0061-error-format-and-definitions.md#index_primary_key_already_exists) error.

##### 3.1.4.5. Lazy Index Creation

If the requested `index_uid` does not exist, and the authorization layer allows it (See [3.1.4.4.1. Async Errors](#31441-async-errors)), Meilisearch will create the index when the related asynchronous task resource is executed. See [3.1.4.3. Response Definition](#3143-response-definition).

#### 3.1.5. `DELETE` - `indexes/:index_uid/documents`

Delete all documents in the specified index.

##### 3.1.5.1. Path Parameters

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `index_uid`              | String                   | true     |

###### 3.1.5.1.1. `index_uid`

- Type: String
- Required: True

Unique identifier of an index.

##### 3.1.5.2. Request Payload Definition
N/A

##### 3.1.5.3. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.1.5.4. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.

###### 3.1.5.4.1. Async Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.1.6. `DELETE` - `indexes/:index_uid/documents/:document_id`

Delete one document based on its unique id.

##### 3.1.6.1. Path Parameters

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `index_uid`              | String                   | True     |
| `document_id`            | String                   | True     |

###### 3.1.6.1.1. `index_uid`

- Type: String
- Required: True

Unique identifier of an index.

###### 3.1.6.1.2. `document_id`

- Type: Integer
- Required: True

Unique identifier of a document.

##### 3.1.6.2. Request Payload Definition
N/A

##### 3.1.6.3. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.1.6.4. Errors

- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.

###### 3.1.6.4.1. Async Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.1.7. `POST` - `indexes/:index_uid/documents/delete-batch`

Delete a selection of documents based on array of document id's.

##### 3.1.7.1. Path Parameters

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `index_uid`              | String                   | True     |

###### 3.1.7.1.1. `index_uid`

- Type: String
- Required: True

Unique identifier of an index.

##### 3.1.7.2 Request Payload Definition

An array of document ids to delete.

- Type: Array
- Required: True

e.g.

```json
[122, 1194, 2501]
```

##### 3.1.7.3. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

##### 3.1.7.4. Errors

- 🔴 Omitting Content-Type header returns a [missing_content_type](0061-error-format-and-definitions.md#missing_content_type) error.
- 🔴 Sending an empty Content-Type returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending a different Content-Type than `application/json` returns an [invalid_content_type](0061-error-format-and-definitions.md#invalid_content_type) error.
- 🔴 Sending an empty payload returns a [missing_payload](0061-error-format-and-definitions.md#missing_payload) error.
- 🔴 Sending an invalid payload returns a [malformed_payload](0061-error-format-and-definitions.md#malformed_payload) error.
- 🔴 Sending an invalid index uid format for the `:index_uid` path parameter returns an [invalid_index_uid](0061-error-format-and-definitions.md#invalid_index_uid) error.
- 🔴 Sending a value with a different type than `Array` for the body request will return a [bad_request](0061-error-format-and-definitions.md#bad_request) error.

###### 3.1.7.4.1 Async Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.1.8. General Errors

These errors apply to all endpoints described here.

##### 3.1.8.1 Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 4. Technical Details
N/A

## 5. Future Possibilities

- Introduce a way to reject fields from a document in the response. e.g. `?fields=-createdAt`