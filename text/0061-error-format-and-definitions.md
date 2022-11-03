- Title: Error Format and Definitions
- Start Date: 2021-08-13
- Specification PR: [#61](https://github.com/meilisearch/specifications/pull/61)
- Discovery Issue: [#44](https://github.com/meilisearch/product/issues/44#issuecomment-896121211)

# Error Format

## 1. Functional Specification

### I. Summary

The error format is updated to no longer contain unnecessary `error` prefix terms in the names of these attributes. e.g. `errorType` becomes `type`. The context is clear enough to understand that this is an `error` object.

This specification also serves as a reference point for the complete list of API errors that the user may encounter.

#### Summary Key Points

- `error` is removed from the attributes name and `type` values.
- An exhaustive list of API errors is defined with their message variants. See Errors List part.

### II. Motivation

The motivation is to stabilize the current `error` resource to a version that conforms to our API convention and thus allows future evolutions on a more solid base. This specification avoids adding unnecessary information in the error object's attribute names.

The second motivation is to describe in an exhaustive way all the errors that the user may encounter during his use of the API. This list will be kept up to date.

### III. Explanation

#### Error Format

##### Attributes

| Field name | type   | Description                                                              |
|------------|--------|--------------------------------------------------------------------------|
| message    | string |  A human-readable message providing context and details about the error. |
| code       | string |  A short string indicating the error code reported.                      |
| type       | string |  The type of error returned. `invalid_request`, `internal`, `auth`.      |
| link       | string |  An URL to the related error-page details for further information.       |

##### Json Response Example

e.g. 401 Unauthorized Response example

```json
{
    "message": "The Authorization header is missing. It must use the bearer authorization method.",
    "code": "missing_authorization_header",
    "type": "auth",
    "link": "https://docs.meilisearch.com/errors#missing_authorization_header"
}
```

- 💡 The error object fields order must conform to the example.

##### type enum

| type            | description                                                                                       |
|-----------------|---------------------------------------------------------------------------------------------------|
| invalid_request | This type of error is usually due to a user input error. It is accompanied by an HTTP code `4xx`. |
| internal        | Usually, this type of error is returned because the search engine can't operate due to machine or configuration constraints. It can be due to limits being reached, such as the size of the disk, the size limit of an index, etc. It can also be an unexpected error. It is accompanied by an HTTP code `5xx`.  |
| auth   | This type of error is returned when it comes to authentication and authorization. It is accompanied by an HTTP code `4xx`. |

---

#### Error list

Following this format, here is the exhaustive list of errors that MeiliSearch can return to an API consumer. This list is updated as MeiliSearch evolves.

Errors can be returned in two different ways: `Synchronous` or `Asynchronous`.

💡 `Synchronous` errors are returned directly by the API in response to a user's request.

💡 Errors returned asynchronously in a `task` object do not include a definition of the HTTP code. An asynchronous error is returned in the payload of a `task` under the `error` object.

# invalid_request type

## bad_request

### Context

This error code is generic. It should not be used. Instead, a clear and precise error code should be determined to guide the user efficiently.

### Error Definition

HTTP code `400 Bad Request`

```json
{
    "message": ":errorMessage",
    "code": "bad_request",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#bad_request"
}
```

- The `:errorMessage` is inferred when the message is generated.

---

## missing_parameter

`Synchronous`

### Context

This error happens when a field is mandatory and missing in the given payload.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:fieldName` field is mandatory.",
    "code": "missing_parameter",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#missing_parameter"
}
```

- The `:fieldname` is inferred when the message is generated.

---

## immutable_field

`Synchronous` / `Asynchronous`

### Context

This error happens when an immutable field is given in a payload dedicated to modify a resource.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `:fieldName` field cannot be modified for the given resource.",
    "code": "immutable_field",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_field"
}
```

- The `:fieldName` is inferred when the message is generated.

--

## api_key_already_exists

`Synchronous`

### Context

This error happens when a user tries to create an API Key that already exists for the given `uid`.

### Error Definition

HTTP Code: `409 Conflict`

```json
{
    "message": "`uid` field value `:value` is already an existing API key.",
    "code": "api_key_already_exists",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#api_key_already_exists"
}
```

- The `:value` is inferred when the message is generated.

---
## invalid_api_key_uid

`Synchronous`

### Context

This error happens when the `uid` field for an `API Key` resource is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`uid` field value `:value` is invalid. It should be a valid UUID v4 string or omitted.",
    "code": "invalid_api_key_uid",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key_uid"
}
```

- The `:value` is inferred when the message is generated.

---

## invalid_api_key_name

`Synchronous`

### Context

This error happens when the `name` field for an `API Key` resource is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`name` field value `:value` is invalid. It should be a string or specified as a null value.",
    "code": "invalid_api_key_name",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key_name"
}
```

- The `:value` is inferred when the message is generated.

---

## invalid_api_key_description

`Synchronous`

### Context

This error happens when the `description` field for an `API Key` resource is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`description` field value `:value` is invalid. It should be a string or specified as a null value.",
    "code": "invalid_api_key_description",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key_description"
}
```

