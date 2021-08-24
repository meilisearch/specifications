- Title: Error Format and Definitions
- Start Date: 2021-08-13
- Specification PR: [#61](https://github.com/meilisearch/specifications/pull/61)
- Discovery Issue: [#44](https://github.com/meilisearch/product/issues/44#issuecomment-896121211)

# Error Format

## 1. Functional Specification

### I. Summary

The error format is updated to no longer contain unnecessary `error` prefix terms in the names of these attributes. e.g. `errorType`. The context is clear enough to understand that this is an `error` object.

This specification also serves as a reference point for the complete list of API errors that the user may encounter.

#### Summary Key Points

- `error` is removed from the attributes name and `type`values.
- An exhaustive list of API errors is defined. See Errors List part.

### II. Motivation

The main motivation is to stabilize the current `error` resource to a version that conforms to our API convention and thus allows future evolutions on a more solid base. This specification avoids adding unnecessary information in the error object's attribute names.

The second motivation is to describe in an exhaustive way all the errors that the user may encounter during his use of the API.

### III. Explanation

#### Error Format

##### Attributes

| Field name | type   | Description                                                              |
|------------|--------|--------------------------------------------------------------------------|
| message    | string |  A human-readable message providing context and details about the error. |
| code       | string |  A short string indicating the error code reported.                      |
| type       | string |  The type of error returned. `invalid_request`, `internal`, `auth`.       |
| link       | string |  An URL to the related error-page details for further information.       |

##### Json Response Example

e.g. 401 Unauthorized Response example

```
{
    "message": "authorization header is missing",
    "code": "missing_authorization_header",
    "type": "auth",
    "link": "https://docs.meilisearch.com/errors#missing_authorization_header"
}
```

##### type enum

| type            | description                                                                                       |
|-----------------|---------------------------------------------------------------------------------------------------|
| invalid_request | This type of error is usually due to a user input error. It is accompanied by an HTTP code `4xx`. |
| internal        | Usually this type of error is returned because the search engine can't perform the operation due to machine or configuration constraints. It can be due to limits being reached such as the size of the disk, the size limit of an index etc.. It can also be an unexpected error. It is accompanied by an HTTP code `5xx`.  |
| auth   | This type of error is returned when it comes to authentication and authorization. It is accompanied by an HTTP code `4xx`. |

---

#### Error list

Following this format here is the exhaustive list of errors that can be returned by MeiliSearch to an API consumer. This list is updated as MeiliSearch evolves.

💡 Errors returned asynchronously in a `task` object, does not include a definition of the http code.

# invalid_request type

## bad_request

### Context

This error code is generic. It should not be used. Instead, a clear and precise error code should be determined.

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

---

## index_already_exists

### Context

This error happens when a user tries to create an index that already exists.

### Error Definition

HTTP Code: `409 Conflict`

```json
{
    "message": "Index :uid already exists.",
    "code": "index_already_exists",
    "type": "invalid_request",
    "link":"https://docs.meilisearch.com/errors#index_already_exists"
}
```

- The `:uid` is inferred when the message is generated.

---

## invalid_index_uid

### Context

This error happens when a user tries to create an index with an invalid uid format.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "Index :uid is not a valid uid. Index uid can be of type integer or string only composed of alphanumeric characters, hyphens (-) and underscores (_).",
    "code": "invalid_index_uid",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_index_uid"
}
```

- The `:uid` is inferred when the message is generated.

---

## index_primary_key_already_exists

### Context

This error happens when a user try to update an index primary key while the index already has one primary key.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "A primary key already exists for the index :uid.",
    "code": "index_primary_key_already_exists",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#primary_key_already_exists"
}
```

---

## missing_document_primary_key

### Context

This error occurs when the engine does not find an identifier to infer in the documents to be inserted during the first ingestion.

### Error Definition

```json
{
    "message": "",
    "code": "missing_document_primary_key",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_document_primary_key"
}
```

---

## missing_document_id

### Context

This error occurs when the engine does not find the primary key previously defined for the index in the document payload.

### Error Definition

```json
{
    "message": "",
    "code": "missing_document_id",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_document_id"
}
```

---

## document_fields_limit_reached

### Context

The maximum number of fields for a document is `65,535`. When this number is exceeded, this error is returned. This error is returned within a `task` for a `documentsAddition` or `documentsPartial` operation.

### Error Definition

```json
{
    "message": "A document cannot contain more than 65,535 fields.",
    "code": "document_fields_limit",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#documents_fields_limit"
}
```

---

## invalid_ranking_rule

### Context

This error occurs when the user specifies a non-existent ranking rule or a malformed custom ranking rule in the settings payload.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": ":rankingRule ranking rule is invalid. Valid ranking rules are Words, Typo, Sort, Proximity, Attribute, Exactness and custom ranking rules.",
    "code": "invalid_ranking_rule",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_ranking_rule"
}
```
- The `:rankingRule` is inferred when the message is generated. It could be a misspelled ranking rule like `Wards` instead of `Words` or a custom ranking rule expressed with a `price::asc` syntax error.

---

## invalid_filter

### Context

This error occurs at search time when there is a syntax error in the `filter` parameter or when an attribute expressed in the filter is not defined in the `filterableAttributes` list.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": ":syntaxErrorHelper. Attribute :attribute is not filterable. Available filterable attributes are ..., ..., ...",
    "code": "invalid_filter",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_filter"
}
```

