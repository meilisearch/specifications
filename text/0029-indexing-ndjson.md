- Title: Indexing NDJSON
- Start Date: 2021-04-12
- Specification PR: [PR-#29](https://github.com/meilisearch/specifications/pull/29)
- Discovery Issue: n/a

# Indexing NDJSON

## 1. Functional Specification

### I. Summary

To index documents, the body of the add documents request has to match a specific format. That specific format is then parsed and tokenized inside MeiliSearch. After which, the documents added are in the pool of searchable and returnable documents.

An [NDJSON](http://ndjson.org/) data format is easier to use than a CSV format because it propose a convenient format for storing structured data.

#### Summary Key Points

- `application/x-ndjson` Content-Type header is now supported.
- The error cases have been strengthened and completed. See Errors part.

### II. Motivation

Currently, the engine only accepts JSON format as a data source. We want to give users the possibility of another simple data format to use. Thus, give them more versatility at the data source choices for the indexing step.

Writing performance is also a motivation since JSON Lines data parsing is less CPU and memory-intensive than parsing standard JSON. When new lines represent separate entries it makes the NDJSON data streamable, thus, more suited for indexing a consequent data set.

While we give the ability to Meilisearch to ingest CSV data for indexing in this [specification](https://github.com/meilisearch/specifications/pull/28), we are aware of the limitations of CSV so we also want to provide a format that is easy to validate. Handling the validity of a CSV can be frustrating and difficult. Only strings can be managed within a CSV. In addition, there is no official specification except [RFC 4180](https://tools.ietf.org/html/rfc4180) which is not sufficient for all data scheme.

Representing nested structures in a JSON object is easy and convenient.

### III. Explanation

Newline-delimited JSON (`ndjson`), line-delimited JSON (`ldjson`), JSON lines (`jsonl`) are three terms expressing the same formats primarily intended for JSON streaming.

As of now, we will use `ndjson` in the next parts to refer to a data format that represents JSON entries separated by a new line character.

- Each entries will represent a document for MeiliSearch.
- Each entries should be a valid JSON object.
- The data should be encoded in UTF-8.

#### Example of a valid NJSON

Given the NDJSON payload
'''
{"id":1, "label": "t-shirt", "price": 4.99, "colors": ["red", "green", "blue"]}
{"id":499, "label": "hoodie", "price": 19.99, "colors": ["purple"]}
'''
the search result should be displayed as
```json
{
  "hits": [
    {
      "id": 1,
      "label": "t-shirt",
      "price": 4.99,
      "colors": [
          "red",
          "green",
          "blue"
      ],
    },
    {
      "id": 499,
      "label": "hoodie",
      "price": 19.99,
      "colors": [
          "purple"
      ],
    }
  ],
  ...
}
```

#### API Endpoints

> Each API endpoints mentioned above will now require a `application/x-ndjson` as `Content-Type` header to be processed as NDJSON data.

**As a developer, I want to upload a NDJSON payload of documents so that end-user can search them**

**POST documents** `/indexes/:indexUid/documents`

```bash
curl \
  -X POST 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: application/x-ndjson' \
  --data-binary '
    {"id":1, "label": "t-shirt", "price": 4.99, "colors": ["red", "green", "blue"]}\n
    {"id":499, "label": "hoodie", "price": 19.99, "colors": ["purple"]}
  '
```
> 202 Accepted - Response

**PUT documents** `/indexes/:indexUid/documents`

```bash
curl \
  -X PUT 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: application/x-ndjson' \
  --data-binary '
    {"id":1, "label": "t-shirt", "price": 4.99, "colors": ["red", "green", "blue"]}\n
    {"id":499, "label": "hoodie", "price": 19.99, "colors": ["purple"]}
  '
```
> 202 Accepted - Response

##### Errors

- 🔴 Omitted `Content-Type` header will lead to a 415 Unsupported Media Type - **missing_content_type** error code.
- 🔴 Sending an empty `Content-Type` will lead to a 415 Unsupported Media Type - **invalid_content_type** error code.
- 🔴 Sending a different `Content-Type` than `application/json`, `application/x-ndjson` or `text/csv` will lead to 415 Unsupported Media Type  **invalid_content_type** error code.
- 🔴 Sending an empty payload will lead to a 400 Bad Request - **missing_payload** error code.
- 🔴 Sending a different payload type than the `Content-Type` header should return a 400 Bad Request - **malformed_payload** error code.
- 🔴 Sending a payload excessing the limit will lead to a 413 Payload Too Large - **payload_too_large** error code.
- 🔴 Sending an invalid ndjson format will lead to a 400 bad_request - **malformed_payload** error code.

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
    "message": "The :payloadType payload provided is malformed. :syntaxErrorHelper."
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

- Provide an interface in the future dashboard to upload NDJSON data into an index.
- Set a payload limit directly related to the type of data format. Currently, the payload size is equivalent to [JSON payload size](https://docs.meilisearch.com/reference/features/configuration.html#payload-limit-size). Metrics on feature usage and configuration update should help to choose a better suited value for this type of data.
