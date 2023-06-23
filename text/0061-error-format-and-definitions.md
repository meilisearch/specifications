# Error Format and Definitions

## 1. Functional Specification

### I. Summary

This specification serves as a reference point for the complete list of API errors that the user may encounter.

### II. Motivation

The motivation is to stabilize the current `error` resource to a version that conforms to our API convention and thus allows future evolutions on a more solid base.

The second motivation is to describe in an exhaustive way all the errors that the user may encounter during his use of the API. This list will be kept up to date.

### III. Explanation

#### Error Format

##### Attributes

| Field name | type   | Description                                                                       |
|------------|--------|-----------------------------------------------------------------------------------|
| message    | string |  A human-readable message providing context and details about the error.          |
| code       | string |  A string indicating the error code reported.                                     |
| type       | string |  The type of error returned. `invalid_request`, `internal`, `system`, and `auth`. |
| link       | string |  An URL to the related error-page details for further information.                |

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
| invalid_request | This type of error is used to indicate an input error. It is accompanied by an HTTP code `4xx`. |
| internal        | This type of error is returned when the search engine can't operate under normal condition.  Most of the time, it's indicating an unexpected error. It is accompanied by an HTTP code `5xx`.  |
| system          | This type of error is used to indicate a system limits being reached, such as the size of the disk, the size limit of an index, etc. It is accompanied by an HTTP code `5xx`. |
| auth            | This type of error is returned when it comes to authentication and authorization. It is accompanied by an HTTP code `4xx`. |

---

#### Error list

Following this format, here is the exhaustive list of errors that MeiliSearch can return to an API consumer. This list is updated as MeiliSearch evolves.

Errors can be returned in two different ways: `Synchronous` or `Asynchronous`.

💡 `Synchronous` errors are returned directly by the API in response to a user's request.

💡 Errors returned asynchronously in a `task` object do not include a definition of the HTTP code. An asynchronous error is returned in the payload of a `task` under the `error` object.

# invalid_request type

## bad_request

### Context

This error code is generic. Whenever an error is thrown for a resource field, a clear and precise error code should be determined to guide the user efficiently.

E.g. Sending an unknow field for a resource raises a generic `bad_request` error.

### Error Definition

HTTP code `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "bad_request",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#bad_request"
}
```

---

## immutable_api_key_uid

`Synchronous`

### Context

This error happens when the `uid` field is given in a payload dedicated to modify an API Key.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `uid` field cannot be modified for the given resource.",
    "code": "immutable_api_key_uid",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#immutable_api_key_uid"
}
```

--

## immutable_api_key_key

`Synchronous`

### Context

This error happens when the `key` field is given in a payload dedicated to modify an API Key.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `key` field cannot be modified for the given resource.",
    "code": "immutable_api_key_uid",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_api_key_key"
}
```

---

## immutable_api_key_actions

`Synchronous`

### Context

This error happens when the `actions` field is given in a payload dedicated to modify an API Key.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `actions` field cannot be modified for the given resource.",
    "code": "immutable_api_key_actions",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_api_key_actions"
}
```

---

## immutable_api_key_indexes

`Synchronous`

### Context

This error happens when the `indexes` field is given in a payload dedicated to modify an API Key.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `indexes` field cannot be modified for the given resource.",
    "code": "immutable_api_key_indexes",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_api_key_indexes"
}
```

---

## immutable_api_key_expires_at

`Synchronous`

### Context

This error happens when the `expiresAt` field is given in a payload dedicated to modify an API Key.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `expiresAt` field cannot be modified for the given resource.",
    "code": "immutable_api_key_expires_at",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_api_key_expires_at"
}
```

___

## immutable_api_key_created_at

`Synchronous`

### Context

This error happens when the `createdAt` field is given in a payload dedicated to modify an API Key.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `createdAt` field cannot be modified for the given resource.",
    "code": "immutable_api_key_created_at",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_api_key_created_at"
}
```

---

## immutable_api_key_updated_at

`Synchronous`

### Context

This error happens when the `updatedAt` field is given in a payload dedicated to modify an API Key.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `updatedAt` field cannot be modified for the given resource.",
    "code": "immutable_api_key_updated_at",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_api_key_updated_at"
}
```

---

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

---

## missing_api_key_actions

`Synchronous`

### Context

This error happens when `actions` is missing from the post api key resource payload.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`actions` field is mandatory.",
    "code": "missing_api_key_actions",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_api_key_actions"
}
```

