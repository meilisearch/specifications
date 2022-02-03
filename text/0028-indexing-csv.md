- Title: Indexing CSV
- Start Date: 2021-04-9
- Specification PR: [PR-#28](https://github.com/Meilisearch/specifications/pull/28)
- Discovery Issue: n/a

# Indexing CSV

## 1. Functional Specification

### I. Summary

To index documents, the body of the add documents request has to match a specific format. That specific format is then parsed and tokenized inside Meilisearch. After which, the documents added are in the pool of searchable and returnable documents.

A [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) data format is broadly used to store and exchange data in a simple format.

Also, in order to boost write performance CSV data format is more suited than JSON for consequent datasets, as keys are not duplicated for every document.

#### Summary Key Points

- The header of the csv payload allows to name the attributes and type them.
- `text/csv` Content-Type header is now supported.
- The error cases have been strengthened and completed. See Errors part.

### II. Motivation

We want to provide our users with an always improved usage experience. Currently, the engine only accepts JSON format as a data source. We want to give users the possibility of another simple data format, well known, to use. Thus, give them more versatility at the data source choices for the indexing (add and update) step.

Since most SQL engines or SQL clients can easily dump data as CSV, it will facilitate Meilisearch adoption by extending the indexing step on a wider range of customer cases than before.

Writing performance is also considered as a motivation since CSV parsing is less CPU and memory intensive than parsing Json due to the streamable capability of the CSV format.

### III.Explanation

#### CSV Formatting Rules

While there's [RFC 4180](https://tools.ietf.org/html/rfc4180) as a try to add a specification for CSV format, we will find a lot of variations from that. Meilisearch features capabilities requires CSV data to be formatted the proper way to be parsable by the engine.

- CSV data format needs to contain a first line representing the list of attributes with the optionally chosen type separated from the attribute name by `:` character. The type is case insensitive.

> An attribute can be specificed with two types: `string` or `number`. A missing type will be interpreted as a `string` by default.
>
> Valid headline example: "id:number","title:string","author","price:number"

- The following CSV lines will represent a document for Meilisearch.
- A `,` character must separate each cell.
- A CSV value should be enclosed in double-quotes when it contains a comma character or a newline to escape it.
- Using double-quotes to enclose fields, then a double-quote appearing inside a field must be escaped by preceding it with another double quote as mentioned in [RFC 4180](https://tools.ietf.org/html/rfc4180).
- Float value should be written with a `.` character, like `3.14`.
- CSV text should be encoded in UTF8.
- The format can't handle array cell values. We are providing `nd-json` format to deal with theses types of attribute in a easier way.

##### `null` value

- If a field is of type `string`, then an empty cell is considered as a `null` value (e.g. `,,`), anything else is turned into a string value (e.g. `, ,` is a single whitespace string)
- If a field is of type `number`, when the trimmed field is empty, it's considered as a `null` value (e.g. `,,` `, ,`); otherwise Meilisearch try to parse the number.

##### Example with a comma inside a cell

Given the CSV payload
```
"id:number","label","price:number","colors","description"
"1","t-shirt","4.99","red","Thus, you will rock at summer time."
```
the search result should be displayed as
```json
{
  "hits": [
    {
      "id": 1,
      "label": "t-shirt",
      "price": 4.99,
      "colors": "red",
      "description": "Hey, you will rock at summer time."
    }
  ],
  ...
}
```

##### Example with a double quote inside a cell

Given the CSV payload
```
"id:number","label","price","colors","description"
"1","t-shirt","4.99","red","Hey, you will ""rock"" at summer time."
```
the search result should be displayed as
```json
{
  "hits": [
    {
      "id": 1,
      "label": "t-shirt",
      "price": "4.99",
      "colors": "red",
      "description": "Hey, you will rock at summer time.",
    }
  ],
  ...
}
```

> Note that the price attribute was not typed as a number. By default, Meilisearch type it as a string.

##### Example with an empty cell

Given the CSV payload
```
id:number,label,price:number,colors
1,t-shirt,,red
```
the search result should be displayed as
```json
{
  "hits": [
    {
      "id": 1,
      "label": "t-shirt",
      "price": null,
      "colors": "red"
    }
  ],
  ...
}
```

#### API Endpoints

> Each API endpoints mentioned above will now require a `text/csv` as `Content-Type` header to be processed as CSV data.

**As a developer, I want to upload a CSV payload of documents so that end-user can search them**

**POST documents** `/indexes/:indexUid/documents`

```bash
curl \
  -X POST 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: text/csv' \
  --data--binary '
    "id","label","price:number","colors","description"\n
    "1","hoodie","19.99","purple","Hey, you will rock at summer time."
  '
```
> 202 Accepted - Response

**PUT documents** `/indexes/:indexUid/documents`

```bash
curl \
  -X PUT 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: text/csv' \
  --data-binary '
    "id","label","price:number","colors","description"\n
    "1","hoodie","19.99","purple","Hey, you will rock at summer time."
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
- 🔴 Sending an invalid CSV format will lead to a 400 bad_request - **malformed_payload** error code.
- 🔴 Sending a CSV header that does not conform to the specification will lead to a 400 bad_request - **malformed_payload** error code.

##### Errors Definition

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

## 2. Technical details
n/a

## 3. Future possibilities

- Provide an interface in the future dashboard to upload CSV data into an index and optionally provide the headers types.
- Set a payload limit directly related to the type of data format. Currently, the payload size is equivalent to [JSON payload size](https://docs.meilisearch.com/reference/features/configuration.html#payload-limit-size). Metrics on feature usage and configuration update should help to choose a better suited value for this type of data format.
- Accepts more common CSV separators