- The `:syntaxErrorHelper` is inferred when a syntax error is encountered.
- The `:attribute` is inferred when the message is generated.

---

## invalid_sort

### Context

This error occurs at search time when there is a syntax error in the `sort` parameter or when an attribute expressed in the sort is not defined in the `sortableAttributes` list.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": ":syntaxErrorHelper. Attribute :attribute is not sortable. Available sortable attributes are ..., ..., ...",
    "code": "invalid_sort",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_sort"
}
```

- The `:syntaxErrorHelper` is inferred when a syntax error is encountered.
- The `:attribute` is inferred when the message is generated.

---

## invalid_geo_field

### Context

This error occurs when the `_geo` field of a document payload is not valid.

### Error Definition

```json
{
    "message": "The _geo field is invalid. :syntaxErrorHelper.",
    "code": "invalid_geo_field",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_geo_field"
}

```

- The `:syntaxErrorHelper` is inferred when a syntax error is encountered.

---

## payload_too_large

### Context

This error occurs when the size of the payload sent exceeds the limit set by the server. The user can correct this error by reducing the payload size or increasing the limit with [this configuration variable](https://docs.meilisearch.com/reference/features/configuration.html#payload-limit-size)

### Error Definition

HTTP Code: `413 Payload Too Large`

```json
{
    "message": "The payload cannot exceed :payloadSizeLimit.",
    "code": "payload_too_large",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#payload_too_large"
}
```

- The `:payloadSizeLimit` is inferred when the message is generated. e.g. `100Mb` by default.

---

## not_found

### Context

This error code is generic. It should not be used. Instead, a clear and precise error code should be determined.

---

## index_not_found

### Context

This error happens when a requested index can't be found.

### Error Definition

HTTP Code: `404 Not Found`

```json
{
    "message": "Index :indexUid not found.",
    "code": "index_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#index_not_found"
}
```

- The `:indexUid` is inferred when the message is generated.

---

## document_not_found

## Context

This error happens when a requested document can't be found.

## Error Definition

HTTP Code: `404 Not Found`

```json
{
    "message": "Document :documentId not found.",
    "code": "document_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#document_not_found"
}
```

- The `:documentId` is inferred when the message is generated.

---

## dump_not_found

### Context

This error happens when a requested dump can't be found.

### Error Definition

HTTP Code: `404 Not Found`

```json
{
    "message": "Dump :dumpId not found.",
    "code": "dump_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#dump_not_found"
}
```

- The `:dumpId` is inferred when the message is generated.

---

## task_not_found

#### Context

This error happens when a requested task can't be found.

#### Error Definition

HTTP Code: `404 Not Found`

```json
{
    "message": "Task :taskUid not found.",
    "code": "task_not_found",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#task_not_found"
}
```

- The `:taskUid` is inferred when the message is generated.

---

## dump_already_processing

### Context

This error occurs when the user tries to launch the creation of a new dump while a creation is already being processed.

### Error Definition

Http Code: `409 Conflict`

```json
{
    "message": "A dump is already processing. You must wait until the current dump being processed is finished before requesting another dump creation.",
    "code": "dump_already_processing",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#dump_already_processing"
}
```

---

## unsupported_media_type

### Context

This error code is generic. It should not be used. Instead, a clear and precise error code should be determined.

---

## invalid_content_type

### Context

This error occurs when the provided content-type is not handled by the API method.

### Error Definition

HTTP Code: `415 Unsupported Media Type`

```json
{
    "message": "The Content-Type :contentType is invalid. Accepted values for Content-Type are: :contentTypeList",
    "code": "invalid_content_type",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_content_type"
}
```

- The `:contentTypeList` is inferred when the message is generated. The values are separated by a `,` char. e.g. `application/json`, `text/csv`.

---

## missing_content_type

### Context

This error occurs when the Content-Type header is missing.

### Error Definition

HTTP Code: `415 Unsupported Media Type`

```json
{
    "message": "A Content-Type header is missing. Accepted values for Content-Type are: :contentTypeList",
    "code": "missing_content_type",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_content_type"
}
```

- The `:contentTypeList` is inferred when the message is generated. The values are separated by a `,` char. e.g. `application/json`, `text/csv`.

---

## missing_payload

### Context

This error occurs when the client does not provide a mandatory payload to the request.

### Error Definition

HTTP Code: `400 Bad Request`

```json
{
    "message": "A :payloadType payload is missing.",
    "code": "missing_payload",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_payload"
}
```

- The `:payloadType` is inferred when the message is generated. e.g. `json`, `ndjson`, `csv`

---

## malformed_payload

### Context

This error occurs when the format sent in the payload is malformed. The payload contains a syntax error.

### Error Definition

HTTP Code: `400 Bad Request`

```json
    "message": ":syntaxErrorHelper. The :payloadType payload provided is malformed.",
    "code": "malformed_payload",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#malformed_payload"