---

## missing_api_key_indexes

`Synchronous`

### Context

This error happens when `indexes` is missing from the post api key resource payload.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`indexes` field is mandatory.",
    "code": "missing_api_key_indexes",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_api_key_indexes"
}
```

---

## missing_api_key_expires_at

`Synchronous`

### Context

This error happens when `expiresAt` is missing from the post api key resource payload.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`expiresAt` field is mandatory.",
    "code": "missing_api_key_expires_at",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_api_key_expires_at"
}
```

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

#### Variant: Sending an invalid index uid format in the `indexes` field.

```json
{
    "message": "`uid` is not a valid index uid pattern. Index uid patterns can be an integer or a string containing only alphanumeric characters, hyphens (-), underscores (_), and optionally end with a star (*).",
    ...
}
```

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

___

## invalid_api_key_offset

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `offset` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_api_key_offset",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key_offset"
}
```

---

## invalid_api_key_limit

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `limit` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_api_key_limit",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key_limit"
}
```

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

---

## missing_index_uid

`Synchronous`

### Context

This error happens when `uid` is missing from the post index resource payload.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "missing_index_uid",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_index_uid"
}
```

---

## invalid_index_uid

`Synchronous`

### Context

This error happens when:

- a value with a different type than `String` for `uid` is specified
- an invalid index uid format is specified in the `:indexUid` path parameter

### Error Definition

HTTP Code: `400 Bad Request`

#### Variant: Sending a different type than `String` for `uid`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_index_uid",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_index_uid"
}
```

#### Variant: Sending an invalid `String` for `uid`

```json
{
    "message": "`:uid` is not a valid index uid. Index uid can be an integer or a string containing only alphanumeric characters, hyphens (-) and underscores (_).",
    ...
}
```

---

## immutable_index_uid

`Synchronous`

### Context

This error happens when the `uid` field is given in a payload dedicated to modify an index.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `uid` field cannot be modified for the given resource.",
    "code": "immutable_index_uid",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_index_uid"
}
```

---

## immutable_index_created_at

`Synchronous`

### Context

This error happens when the `createdAt` field is given in a payload dedicated to modify an index.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `createdAt` field cannot be modified for the given resource.",
    "code": "immutable_index_created_at",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_index_created_at"
}
```

---

## immutable_index_updated_at

`Synchronous`

### Context

This error happens when the `updatedAt` field is given in a payload dedicated to modify an index.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "The `updatedAt` field cannot be modified for the given resource.",
    "code": "immutable_index_updated_at",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#immutable_index_updated_at"
}
```

---

## invalid_index_limit

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `limit` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_index_limit",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_index_limit"
}
```

---

## invalid_index_offset

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `offset` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_index_offset",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_index_offset"
}
```

---

## invalid_index_primary_key

`Synchronous`

### Context

This error occurs when a value with a different type than `string` or `null` is specified for the `primaryKey` field.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_index_primary_key",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_index_primary_key"
}
```

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

---

## index_primary_key_no_candidate_found

`Asynchronous`

### Context

This error occurs when the engine does not find an identifier in the payload documents to define it as the primary key of the index during the inference process when no document has already been inserted.

### Error Definition

```json
{
    "message": "The primary key inference failed as the engine did not find any field ending with `id` in its name. Please specify the primary key manually using the `primaryKey` query parameter.",
    "code": "index_primary_key_no_candidate_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#index_primary_key_no_candidate_found"
}
```

---

## index_primary_key_multiple_candidates_found

`Asynchronous`

### Error Definition

```json
{
    "message": "The primary key inference failed as the engine found `:numCandidates` fields ending with `id` in their names: '`:firstCandidate`' and '`:secondCandidate`'. Please specify the primary key manually using the `primaryKey` query parameter.",
    "code": "index_primary_key_multiple_candidates_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#index_primary_key_multiple_candidates_found"
}
```

- The `:numCandidates` is inferred when the message is generated. It is the number of fields that could serve as a primary key according to the engine's inference rules.
- The `:firstCandidate` and `:secondCandidate` are inferred when the message is generated. They are the name of two of the fields that could serve as a primary key according to the engine's inference rules.

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

