- Title: Csv Indexation
- Start Date: 2021-04-9
- Specification PR: [PR-#28](https://github.com/meilisearch/specifications/pull/28)
- MeiliSearch Issues: [#128](https://github.com/meilisearch/transplant/issues/128),[#1332](https://github.com/meilisearch/MeiliSearch/issues/1332)

## 1. Feature Description and Interaction

### I. Summary

The initiation step of document indexing is to send some file matching a format to be parsed and tokenized in order to give search results to end-users. A [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) file format is broadly used to store and exchange data in a simple format.

Also, in order to boost write performance csv files are more suited than JSON for consequent datasets.

### II. Motivation

We want to provide our users with an always improved usage experience. Currently, the engine only accepts JSON format as a data source. We want to give users the possibility of another simple file format, well known, to use. Thus, give them more versatility at the data source choices for the indexation (add and update) step.

Since most of SQL engines or SQL clients can easily dump data as a csv file, it will facilitate MeiliSearch adoption by extending the indexation step on a wider range of customer cases than before.

Writing performance is also considered as a motivation since CSV parsing is less CPU and memory intensive than parsing Json due to the streamable capability of the CSV format.

### III. Additional Materials
N/A

### IV.Explanation

#### Csv Formatting Rules

While there's [RFC 4180](https://tools.ietf.org/html/rfc4180) as a try to add a specification for csv format, we will find a lot of variations from that. MeiliSearch features capabilities requires the csv file to be formatted the proper way to be parsable by the engine.

- A csv file need to contain a first line representing the list of attributes with the choosen type separated from the attribute name by `:` character. 

> An attribute can be specificed with two types: `string` or `number`. A missing type will be interpreted as a `string` by default.
>
> Valid headline example: "id:number","title:string","author","price:number"

- Each of the following lines will represent a document for MeiliSearch.
- Each value should be enclosed in double quotes tags to handle comma character presence inside.
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
      "description": "Hey, you will rock at summer time.",
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
 
#### Add or Replace Documents [📎](https://docs.meilisearch.com/reference/api/documents.html#add-or-replace-documents)

```curl
curl \
  -X POST 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: text/csv' \
  --data-binary data.csv
```
> Response code: 202 Accepted

##### Error codes

> - Sending a different payload than the `Content-Type` header should return a `415 unsupported_media_type` error.
> - Too large payload according to the limit should return a `413 payload_too_large` error 
> - Wrong file encoding should return a `420 unprocessable_entity` error
> - Invalid CSV data should return a `420 unprocessable_entity` error

### Add or Update Documents [📎](https://docs.meilisearch.com/reference/api/documents.html#add-or-update-documents)

```curl
curl \
  -X PUT 'http://localhost:7700/indexes/movies/documents' \
  -H 'Content-Type: text/csv' \
  --data-binary data.csv
```
> Response code: 202 Accepted

##### Errors handling

> - Sending a different payload than the `Content-Type` header should return a `415 unsupported_media_type` error.
> - Too large payload according to the limit should return a `413 payload_too_large` error 
> - Wrong file encoding should return a `420 unprocessable_entity` error
> - Invalid CSV data should return a `420 unprocessable_entity` error

### V. Impact on documentation

This feature should impact MeiliSearch users documentation by adding mention of csv capability inside Documents scope at [Add or replace documents](https://docs.meilisearch.com/reference/api/documents.html#add-or-replace-documents) and [Add or update documents](https://docs.meilisearch.com/reference/api/documents.html#add-or-update-documents).

We should also not only mention JSON format in `unsupported_media_type` section on the [errors page](https://docs.meilisearch.com/errors/#unsupported_media_type) and add CSV format.

Documentation should also guide the user in the correct way to properly format and send the csv file. Adding a dedicated page for the purpose of formatting and sending a CSV file should be considered.

### VI. Impact on SDKs

This feature should impact MeiliSearch SDK's in the future by adding the possibility to send a CSV to MeiliSearch on the previous explicited endpoints.

## 2. Technical Implementation

### I. Architecture
N/A

### II. Implementation Details
N/A

### III. Corner Cases
N/A

## 3. Future possibilities

- Provide an interface in the future dashboard to upload CSV data into an index.
- Set a payload limit directly related to the type of files. Currently, the payload size is equivalent to [JSON payload size](https://docs.meilisearch.com/reference/features/configuration.html#payload-limit-size). Metrics on feature usage and configuration update should help to choose a better suited value for this type of file.