- The `:value` is inferred when the message is generated.

---

## invalid_api_key_actions

`Synchronous`

### Context

This error happens when the `actions` field for an `API Key` resource is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`actions` field value `:value` is invalid. It should be an array of string representing action names.",
    "code": "invalid_api_key_actions",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key_actions"
}
```

- The `:value` is inferred when the message is generated.

---

## invalid_api_key_indexes

`Synchronous`

### Context

This error happens when the `indexes` field for an `API Key` resource is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`indexes` field value `:value` is invalid. It should be an array of string representing index names.",
    "code": "invalid_api_key_indexes",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key_indexes"
}
```

- The `:value` is inferred when the message is generated.

---

## invalid_api_key_expires_at

`Synchronous`

### Context

This error happens when the `expiresAt` field for an `API Key` resource is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`expiresAt` field value `:value` is invalid. It should follow the RFC 3339 format to represents a date or datetime in the future or specified as a null value. e.g. 'YYYY-MM-DD' or 'YYYY-MM-DD HH:MM:SS'.",
    "code": "invalid_api_key_expires_at",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key_expires_at"
}
```

- The `:value` is inferred when the message is generated.

---

## invalid_typo_tolerance_min_word_size_for_typos

`Asynchronous`

### Context

This error happens when the `minWordSizeForTypos` object of the `typo` resource is invalid.

### Error Definition

```json
{
    "message": "`minWordSizeForTypos` setting is invalid. `oneTypo` and `twoTypos` fields should be between `0` and `255`, and `twoTypos` should be greater or equals to `oneTypo` but found `oneTypo: :oneTypo` and twoTypos: twoTypos`.",
    "code": "invalid_typo_tolerance_min_word_size_for_typos",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_typo_tolerance_min_word_size_for_typos"
}
```

- The `:oneTypo` is inferred when the message is generated.
- The `:twoTypos` is inferred when the message is generated.

---

## index_already_exists

`Asynchronous`

### Context

This error happens when a user tries to create an index that already exists.

### Error Definition

```json
{
    "message": "Index `:uid` already exists.",
    "code": "index_already_exists",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#index_already_exists"
}
```

- The `:uid` is inferred when the message is generated.

---

## invalid_index_uid

`Synchronous`

### Context

This error happens when a user tries to create an index with an invalid uid format.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
     "message": "`uid` is not a valid index uid. Index uid can be an integer or a string containing only alphanumeric characters, hyphens (-) and underscores (_).",
    "code": "invalid_index_uid",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_index_uid"
}
```

- The `:uid` is inferred when the message is generated.

---

## index_primary_key_already_exists

`Asynchronous`

### Context

This error happens when a user tries to update an index primary key while the index already has one primary key.

### Error Definition

```json
{
    "message": "Index already has a primary key: `:primaryKey`.",
    "code": "index_primary_key_already_exists",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#index_primary_key_already_exists"
}
```

- The `:primaryKey` is inferred when the message is generated.

---

## primary_key_inference_failed

`Asynchronous`

### Context

This error occurs when the engine does not find an identifier in the payload documents to define it as the primary key of the index during the inference process when no document has already been inserted.

### Error Definition

```json
{
    "message": "The primary key inference process failed because the engine did not find any fields containing `id` substring in their name. If your document identifier does not contain any `id` substring, you can set the primary key of the index.",
    "code": "primary_key_inference_failed",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#primary_key_inference_failed"
}
```

---

## missing_document_id

`Asynchronous`

### Context

This error occurs when the engine does not find the primary key previously defined for the index in the document payload.

### Error Definition

```json
{
    "message": "Document doesn't have a `:primaryKey` attribute: `:documentRepresentation`.",
    "code": "missing_document_id",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_document_id"
}
```

- The `:primaryKey` is inferred when the message is generated. This is the value of the primaryKey attribute of the related index object.
- The `:documentRepresentation` is inferred when the message is generated.