---

## invalid_document_fields

`Synchronous`

### Context

This error occurs if a value with a different type than `String` for `fields` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_document_fields",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_document_fields"
}
```

---

## invalid_document_filter

`Synchronous` / `Asynchronous`

### Context

This error occurs if a value with a different type than `String`, `Array of String` or `Array of array of String` for `filter` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_document_filter",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_document_filter"
}
```

---

## missing_document_filter

`Synchronous`

### Context

This error happens when `filter` is missing from a delete documents by filter operation.

### Error Definition

In the first case:

HTTP Code: `400 Bad Request`

```json
{
    "message": "`filter` field is mandatory.",
    "code": "missing_document_filter",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_document_filter"
}
```

---

## invalid_document_limit

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `limit` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_document_limit",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_document_limit"
}
```

---

## invalid_document_offset

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `offset` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_document_offset",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_document_offset"
}
```

---

## document_fields_limit_reached

`Synchronous` — The error can be synchronous if a document with a number higher than the allowed field limit is sent.

`Asynchronous` — The error can be asynchronous if the limit is reached when adding one or many fields during a document update.

### Context

The maximum number of fields for a document is `65,535`. When this number is exceeded, this error is returned. This error is returned within a `task` for a `documentAdditionOrUpdate` operation.

### Error Definition

HTTP Code: `400 Bad Request` when `Synchronous`

```json
{
    "message": "A document cannot contain more than 65,535 fields.",
    "code": "document_fields_limit_reached",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#documents_fields_limit_reached"
}
```

---

## invalid_settings_displayed_attributes

`Synchronous`

### Context

This error occurs when a value type different of `Array of String`, `[]`,  or `null` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_displayed_attributes",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_displayed_attributes"
}
```

---

## invalid_settings_searchable_attributes

`Synchronous`

### Context

This error occurs when a value type different of `Array of String`, `[]`,  or `null` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_searchable_attributes",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_searchable_attributes"
}
```

---

## invalid_settings_filterable_attributes

`Synchronous`

### Context

This error occurs when a value type different of `Array of String`, `[]`,  or `null` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_filterable_attributes",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_filterable_attributes"
}
```

---

## invalid_settings_sortable_attributes

`Synchronous`

### Context

This error occurs when a value type different of `Array of String`, `[]`,  or `null` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_sortable_attributes",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_sortable_attributes"
}
```

---

## invalid_settings_ranking_rules

`Synchronous`

### Context

This error occurs when:

- an invalid format for the settings payload is specified
- a non-existent ranking rule is specified
- a malformed custom ranking rule is specified
- a custom ranking rule is specified for a reserved keyword

### Error Definition

HTTP Code: `400 Bad Request` when `Synchronous`

#### Variant: Sending an invalid format

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_ranking_rules",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_ranking_rules"
}
```

#### Variant: Sending an inexistent ranking rule or an invalid custom ranking rule syntax.

```json
{
    "message": "`:rankingRule` ranking rule is invalid. Valid ranking rules are words, typo, sort, proximity, attribute, exactness and custom ranking rules.",
}
```

#### Variant: Specifying a custom ranking rule on reserved fields `_geo` or `_geoDistance`

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a ranking rule.",
    ...
}
```

#### Variant: Specifying a custom ranking rule on reserved expression `_geoPoint`

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a ranking rule. `:reservedKeyword` can only be used for sorting at search time.",
    ...
}
```

#### Variant: Specifying a custom ranking rule on reserved expressions `_geoRadius` / `_geoBoundingBox`

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a ranking rule. `:reservedKeyword` can only be used for filtering at search time.",
    ...
}
```

---

## invalid_settings_stop_words

`Synchronous`

### Context

This error occurs when a value type different of `Array of String`, `[]`,  or `null` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_stop_words",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_stop_words"
}
```

---

## invalid_settings_synonyms

`Synchronous`

### Context

This error occurs when a value type different of `Object`,  or `null` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_synonyms",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_synonyms"
}
```

---

## invalid_settings_distinct_attribute

`Synchronous`

### Context

