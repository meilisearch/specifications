- Title: Indexing CSV
- Start Date: 2021-04-9
- Specification PR: [PR-#28](https://github.com/meilisearch/specifications/pull/28)
- MeiliSearch Tracking-Issues:

# Indexing CSV

## 1. Feature Description and Interaction

### I. Summary

To index documents, the body of the add documents request has to match a specific format. That specific format is then parsed and tokenized inside MeiliSearch. After which, the documents added are in the pool of searchable and returnable documents.

A [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) data format is broadly used to store and exchange data in a simple format.

Also, in order to boost write performance CSV data format is more suited than JSON for consequent datasets has keys are not duplicated for every document.

### II. Motivation

We want to provide our users with an always improved usage experience. Currently, the engine only accepts JSON format as a data source. We want to give users the possibility of another simple data format, well known, to use. Thus, give them more versatility at the data source choices for the indexing (add and update) step.

Since most SQL engines or SQL clients can easily dump data as CSV, it will facilitate MeiliSearch adoption by extending the indexing step on a wider range of customer cases than before.

Writing performance is also considered as a motivation since CSV parsing is less CPU and memory intensive than parsing Json due to the streamable capability of the CSV format.

### III. Additional Materials
N/A

### IV.Explanation

#### Csv Formatting Rules

While there's [RFC 4180](https://tools.ietf.org/html/rfc4180) as a try to add a specification for csv format, we will find a lot of variations from that. MeiliSearch features capabilities requires csv data to be formatted the proper way to be parsable by the engine.

- CSV data format needs to contain a first line representing the list of attributes with the optionally chosen type separated from the attribute name by `:` character. The type is case insensitive.

> An attribute can be specificed with two types: `string` or `number`. A missing type will be interpreted as a `string` by default.
>
> Valid headline example: "id:number","title:string","author","price:number"

- The following CSV lines will represent a document for MeiliSearch.
- A CSV value should be enclosed in double-quotes when it contains a comma character to escape it.
- Using double-quotes to enclose fields, then a double-quote appearing inside a field must be escaped by preceding it with another double quote as mentioned in [RFC 4180](https://tools.ietf.org/html/rfc4180).
- Float value should be written with a `.` character, like `3.14`.
- CSV text should be encoded in UTF8.
- The format can't handle array cell values. We are providing `nd-json` format to deal with theses types of attribute in a easier way.

##### Example with a comma inside a cell

Given the csv payload
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

Given the csv payload
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

> Note that the price attribute was not typed as a number. By default, MeiliSearch type it as a string.

#### API Endpoints

> Each API endpoints mentioned above will now require a `text/csv` as `Content-Type` header to process CSV data.
> ⚠ A missing Content-Type will be interpreted as `application/json` since it's the current behavior. Giving an `application/json` Content-Type leads to the same behavior.

#### Add or Replace Documents [📎](https://docs.meilisearch.com/reference/api/documents.html#add-or-replace-documents)

```curl
curl \
  -X POST 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: text/csv' \
  --binary-data '
    "id","label","price:number","colors","description"\n
    "1","hoodie","19.99","purple","Hey, you will rock at summer time."
  '
```
> Response code: 202 Accepted

##### Error codes

> - Sending a different payload than the `Content-Type` header should return a `400 bad_request` error.
> - Too large payload according to the limit should return a `413 payload_too_large` error
> - Wrong encoding should return a `400 bad_request` error
> - Invalid CSV data should return a `400 bad_request` error

### Add or Update Documents [📎](https://docs.meilisearch.com/reference/api/documents.html#add-or-update-documents)

```curl
curl \
  -X PUT 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: text/csv' \
  --binary-data '
    "id","label","price:number","colors","description"\n
    "1","hoodie","19.99","purple","Hey, you will rock at summer time."
  '
```
> Response code: 202 Accepted

##### Errors handling

> - Sending a different payload than the `Content-Type` header should return a `400 bad_request` error.
> - Too large payload according to the limit should return a `413 payload_too_large` error
> - Wrong encoding should return a `400 bad_request` error
> - Invalid CSV data should return a `400 bad_request` error

### V. Impact on documentation

This feature should impact MeiliSearch users documentation by adding mention of csv capability inside Documents scope at [Add or replace documents](https://docs.meilisearch.com/reference/api/documents.html#add-or-replace-documents) and [Add or update documents](https://docs.meilisearch.com/reference/api/documents.html#add-or-update-documents). It should also mention that a missing Content-Type will be interpreted as `application/json` since it's the current behavior. Giving an `application/json` Content-Type leads to the same behavior.

We should also not only mention JSON format in `unsupported_media_type` section on the [errors page](https://docs.meilisearch.com/errors/#unsupported_media_type) and add CSV format. The documentation says "Currently, MeiliSearch supports only JSON payloads."

Documentation should also guide the user in the correct way to properly format and send csv data. Adding a dedicated page for the purpose of formatting and sending CSV data should be considered.

### VI. Impact on SDKs

This feature should impact MeiliSearch SDKs in the future by adding the possibility to send a CSV to MeiliSearch on the previous explicit endpoints. Simplifying the typing of the headers could also be handled by the SDKs.

## 2. Technical Aspects
N/A

## 3. Future possibilities

- Provide an interface in the future dashboard to upload CSV data into an index and optionally provide the headers types.
- Set a payload limit directly related to the type of data format. Currently, the payload size is equivalent to [JSON payload size](https://docs.meilisearch.com/reference/features/configuration.html#payload-limit-size). Metrics on feature usage and configuration update should help to choose a better suited value for this type of data format.