---

## invalid_document_id

`Asynchronous`

### Context

This error occurs when the value of a document identifier does not meet the requirements of the engine.

### Error Definition

```json
{
    "message": "Document identifier `:documentId` is invalid. A document identifier can be of type integer or string, only composed of alphanumeric characters (a-z A-Z 0-9), hyphens (-) and underscores (_).",
    "code": "invalid_document_id",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_document_id"
}
```

- The `:documentId` is inferred when the message is generated.

---

## document_fields_limit_reached

`Synchronous` — The error can be synchronous if a document with a number higher than the allowed field limit is sent.

`Asynchronous` — The error can be asynchronous if the limit is reached when adding one or many fields during a document update.

### Context

The maximum number of fields for a document is `65,535`. When this number is exceeded, this error is returned. This error is returned within a `task` for a `documentsAddition` or `documentsPartial` operation.

### Error Definition

```json
{
    "message": "A document cannot contain more than 65,535 fields.",
    "code": "document_fields_limit_reached",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#documents_fields_limit_reached"
}
```

---

## invalid_ranking_rule

`Asynchronous`

### Context

This error occurs when the user specifies a non-existent ranking rule, a malformed custom ranking rule in the settings payload, or tries to specify a custom ranking rule on the reserved keywords `_geo` and `_geoDistance`.

### Error Definition

#### Variant: Sending an inexistent ranking rule or an invalid custom ranking rule syntax.

```json
{
    "message": "`:rankingRule` ranking rule is invalid. Valid ranking rules are words, typo, sort, proximity, attribute, exactness and custom ranking rules.",
    "code": "invalid_ranking_rule",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_ranking_rule"
}
```

- The `:rankingRule` is inferred when the message is generated. It could be a misspelled ranking rule like `Wards` instead of `Words` or a custom ranking rule expressed with a syntax error.

#### Variant: Specifying a custom ranking rule on reserved fields `_geo` or `_geoDistance`

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a ranking rule.",
    ...
}
```

- The `:reservedKeyword` is inferred when the message is generated.

#### Variant: Specifying a custom ranking rule on reserved expression `_geoPoint`

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a ranking rule. `:reservedKeyword` can only be used for sorting at search time.",
    ...
}
```

- The `:reservedKeyword` is inferred when the message is generated.

#### Variant: Specifying a custom ranking rule on reserved expression `_geoRadius`

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a ranking rule. `:reservedKeyword` can only be used for filtering at search time.",
    ...
}
```

- The `:reservedKeyword` is inferred when the message is generated.

---

## invalid_filter

`Synchronous`

### Context

This error occurs at search time when there is a syntax error in the `filter` parameter, when an attribute expressed in the filter is not defined in the `filterableAttributes` list or when a reserved keywords like `_geo`, `_geoDistance` and `_geoPoint` is used as a filter.

### Error Definition

HTTP Code: `400 Bad Request`

#### Variant: Filtering on a non filterable attribute

```json
{
    "message": "Attribute `:attribute` is not filterable. Available filterable attributes are: `:filterableAttributes`.",
    "code": "invalid_filter",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_filter"
}
```

- The `:attribute` is inferred when the message is generated.
- The `:filterableAttributes` is inferred when the message is generated. It contains the list of filterable attributes separated by a comma. `filterableAttribute1, filterableAttribute2, ...`

#### Variant: Filtering on a non filterable attribute when `filterableAttributes` is empty

```json
{
    "message": "Attribute `:attribute` is not filterable. This index does not have configured filterable attributes.",
    "code": "invalid_filter",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_filter"
}
```

- `:attribute` is inferred when the message is generated.

#### Variant: Using `_geoDistance` as a filter expression

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a filter expression.",
    ...
}
```

- The `:reservedKeyword` is inferred when the message is generated.