This error occurs when a value type different of `String` or `null` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_distinct_attribute",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_distinct_attribute"
}
```

---

## invalid_settings_typo_tolerance

`Asynchronous` / `Synchronous`

### Context

This error when:

- a value different from `null` or with a different type than `Boolean` is specified for the `enabled` field
- a value different from `null` or with a different type than `Array of String` is specified for the `disableOnAttributes` field
- a value different from `null` or with a different type than `Array of String` is specified for the `disableOnWords` field
- a value different from `null` or with a different type than `Integer` is specified for the `minWordSizeForTypos` object fields.
- only one of the fields `oneTypo` or `twoTypos` for the `minWordSizeForTypos` is specified and the value provided is invalid. (`Asynchronous`)
- both `oneTypo` and `twoTypos` fields are specified for the `minWordSizeForTypos` and the values provided are invalid. (`Synchronous`)

### Error Definition

HTTP Code: `400 Bad Request` when `Synchronous`

#### Variant: `enabled`, `disableOnAttributes`, `disableOnWords` properties are invalid regarding their expected format.

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_typo_tolerance",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_typo_tolerance"
}
```

#### Variant: `minWordSizeForTypos` object is invalid.

```json
{
    "message": "`minWordSizeForTypos` setting is invalid. `oneTypo` and `twoTypos` fields should be between `0` and `255`, and `twoTypos` should be greater or equals to `oneTypo` but found oneTypo: `:oneTypo` and twoTypos: `:twoTypos`.",
}
```

---

## invalid_settings_faceting

`Synchronous`

### Context

This error occurs when a value different from `null` or with a different type than `Integer` is specified for the `maxValuesPerFacet` field.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_faceting",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_faceting"
}
```

---

## invalid_settings_pagination

`Synchronous`

### Context

This error occurs when a value different from `null` or with a different type than `Integer` is specified for the `maxTotalHits` field.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_settings_pagination",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_settings_pagination"
}
```

---

## invalid_search_filter

`Synchronous`

### Context

This error occurs when:

- there is a syntax error in the `filter` parameter
- an attribute expressed in the filter is not defined in the `filterableAttributes` list
- a reserved keyword like `_geo`, `_geoDistance` and `_geoPoint` is used as a filter

### Error Definition

HTTP Code: `400 Bad Request`

#### Variant: Filtering on a non filterable attribute

```json
{
    "message": "Attribute `:attribute` is not filterable. Available filterable attributes are: `:filterableAttributes`.",
    "code": "invalid_search_filter",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_filter"
}
```

- `:filterableAttributes` contains the list of filterable attributes separated by a comma. `filterableAttribute1, filterableAttribute2, ...`

#### Variant: Filtering on a non filterable attribute when `filterableAttributes` is empty

```json
{
    "message": "Attribute `:attribute` is not filterable. This index does not have configured filterable attributes.",
    ...
}
```

#### Variant: Using `_geoDistance` as a filter expression

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a filter expression.",
    ...
}
```

#### Variant: Using `_geo` or `_geoPoint` as a filter expression

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a filter expression. Use the _geoRadius(latitude, longitude, distance) built-in rule to filter on _geo field coordinates.",
    ...
}
```

#### Variant: Invalid syntax for the `filter` parameter

```json
{
    "message": "Invalid syntax for the filter parameter: `:syntaxErrorHelper`.",
    ...
}
```

---

## invalid_search_sort

`Synchronous`

### Context

This error occurs when:

- there is a syntax error in the `sort` parameter
- an attribute expressed in the sort is not defined in the `sortableAttributes` list, sort at search time while the `sort` ranking rule is missing from the settings
- a reserved keyword like `_geo`, `_geoDistance` and `_geoRadius` is used as a sort expression

### Error Definition:

HTTP Code: `400 Bad Request`

#### Variant: Sorting on a non sortable attribute

```json
{
    "message": "Attribute `:attribute` is not sortable. Available sortable attributes are: `:sortableAttributes`.",
    "code": "invalid_search_sort",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_sort"
}
```

- `:sortableAttributes` contains the list of sortable attributes separated by a comma. `sortableAttribute1, sortableAttribute2, ...`

#### Variant: Sorting on a non sortable attribute when `sortableAttributes` is empty

```json
{
    "message": "Attribute `:attribute` is not sortable. This index does not have configured sortable attributes.",
    ...
}
```

