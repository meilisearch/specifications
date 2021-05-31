- Title: Indexing NDJSON
- Start Date: 2021-04-12
- Specification PR: [PR-#29](https://github.com/meilisearch/specifications/pull/29)
- MeiliSearch Tracking-Issues: TBD

# Indexing NDJSON

## 1. Feature Description and Interaction

### I. Summary

To index documents, the body of the add documents request has to match a specific format. That specific format is then parsed and tokenized inside MeiliSearch. After which, the documents added are in the pool of searchable and returnable documents.

An [NDJSON](http://ndjson.org/) data format is easier to use than a CSV format because it propose a convenient format for storing structured data.

### II. Motivation

Currently, the engine only accepts JSON format as a data source. We want to give users the possibility of another simple data format to use. Thus, give them more versatility at the data source choices for the indexing step.

Writing performance is also a motivation since JSON Lines data parsing is less CPU and memory-intensive than parsing standard JSON. When new lines represent separate entries it makes the NDJSON data streamable, thus, more suited for indexing a consequent data set.

While we give the ability to Meilisearch to ingest CSV data for indexing in this [specification](https://github.com/meilisearch/specifications/pull/28), we are aware of the limitations of CSV so we also want to provide a format that is easy to validate. Handling the validity of a CSV can be frustrating and difficult. Only strings can be managed within a CSV. In addition, there is no official specification except [RFC 4180](https://tools.ietf.org/html/rfc4180) which is not sufficient for all data scheme.

Representing nested structures in a JSON object is easy and convenient.

### III. Additional Materials

TBD

### IV. Explanation

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

#### Add or Replace Documents [📎](https://docs.meilisearch.com/reference/api/documents.html#add-or-replace-documents)

```curl
curl \
  -X POST 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: application/x-ndjson' \
  --binary-data '
    {"id":1, "label": "t-shirt", "price": 4.99, "colors": ["red", "green", "blue"]}\n
    {"id":499, "label": "hoodie", "price": 19.99, "colors": ["purple"]}
  '
```
> Response code: 202 Accepted

##### Error codes

> - Sending a different payload than the `Content-Type` header should return a `400 bad_request` error.
> - Too large payload according to the limit should return a `413 payload_too_large` error
> - Wrong encoding should return a `400 bad_request` error
> - Invalid NDJSON data should return a `400 bad_request` error

### Add or Update Documents [📎](https://docs.meilisearch.com/reference/api/documents.html#add-or-update-documents)

```curl
curl \
  -X PUT 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: application/x-ndjson' \
  --binary-data '
    {"id":1, "label": "t-shirt", "price": 4.99, "colors": ["red", "green", "blue"]}\n
    {"id":499, "label": "hoodie", "price": 19.99, "colors": ["purple"]}
  '
```
> Response code: 202 Accepted

##### Errors handling

> - Sending a different payload than the `Content-Type` header should return a `400 bad_request` error.
> - Too large payload according to the limit should return a `413 payload_too_large` error
> - Wrong encoding should return a `400 bad_request` error
> - Invalid NDJSON data should return a `400 bad_request` error

### V. Impact on documentation

This feature should impact MeiliSearch users documentation by adding the possibility to use `ndjson` as an accepted format in the  Documents scope at [Add or replace documents](https://docs.meilisearch.com/reference/api/documents.html#add-or-replace-documents) and [Add or update documents](https://docs.meilisearch.com/reference/api/documents.html#add-or-update-documents). It should also mention that a missing Content-Type will be interpreted as `application/json` since it's the current behavior. Giving an `application/json` Content-Type leads to the same behavior.

We should also not only mention JSON format in `unsupported_media_type` section on the [errors page](https://docs.meilisearch.com/errors/#unsupported_media_type) and add `ndjson` format. The documentation says "Currently, MeiliSearch supports only JSON payloads."

Documentation should also guide the user in the correct way to properly format and send ndjson data. Adding a dedicated page for the purpose of formatting and sending ndjson data should be considered.

### VI. Impact on SDKs

This feature should impact MeiliSearch SDK's in the future by adding the possibility to send ndjson data to MeiliSearch on the previous explicited endpoints.

## 2. Technical Aspects

### I. Technical details

⚠ A missing Content-Type will be interpreted as `application/json` since it's the current behavior. Giving an `application/json` Content-Type leads to the same behavior.

## 3. Future possibilities
- Provide an interface in the future dashboard to upload NDJSON data into an index.
- Set a payload limit directly related to the type of data format. Currently, the payload size is equivalent to [JSON payload size](https://docs.meilisearch.com/reference/features/configuration.html#payload-limit-size). Metrics on feature usage and configuration update should help to choose a better suited value for this type of data.