#### Variant: Using `_geo` or `_geoPoint` as a filter expression

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a filter expression. Use the _geoRadius(latitude, longitude, distance) built-in rule to filter on _geo field coordinates.",
    ...
}
```

- The `:reservedKeyword` is inferred when the message is generated.

#### Variant: Invalid syntax for the `filter` parameter

```json
{
    "message": "Invalid syntax for the filter parameter: `:syntaxErrorHelper`.",
    ...
}
```

- The `:syntaxErrorHelper` is inferred when the message is generated.

---

## invalid_sort

`Synchronous`

### Context

This error occurs at search time when there is a syntax error in the `sort` parameter, when an attribute expressed in the sort is not defined in the `sortableAttributes` list, sort at search time while the `sort` ranking rule is missing from the settings, or using reserved keywords like `_geo`, `_geoDistance` and `_geoRadius` as a sort expression.

### Error Definition:

HTTP Code: `400 Bad Request`

#### Variant: Sorting on a non sortable attribute

```json
{
    "message": "Attribute `:attribute` is not sortable. Available sortable attributes are: `:sortableAttributes`.",
    "code": "invalid_sort",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_sort"
}
```

- The `:attribute` is inferred when the message is generated.
- The `:sortableAttributes` is inferred when the message is generated. It contains the list of sortable attributes separated by a comma. `sortableAttribute1, sortableAttribute2, ...`

#### Variant: Sorting on a non sortable attribute when `sortableAttributes` is empty

```json
{
    "message": "Attribute `:attribute` is not sortable. This index does not have configured sortable attributes.",
    "code": "invalid_sort",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_sort"
}
```

- The `:attribute` is inferred when the message is generated.

#### Variant: Using `_geoDistance` as a sort expression

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a sort expression.",
    ...
}
```

- The `:reservedKeyword` is inferred when the message is generated.

#### Variant: Using `_geo` or `_geoRadius` as a sort expression
```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a sort expression. Use the _geoPoint(latitude, longitude) built-in rule to sort on _geo field coordinates.",
    ...
}
```

- The `:reservedKeyword` is inferred when the message is generated.

#### Variant: Invalid syntax for the `sort`parameter

```json
{
    "message": "Invalid syntax for the sort parameter: `:syntaxErrorHelper`.",
    ...
}
```

- The `:syntaxErrorHelper` is inferred when the message is generated.

#### Variant: Specifying `sort` at search time while the sort ranking rule isn't set in the ranking rules settings

```json
{
    "message": "The sort ranking rule must be specified in the ranking rules settings to use the sort parameter at search time.",
    ...
}
```

---

## invalid_geo_field

`Asynchronous`

### Context

These errors occurs when the `_geo` field of a document payload is not valid. Either the latitude / longitude is missing or is not a number.

### Error Definition

#### Variant: `_geo` field is not an object.

```json
{
    "message": "The `_geo` field in the document with the id: `:documentId` is not an object. Was expecting an object with the `_geo.lat` and `_geo.lng` fields but instead got `:field`.",
    "code": "invalid_geo_field",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_geo_field"
}
```

#### Variant: Missing `_geo.lat` **and** `_geo.lng` field.

```json
{
    "message": "Could not find latitude nor longitude in the document with the id: `:documentId`. Was expecting `_geo.lat` and `_geo.lng` fields.",
    "code": "invalid_geo_field",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_geo_field"
}
```

#### Variant: Missing `_geo.lat` **or** `_geo.lng` field.

```json
{
    "message": "Could not find :coord in the document with the id: `:documentId`. Was expecting a `:field` field.",
    "code": "invalid_geo_field",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_geo_field"
}
```

#### Variant: Coordinate can't be parsed.

```json
{
    "message": "Could not parse :coord in the document with the id: `:documentId`. Was expecting a finite number but instead got `:value`.",
    "code": "invalid_geo_field",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_geo_field"
}
```

- The `:documentId` is inferred when the message is generated.
- The `:coord` is either `latitude` or `longitude` depending on what's wrong.
- The `:field` is either `_geo.lat` or `_geo.lng` depending on what's wrong.

---

## payload_too_large

`Synchronous`

### Context