#### Variant: Using `_geoDistance` as a sort expression

```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a sort expression.",
    ...
}
```

#### Variant: Using `_geo` or `_geoRadius` as a sort expression
```json
{
    "message": "`:reservedKeyword` is a reserved keyword and thus can't be used as a sort expression. Use the _geoPoint(latitude, longitude) built-in rule to sort on _geo field coordinates.",
    ...
}
```

#### Variant: Invalid syntax for the `sort`parameter

```json
{
    "message": "Invalid syntax for the sort parameter: `:syntaxErrorHelper`.",
    ...
}
```

#### Variant: Specifying `sort` at search time while the sort ranking rule isn't set in the ranking rules settings

```json
{
    "message": "The sort ranking rule must be specified in the ranking rules settings to use the sort parameter at search time.",
    ...
}
```

---

## invalid_search_q

`Synchronous`

### Context

This error occurs if a value with a different type than `String` or `null` for `q` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_q",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_q"
}
```

---

## invalid_search_offset

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `offset` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_offset",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_offset"
}
```

---

## invalid_search_limit

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `limit` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_limit",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_limit"
}
```

---

## invalid_search_page

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `page` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_page",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_page"
}
```

---

## invalid_search_hits_per_page

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `hitsPerPage` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_hits_per_page",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_hits_per_page"
}
```

---

## invalid_search_attributes_to_retrieve

`Synchronous`

### Context

This error occurs if a value with a different type than `Array of String`, `String` or `null` for `attributesToRetrieve` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_attributes_to_retrieve",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_attributes_to_retrieve"
}
```

---

## invalid_search_attributes_to_crop

`Synchronous`

### Context

This error occurs if a value with a different type than `Array[String]`, `String` or `null` for `attributesToCrop` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_attributes_to_crop",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_attributes_to_crop"
}
```

---

## invalid_search_crop_length

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `cropLength` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_crop_length",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_crop_length"
}
```

---

## invalid_search_attributes_to_highlight

`Synchronous`

### Context

This error occurs if a value with a different type than `Array[String]`, `String` or `null` for `attributesToHighlight` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_attributes_to_highlight",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_attributes_to_highlight"
}
```

---

## invalid_search_show_matches_position

`Synchronous`

### Context

This error occurs if a value with a different type than `Boolean` or `null` for `showMatchesPosition` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_show_matches_position",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_show_matches_position"
}
```

---

## invalid_search_facets

`Synchronous`

### Context

This error occurs when:

- A value with a different type than `Array of String`, `String` or `null` for `facets` is specified.
- A field not defined as a `filterableAttributes` for `facets` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

#### Variant: An type is given for `facets`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_facets",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_facets"
}
```

#### Variant: A given field for `facets` is not specified as a `filterableAttributes` settings

```json
{
    "message": "Invalid facet distribution, the fields `:fieldName` are not set as filterable.",
    ...
}
```

---

## invalid_search_highlight_pre_tag

`Synchronous`

### Context

This error occurs if a value with a different type than `String` for `highlightPreTag` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_highlight_pre_tag",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_highlight_pre_tag"
}
```

---

## invalid_search_highlight_post_tag

`Synchronous`

### Context

This error occurs if a value with a different type than `String` for `highlightPostTag` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_highlight_post_tag",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_highlight_post_tag"
}
```

---

## invalid_search_crop_marker

`Synchronous`

### Context

This error occurs if a value with a different type than `String` or `null` for `cropMarker` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_crop_marker",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_crop_marker"
}
```

---

## invalid_search_matching_strategy

`Synchronous`

### Context

This error occurs if a value with a different type than `String` and other than `last` or `all` as a value for `matchingStrategy` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_search_matching_strategy",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_search_matching_strategy"
}
```

---

## missing_facet_search_facet_name

`Synchronous`

### Context

This error occurs if `facetName` isn't specified when making a facet search call.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "missing_facet_search_facet_name",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_facet_search_facet_name"
}
```

---

## invalid_facet_search_facet_name

`Synchronous`

### Context

This errors occurs when the provided value for `facetName`:

