- Title: Indexing JSON
- Start Date: 2022-08-16
- Specification PR: [PR-#167](https://github.com/meilisearch/specifications/pull/167)
- Discovery Issue: n/a

# Indexing JSON

## 1. Functional Specification

### I. Summary

To index documents, the body of the add documents request has to match a specific format. That specific format is then parsed and tokenized inside MeiliSearch. After which, the documents added are in the pool of searchable and returnable documents.

A [JSON](http://json.org/) data format is easier to use than a CSV format because it propose a convenient format for storing structured data.
When indexing multiple documents you should prefer using [ndjson](0028-indexing-ndjson.md). It's faster and more concise.

#### Summary Key Points

- `application/json` Content-Type header is now supported.
- The error cases have been strengthened and completed. See Errors part.

### II. Explanation

- Meilisearch accept an array of documents
- Or a single document.
- The data should be encoded in UTF-8.

#### Example of a valid JSON

To send the following documents;
'''
{"id":1, "label": "t-shirt", "colors": ["red", "green", "blue"]}
{"id":499, "label": "hoodie", "colors": ["purple"]}
'''

You can send them in an array like that;
```json
[
  {"id":1, "label": "t-shirt", "colors": ["red", "green", "blue"]},
  {"id":499, "label": "hoodie", "colors": ["purple"]}
]
```

You also have the possibility to format the json however you like. Here is another way to send the two same documents;
```json
[
  {
    "id":1,
    "label": "t-shirt",
    "colors": ["red", "green", "blue"]
  },
  {
    "id":499, "label": "hoodie", "colors": ["purple"]
  }
]
```

Or you could two requests to send the documents directly;
```json
{
  "id":1,
  "label": "t-shirt",
  "colors": ["red", "green", "blue"]
}
```
And
```json
{
  "id":499, "label": "hoodie", "colors": ["purple"]
}
```
For example.

/!\ Be cautious though, if you send the two documents in a single request like that for example;

```json
{
  "id":1,
  "label": "t-shirt",
  "colors": ["red", "green", "blue"]
}
{
  "id":499, "label": "hoodie", "colors": ["purple"]
}
```
Meilisearch will only index the first document and **won't** throw an error.


#### API Endpoints

> Each API endpoints mentioned above will now require a `application/json` as `Content-Type` header to be processed as JSON data.

**As a developer, I want to upload a JSON payload of documents so that end-user can search them**

**POST documents** `/indexes/:indexUid/documents`

```bash
curl \
  -X POST 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: application/json' \
  --data-binary '
    [{"id":1, "label": "t-shirt", "price": 4.99, "colors": ["red", "green", "blue"]},{"id":499, "label": "hoodie", "price": 19.99, "colors": ["purple"]}]'
```
> 202 Accepted - Response

**PUT documents** `/indexes/:indexUid/documents`

```bash
curl \
  -X PUT 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: application/json' \
  --data-binary '
    [{"id":1, "label": "t-shirt", "price": 4.99, "colors": ["red", "green", "blue"]},{"id":499, "label": "hoodie", "price": 19.99, "colors": ["purple"]}]'
```
> 202 Accepted - Response

##### Errors

- 🔴 Omitted `Content-Type` header will lead to a 415 Unsupported Media Type - **missing_content_type** error code.
- 🔴 Sending an empty `Content-Type` will lead to a 415 Unsupported Media Type - **invalid_content_type** error code.
- 🔴 Sending a different `Content-Type` than `application/json`, `application/x-ndjson` or `text/csv` will lead to 415 Unsupported Media Type  **invalid_content_type** error code.
- 🔴 Sending an empty payload will lead to a 400 Bad Request - **missing_payload** error code.
- 🔴 Sending a different payload type than the `Content-Type` header should return a 400 Bad Request - **malformed_payload** error code.
- 🔴 Sending a payload excessing the limit will lead to a 413 Payload Too Large - **payload_too_large** error code.
- 🔴 Sending an invalid json format will lead to a 400 bad_request - **malformed_payload** error code.

##### Errors Definition

## missing_content_type

### Context

This error occurs when the Content-Type header is missing.

### Error Definition

HTTP Code: `415 Unsupported Media Type`

```json
{
    "message": "A Content-Type header is missing. Accepted values for the Content-Type header are: :contentTypeList",
    "code": "missing_content_type",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#missing_content_type"
}
```

- The `:contentTypeList` is inferred when the message is generated. The values are separated by a `,` char. e.g. `application/json`, `text/csv`.

---

## invalid_content_type

### Context

This error occurs when the provided content-type is not handled by the API method.

### Error Definition

HTTP Code: `415 Unsupported Media Type`

```json
{
    "message": "The Content-Type :contentType is invalid. Accepted values for the Content-Type header are: :contentTypeList",
    "code": "invalid_content_type",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#invalid_content_type"
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
    "message": "The :payloadType payload provided is malformed. :syntaxErrorHelper.",
    "code": "malformed_payload",
    "type": "invalid_request",
    "link": "https://docs.meilisearch.com/errors#malformed_payload"
```

- The `:payloadType` is inferred when the message is generated. e.g. `json`, `ndjson`, `csv`
- The `:syntaxErrorHelper` is inferred when the message is generated.

---

## 2. Technical details
n/a

## 3. Future possibilities

- Throw an error when there is multiple documents in the payload but not in an array.