This error occurs when the size of the payload sent exceeds the limit set by the server. The user can correct this error by reducing the payload size or increasing the limit with [this configuration variable](https://docs.meilisearch.com/reference/features/configuration.html#payload-limit-size)

### Error Definition

HTTP Code: `413 Payload Too Large`

```json
{
    "message": "The provided payload reached the size limit.",
    "code": "payload_too_large",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#payload_too_large"
}
```

---

## not_found

### Context

This error code is generic. It should not be used. Instead, a clear and precise error code should be determined.

---

## index_not_found

`Asynchronous` / `Synchronous`

### Context

This error happens when a requested index can't be found.

### Error Definition

HTTP Code: `404 Not Found` when

#### Variant: Multiples indexUids can't be found

```json
{
    "message": "Indexes `indexUids` not found.",
    "code": "index_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#index_not_found"
}
```

- The `:indexUids` is inferred when the message is generated, values are separated by `,`.

#### Variant: An index can't be found

```json
{
    "message": "Index `:indexUid` not found.",
    "code": "index_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#index_not_found"
}
```

- The `:indexUid` is inferred when the message is generated.

---

## duplicate_index_found

`Synchronous`

### Context

This error happens when the same indexUid is used twice in the `POST`- `swap-indexes` payload.

### Error Definition


#### Variant: A single indexUid is found twice in the payload

```json
{
    "message": "Indexes must be declared only once during a swap. `:indexUid` was specified several times.",
    "code": "duplicate_index_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#duplicate_index_found"
}
```

- The `:indexUid` is inferred when the message is generated.

#### Variant: Several indexUids are found twice in the payload

```json
{
    "message": "Indexes must be declared only once during a swap. `:indexUids` were specified several times.",
    "code": "duplicate_index_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#duplicate_index_found"
}
```

- The `:indexUids` is inferred when the message is generated, values are separated by `,`.

---

## document_not_found

`Synchronous`

## Context

This error happens when a requested document can't be found.

## Error Definition

HTTP Code: `404 Not Found`

```json
{
    "message": "Document `:documentId` not found.",
    "code": "document_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#document_not_found"
}
```

- The `:documentId` is inferred when the message is generated.

---

## dump_not_found

`Synchronous`

### Context

This error happens when a requested dump can't be found.

### Error Definition

HTTP Code: `404 Not Found`

```json
{
    "message": "Dump `:dumpId` not found.",
    "code": "dump_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#dump_not_found"
}
```

- The `:dumpId` is inferred when the message is generated.

---

## task_not_found

`Synchronous`

#### Context

This error happens when a requested task can't be found.

#### Error Definition

HTTP Code: `404 Not Found`

```json
{
    "message": "Task `:taskUid` not found.",
    "code": "task_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#task_not_found"
}
```

- The `:taskUid` is inferred when the message is generated.

---

## invalid_task_status

### Context

This error happens when a requested task status is invalid.

#### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task status `:status` is invalid. Available task statuses are: `:taskStatuses`.",
    "code": "invalid_task_status",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#invalid_task_status"
}
```

- The `:status` is inferred when the message is generated.
- The `:taskStatuses` is inferred when the message is generated.

---

## invalid_task_type

### Context

This error happens when a requested task type is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task type `:type` is invalid. Available task types are: `:taskTypes`.",
    "code": "invalid_task_type",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#invalid_task_type"
}
```

- The `:type` is inferred when the message is generated.
- The `:taskTypes` is inferred when the message is generated.

---

## api_key_not_found

`Synchronous`

### Context

This error happens when a requested api key can't be found.

#### Error Definition

HTTP Code: `404 Not Found`

```json
    "message": "API key `:apiKey` not found.",
    "code": "api_key_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#api_key_not_found"
```

- The `:apiKey` is inferred when the message is generated.

---

## invalid_content_type

`Synchronous`

### Context

This error occurs when the provided content-type is not handled by the API method.

### Error Definition

HTTP Code: `415 Unsupported Media Type`

```json
{
    "message": "The Content-Type `:contentType` is invalid. Accepted values for the Content-Type header are: `:contentTypeList`.",
    "code": "invalid_content_type",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_content_type"
}
```

- The `:contentTypeList` is inferred when the message is generated. The values are separated by a `,` char. e.g. `application/json`, `text/csv`.

---

## missing_content_type

`Synchronous`

### Context

This error occurs when the Content-Type header is missing.

### Error Definition

HTTP Code: `415 Unsupported Media Type`

```json
{
    "message": "A Content-Type header is missing. Accepted values for the Content-Type header are: `:contentTypeList`.",
    "code": "missing_content_type",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_content_type"
}
```

- The `:contentTypeList` is inferred when the message is generated. The values are separated by a `,` char. e.g. `application/json`, `text/csv`.

---

## missing_payload

`Synchronous`

### Context

This error occurs when the client does not provide a mandatory payload to the request.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "A `:payloadType` payload is missing.",
    "code": "missing_payload",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_payload"
}
```

- The `:payloadType` is inferred when the message is generated. e.g. `json`, `ndjson`, `csv`

---

## malformed_payload

`Synchronous`

### Context

This error occurs when the format sent in the payload is malformed. The payload contains a syntax error.

### Error Definition

HTTP Code: `400 Bad Request`

#### Variant: Syntax error

```json
{
    "message": "The `:payloadType` payload provided is malformed. `:syntaxErrorHelper`.",
    "code": "malformed_payload",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#malformed_payload"
}
```

- The `:payloadType` is inferred when the message is generated. e.g. `json`, `ndjson`, `csv`
- The `:syntaxErrorHelper` is inferred when the message is generated.

---

# internal type

## internal

`Asynchronous` / `Synchronous`

### Context

This error code occurs when an unknown and undetermined error has occurred at the server. This is a error that should not happen.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "An internal error has occurred. `:reason`.",
    "code": "internal",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#internal"
}
```