```

- The `:payloadType` is inferred when the message is generated. e.g. `json`, `ndjson`, `csv`
- The `:syntaxErrorHelper` is inferred when the message is generated.

---

# internal type

## internal

### Context

This error code occurs when an unknown and undetermined error has occurred at the server. This is a error that should not happen.

### Error Definition

HTTP Code: `500 Internal Server Error``

```json
{
    "message": "An internal error has occurred. :reason.",
    "code": "internal",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#internal"
}
```

- The `:reason` is inferred when the message is generated. e.g. `Internal: decoding failed`

---

## index_creation_failed

### Context

This error occurs when an index creation could not be completed for various reasons.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "The creation of the :uid index has failed due to :reason",
    "code": "index_creation_failed",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#index_creation_failed"
}
```

- The `:reason` is inferred when the message is generated. e.g. `Write Permission denied`

---

## index_not_accessible

### Context

This error occurs when an index can be accessed for various reasons.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "The index :uid can't be accessed due to :reason",
    "code": "index_not_accessible",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#index_not_accessible"
}
```

- The `:reason` is inferred when the message is generated. e.g. `Read Permission denied`

---

## unretrievable_document

### Context

This error occurs when a document cannot be found in the system due to an inconsistent state that can occur for several reasons.

### Error Definition

HTTP Code: `500 Internal Server Error`

```json
{
    "message": "The document :documentId is unretrievable due to :reason.",
    "code": "unretrievable_document",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#unretrievable_document"
}
```

- The `documentId` is inferred when the message is generated.
- The `:reason` is inferred when the message is generated.

---

## invalid_state

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

### Context

This error occurs during the dump creation process. The dump creation was interrupted for various reasons.

💡 Errors returned asynchronously in a `dump` object, does not include a definition of the http code.

### Error Definition

```json
{
    "message": "The creation of the dump :dumpId failed due to :reason.",
    "code": "dump_process_failed",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#dump_process_failed"
}
```

- The `:reason` is inferred when the message is generated.

---

## database_size_limit_reached

### Context

This error occurs when the user tries to add documents and the maximum size of the database reaches the limit. The user can correct this error by increasing the database size limit.

### Error Definition

HTTP code `500 Internal Server Error`

```json
{
    "message": "The maximum size of :databaseSizeLimit for the database has been reached.",
    "code": "database_size_limit_reached",
    "type": "internal",
    "link": "https://docs.meilisearch.com/errors#database_size_limit_reached"
}
```

- The `:databaseSizeLimit` is inferred when the message is generated. e.g. `100GiB`

---

## no_space_left_on_device

### Context

This error occurs when the host system partition has reached its maximum capacity and is no longer accepting writes.

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

### Context

This error occurs when the data.ms folder is in an inconsistent state. It can happen for various reasons. An .mdb file can be corrupted, the data.ms folder has been replaced by a file...

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

### Context

This error occurs when the route is protected and the `X-MEILI-API-KEY` header is not provided.

### Error Definition

HTTP Code: `401 Unauthorized`

```json
{
    "message": "The X-MEILI-API-KEY header is missing.",
    "code": "missing_authorization_header",
    "type": "auth",
    "link": "https://docs.meilisearch.com/errors#missing_authorization_header"
}
```

---

## invalid_api_key

### Context

This error occurs when the route is protected and the value of the `X-MEILI-API-KEY` header does not allow access to the resource.

### Error Definition

HTTP Code: `403 Forbidden`

```json
{
    "message": "The provided API Key is invalid.",
    "code": "invalid_api_key",
    "type": "auth",
    "link": "https://docs.meilisearch.com/errors#invalid_api_key"
}
```

---

## 2. Technical details
N/A

## 3. Future Possibilities

- Add a `parameter` attribute in the `error` object to indicate which parameter of the request caused an error.
- Add more specific error types. For example, explode internal errors into several distinct types.