- Is not a string
- Is not defined in the `filterableAttributes` index setting

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_facet_search_facet_name",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_facet_search_facet_name"
}
```

---

## invalid_facet_search_facet_query

`Synchronous`

### Context

This errors occurs when the provided value for `facetQuery`:

- Is not a string or null

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_facet_search_facet_query",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_facet_search_facet_query"
}
```

---

## invalid_document_geo_field

`Asynchronous`

### Context

These errors occurs when the `_geo` field of a document payload is not valid. Either the latitude / longitude is missing or is not a number.

### Error Definition

#### Variant: `_geo` field is not an object.

```json
{
    "message": "The `_geo` field in the document with the id: `:documentId` is not an object. Was expecting an object with the `_geo.lat` and `_geo.lng` fields but instead got `:field`.",
    "code": "invalid_document_geo_field",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_document_geo_field"
}
```

#### Variant: Missing `_geo.lat` **and** `_geo.lng` field.

```json
{
    "message": "Could not find latitude nor longitude in the document with the id: `:documentId`. Was expecting `_geo.lat` and `_geo.lng` fields.",
    ...
}
```

#### Variant: Missing `_geo.lat` **or** `_geo.lng` field.

```json
{
    "message": "Could not find :coord in the document with the id: `:documentId`. Was expecting a `:field` field.",
    ...
}
```

#### Variant: Coordinate can't be parsed.

```json
{
    "message": "Could not parse :coord in the document with the id: `:documentId`. Was expecting a finite number but instead got `:value`.",
    ...
}
```

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
    "message": "The provided payload reached the size limit. The maximum accepted payload size is :playloadSizeLimit.",
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

HTTP Code when `Synchronous`:
- if the index uid was specified as part of the URL, `404 Not Found`
- if the index uid was specified as part of the POST body, `400 Bad Request`

#### Variant: Multiples indexUids can't be found

```json
{
    "message": "Indexes `:indexUids` not found.",
    "code": "index_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#index_not_found"
}
```

- `:indexUids` values are separated by `,`.

#### Variant: An index can't be found

```json
{
    "message": "Index `:indexUid` not found.",
    ...
}
```

---

## missing_swap_indexes

`Synchronous`

### Context

This error happens when `indexes` is missing from a swap operation object.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`indexes` field is mandatory.",
    "code": "missing_swap_indexes",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_swap_indexes"
}
```

---

## invalid_swap_indexes

`Synchronous`

### Context

This error happens when:

- an `indexes` array not containing **exactly** 2 index uids for a swap operation object is specified in the payload
- An index name is invalid in the `indexes` array.

### Error Definition

#### Variant: `indexes` does not contains **exactly** 2 index uids

```json
{
    "message": "Two indexes must be given for each swap. The list `:indexesList` contains `:indexesNumber` indexes.",
    "code": "invalid_swap_indexes",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_swap_indexes"
}
```

#### Variant: `indexes` contains one index uid being invalid

```json
{
    "message": "`:uid` is not a valid index uid. Index uid can be an integer or a string containing only alphanumeric characters, hyphens (-) and underscores (_).",
    ...
}
```

---

## invalid_swap_duplicate_index_found

`Synchronous`

### Context

This error happens when the same indexUid is used twice in the `POST`- `swap-indexes` payload.

### Error Definition


#### Variant: A single indexUid is found twice in the payload

```json
{
    "message": "Indexes must be declared only once during a swap. `:indexUid` was specified several times.",
    "code": "invalid_swap_duplicate_index_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_swap_duplicate_index_found"
}
```

#### Variant: Several indexUids are found twice in the payload

```json
{
    "message": "Indexes must be declared only once during a swap. `:indexUids` were specified several times.",
    ...
}
```

- `:indexUids` values are separated by `,`.

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

---

## invalid_task_uids

`Synchronous`

### Context

This error occurs when the `uids` query parameter filter is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task uid `:uid` is invalid. It should only contains numeric characters separated by `,` character.",
    "code": "invalid_task_uids",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_uids"
}
```

---

## invalid_task_index_uids

### Context

This error occurs when the `indexUids` query parameter contains an invalid index uid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:uid` is not a valid index uid. Index uid can be an integer or a string containing only alphanumeric characters, hyphens (-) and underscores (_).",
    "code": "invalid_task_index_uids",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_index_uids"
}
```

---

## invalid_task_before_enqueued_at