- The `:reason` is inferred when the message is generated. e.g. `Internal: decoding failed`

---

## index_creation_failed

`Asynchronous`

### Context

This error occurs when an index creation could not be completed for various reasons.

### Error Definition

```json
{
    "message": "The creation of the `:uid` index has failed due to `:reason`.",
    "code": "index_creation_failed",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#index_creation_failed"
}
```

- The `:reason` is inferred when the message is generated. e.g. `Write Permission denied`

---

## index_not_accessible

`Asynchronous` / `Synchronous`

### Context

 This error occurs when an index cannot be accessed for various reasons.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "The index `:uid` can't be accessed due to `:reason`.",
    "code": "index_not_accessible",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#index_not_accessible"
}
```

- The `:reason` is inferred when the message is generated. e.g. `Read Permission denied`

---

## unretrievable_document

`Asynchronous` / `Synchronous`

### Context

This error occurs when a document cannot be found in the system due to an inconsistent state that can occur for several reasons.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "The document `:documentId` is unretrievable due to `:reason`.",
    "code": "unretrievable_document",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#unretrievable_document"
}
```

- The `documentId` is inferred when the message is generated.
- The `:reason` is inferred when the message is generated.

---

## invalid_state

`Asynchronous` / `Synchronous`

### Context

This error occurs when the database is in an inconsistent state due to an uncontrolled internal error.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "The database is in an invalid state.",
    "code": "invalid_state",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#invalid_state"
}
```

---

## dump_process_failed

`Asynchronous`

### Context

This error occurs during the dump creation process. The dump creation was interrupted for various reasons.

### Error Definition

```json
{
    "message": "The creation of the dump `:dumpId` failed due to `:reason`.",
    "code": "dump_process_failed",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#dump_process_failed"
}
```

- The `:reason` is inferred when the message is generated. e.g. `Permission denied`

---

## database_size_limit_reached

`Asynchronous`

### Context

This error occurs when the user tries to add documents and the maximum size of the database reaches the limit. The user can correct this error by increasing the database size limit.

### Error Definition

```json
{
    "message": "Maximum database size of `:databaseSizeLimit` has been reached.",
    "code": "database_size_limit_reached",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#database_size_limit_reached"
}
```

- The `:databaseSizeLimit` is inferred when the message is generated. e.g. `100GiB`

---

## no_space_left_on_device

`Asynchronous` / `Synchronous`

### Context

This error occurs when the host system partition has reached its maximum capacity and no longer accepts writes.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "There is no more space left on the device. Consider increasing the size of the disk/partition.",
    "code": "no_space_left_on_device",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#no_space_left_on_device"
}
```

---

## invalid_store_file

`Asynchronous` / `Synchronous`

### Context

This error occurs when the `data.ms` folder is in an inconsistent state. It can happen for various reasons. An .mdb file can be corrupted, the data.ms folder has been replaced by a file, etc.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "The database file is in an invalid state.",
    "code": "invalid_store_file",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#invalid_store_file"
}
```

---

# auth type

## missing_authorization_header

`Synchronous`

### Context

This error occurs when the route is protected, and the `Authorization` header is not provided.

### Error Definition

HTTP Code: `401 Unauthorized`

```json
{
    "message": "The Authorization header is missing. It must use the bearer authorization method.",
    "code": "missing_authorization_header",
    "type": "auth",
    "link": "https://docs.meilisearch.com/errors#missing_authorization_header"
}
```

---

## invalid_api_key

`Synchronous`

### Context

This error occurs when the route is protected, and the value of the `Authorization` header does not allow access to the resource.

### Error Definition

HTTP Code: `403 Forbidden`

```json
{
    "message": "The provided API key is invalid.",
    "code": "invalid_api_key",
    "type": "auth",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key"
}
```

---

## 2. Technical details
N/A

## 3. Future Possibilities

- Add a `parameter` attribute in the `error` object to indicate which request parameter caused an error.
- Add more specific error types. For example, explode internal errors into several distinct types.