`Synchronous`

### Context

This error occurs when the `beforeEnqueuedAt` query parameter filter is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task `beforeEnqueuedAt` `:value` is invalid. It should follow the RFC 3339 format. e.g. 'YYYY-MM-DD' or 'YYYY-MM-DD HH:MM:SS'.",
    "code": "invalid_task_before_enqueued_at",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_before_enqueued_at"
}
```

---

## invalid_task_after_enqueued_at

`Synchronous`

### Context

This error occurs when the `afterEnqueuedAt` query parameter filter is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task `afterEnqueuedAt` `:value` is invalid. It should follow the RFC 3339 format. e.g. 'YYYY-MM-DD' or 'YYYY-MM-DD HH:MM:SS'.",
    "code": "invalid_task_after_enqueued_at",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_after_enqueued_at"
}
```

---

## invalid_task_before_started_at

`Synchronous`

### Context

This error occurs when the `beforeStartedAt` query parameter filter is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task `beforeStartedAt` `:value` is invalid. It should follow the RFC 3339 format. e.g. 'YYYY-MM-DD' or 'YYYY-MM-DD HH:MM:SS'.",
    "code": "invalid_task_before_started_at",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_before_started_at"
}
```

---

## invalid_task_after_started_at

`Synchronous`

### Context

This error occurs when the `afterStartedAt` query parameter filter is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task `afterStartedAt` `:value` is invalid. It should follow the RFC 3339 format. e.g. 'YYYY-MM-DD' or 'YYYY-MM-DD HH:MM:SS'.",
    "code": "invalid_task_after_started_at",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_after_started_at"
}
```

---

## invalid_task_before_finished_at

`Synchronous`

### Context

This error occurs when the `beforeFinishedAt` query parameter filter is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task `beforeFinishedAt` `:value` is invalid. It should follow the RFC 3339 format. e.g. 'YYYY-MM-DD' or 'YYYY-MM-DD HH:MM:SS'.",
    "code": "invalid_task_before_finished_at",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_before_finished_at"
}
```

---

## invalid_task_after_finished_at

`Synchronous`

### Context

This error occurs when the `afterFinishedAt` query parameter filter is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task `afterFinishedAt` `:value` is invalid. It should follow the RFC 3339 format. e.g. 'YYYY-MM-DD' or 'YYYY-MM-DD HH:MM:SS'.",
    "code": "invalid_task_after_finished_at",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_after_finished_at"
}
```

---

## invalid_task_statuses

`Synchronous`

### Context

This error happens when the `status` query parameter filter is invalid.

#### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task status `:status` is invalid. Available task statuses are: `:taskStatuses`.",
    "code": "invalid_task_statuses",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#invalid_task_statuses"
}
```

---

## invalid_task_types

`Synchronous`

### Context

This error happens when the `types` query parameter filter is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task type `:type` is invalid. Available task types are: `:taskTypes`.",
    "code": "invalid_task_types",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#invalid_task_types"
}
```

---

## invalid_task_canceled_by

`Synchronous`

### Context

This error happens when the `canceledBy` query parameter filter is invalid.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Task canceledBy `:canceledBy` is invalid. It should only contains numeric characters separated by `,` character.",
    "code": "invalid_task_canceled_by",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#invalid_task_canceled_by"
}
```

---

## missing_task_filters

`Synchronous`

### Context

This error happens when no query parameters are given when a task cancelation or a task deletion request is sent.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Query parameters to filter the tasks to `:operation` are missing. Available query parameters are: `queryParametersNames`",
    "code": "missing_task_filters",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#missing_task_filters"
}
```

---

## invalid_task_limit

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `limit` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_task_limit",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_limit"
}
```

---

## invalid_task_from

`Synchronous`

### Context

This error occurs if a value with a different type than `Integer` for `from` is specified.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "`:deserr_helper`",
    "code": "invalid_task_from",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_task_from"
}
```

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

- `:contentTypeList` values are separated by a `,` char. e.g. `application/json`, `text/csv`.

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

- `:contentTypeList` values are separated by a `,` char. e.g. `application/json`, `text/csv`.

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

- `:payloadType` is e.g. `json`, `ndjson`, `csv`

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

- `:payloadType` is e.g. `json`, `ndjson`, `csv`

---

# internal type

## internal

`Asynchronous` / `Synchronous`

### Context

This error code occurs when an unknown and undetermined error has occurred at the server. This is a error that should not happen.

### Error Definition

HTTP Code: `500 Internal Server Error` when `Synchronous`

```json
{
    "message": "An internal error has occurred. `:reason`.",
    "code": "internal",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#internal"
}
```

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

---

## unretrievable_document

`Asynchronous` / `Synchronous`

### Context

This error occurs when a document cannot be found in the system due to an inconsistent state that can occur for several reasons.

### Error Definition

HTTP Code: `500 Internal Server Error` when `Synchronous`

```json
{
    "message": "The document `:documentId` is unretrievable due to `:reason`.",
    "code": "unretrievable_document",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#unretrievable_document"
}
```

---

## invalid_state

`Asynchronous` / `Synchronous`

### Context

This error occurs when the database is in an inconsistent state due to an uncontrolled internal error.

### Error Definition

HTTP Code: `500 Internal Server Error` when `Synchronous`

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

- `:databaseSizeLimit` is e.g. `100GiB`

---

## invalid_store_file

`Asynchronous` / `Synchronous`

### Context

This error occurs when the `data.ms` folder is in an inconsistent state. It can happen for various reasons. An .mdb file can be corrupted, the data.ms folder has been replaced by a file, etc.

### Error Definition

HTTP Code: `500 Internal Server Error` when `Synchronous`

```json
{
    "message": "The database file is in an invalid state.",
    "code": "invalid_store_file",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#invalid_store_file"
}
```

---

# system type

## no_space_left_on_device

`Asynchronous` / `Synchronous`

### Context

This error occurs when the host system partition has reached its maximum capacity and no longer accepts writes.
It can also happens when the task queue reaches its limit of ~10GiB of tasks (~10M tasks).

### Error Definition

HTTP Code: `500 Internal Server Error` when `Synchronous`

```json
{
    "message": "`:kernelMessage`",
    "code": "no_space_left_on_device",
    "type": "system",
    "link": "https://docs.meilisearch.com/errors#no_space_left_on_device"
}
```

In the case of the task queue being full the HTTP Code returned is `422 Unprocessable Entity` when `Synchronous`.

```json
{
    "message": "Meilisearch cannot receive write operations because the limit of the task database has been reached. Please delete tasks to continue performing write operations.",
    "code": "no_space_left_on_device",
    "type": "system",
    "link": "https://docs.meilisearch.com/errors#no_space_left_on_device"
}
```

---

## too_many_open_files

`Asynchronous` / `Synchronous` when `Synchronous`

### Context

This error occurs when the host system can't open more files.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "`:kernelMessage`",
    "code": "too_many_open_files",
    "type": "system",
    "link": "https://docs.meilisearch.com/errors#too_many_open_files"
}
```

---

## io_error

`Asynchronous` / `Synchronous` when `Synchronous`

### Context

This error generally occurs when the host system have no space left on device or when the database doesn't have read or write right.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "`:kernelMessage`. This error generally happens when you have no space left on device or when your database doesn't have read or write right.",
    "code": "io_error",
    "type": "system",
    "link": "https://docs.meilisearch.com/errors#io_error"
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

## missing_master_key

`Synchronous`

### Context

For some specific protected routes (i.e. `/keys`) the master key must be defined before accessing it. This error indicates to the user that he must first define a master key when launching Meilisearch.

### Error Definition

HTTP Code: `401 Forbidden`

```json
{
    "message": "Meilisearch is running without a master key. To access this API endpoint, you must have set a master key at launch.",
    "code": "missing_master_key",
    "type": "auth",
    "link": "https://docs.meilisearch.com/errors#missing_master_key"
}
```

---

## invalid_document_csv_delimiter

`Synchronous`

### Context

The csv delimiter must be exactly one char long, and this char must be an ASCII character.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Invalid value in parameter `csvDelimiter`: expected a string of one character, but found the following string of 5 characters: `doggo`",
    "code": "invalid_document_csv_delimiter",
    "type": "invalid_request"
    "link": "https://docs.meilisearch.com/errors#invalid_document_csv_delimiter",
}```

---

## 2. Technical details
n/a

## 3. Future Possibilities
n/a
