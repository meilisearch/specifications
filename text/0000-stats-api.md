# Stats API

## 1. Summary

This specification describes the stats API endpoints.

Stats routes give extended information and metrics about indexes and the Meilisearch database.

## 2. Motivation
N/A

## 3. Functional Specification

### 3.1. API Endpoints Definition

Manipulate documents of a Meilisearch index.

- [3.1.1. `GET` - `stats`](#311-get---stats)
- [3.1.2. `GET` - `indexes/:index_uid/stats`](#312-get---indexesindexuidstats)

#### 3.1.1. `GET` - `stats`

Get stats for all indexes.

##### 3.1.1.1. Path Parameters

| Parameter               | Type                     | Required |
|-------------------------|--------------------------|----------|
| `index_uid`             | String                   | true     |


###### 3.1.1.1.1. `index_uid`

- Type: String
- Required: True

Unique identifier of an index.

##### 3.1.1.2. Response Definition

In addition to all fields returned by [3.1.2. `GET` - `indexes/:index_uid/stats`](#312-get---indexesindexuidstats), this route returns the instance-level fields.

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `databaseSize`           | Integer                  | true     |
| `lastUpdate`             | String                   | true     |
| `indexes`                | Object                   | true     |

###### 3.1.1.2.1. `databaseSize`

- Type: Integer
- Required: True

Size of the database in bytes. It represents the size on disk of all the indexes.

###### 3.1.1.2.2. `lastUpdate`

- Type: String
- Required: True

When the last update was made to the database in the RFC 3339 format.

###### 3.1.1.2.3. `indexes`

- Type: Object
- Required: True

Object representing the statistics for each index found in the database.

e.g.

```json
{
  "databaseSize": 447819776,
  "lastUpdate": "2019-11-15T11:15:22.092896Z",
  "indexes": {
    "movies": {
      "numberOfDocuments": 19654,
      "isIndexing": false,
      "fieldDistribution": {
        "poster": 19654,
        "overview": 19654,
        "title": 19654,
        "id": 19654,
        "release_date": 19654
      }
    }
}
```

See [3.1.2.2. Response Definition](#3122-response-definition) section.

##### 3.1.1.3. Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.

#### 3.1.2. `GET` - `indexes/:index_uid/stats`

Get stats of an index.

##### 3.1.2.1. Path Parameters

| Parameters               | Type                     | Required |
|--------------------------|--------------------------|----------|
| `index_uid`              | String                   | true     |

###### 3.1.2.1.1. `index_uid`

- Type: String
- Required: True

Unique identifier of an index.

##### 3.1.2.2. Request Payload Definition
N/A

##### 3.1.2.3. Response Definition

| Field                    | Type                     | Required |
|--------------------------|--------------------------|----------|
| `numberOfDocuments`      | Integer                  | true     |
| `isIndexing`             | Boolean                  | true     |
| `fieldDistribution`      | Object                   | true     |

###### 3.1.2.3.1. `numberOfDocuments`

- Type: Integer
- Required: True

Total number of documents in an index.

###### 3.1.2.3.2. `isIndexing`

- Type: Boolean
- Required: True

If true, it indicates that the index is indexing documents.

###### 3.1.2.3.3. `fieldDistribution`

- Type: Object
- Required: True

Lists every field in the index along with the total number of documents in the index that contain that field.

`fieldDistribution` is not impacted by searchableAttributes or displayedAttributes. Meaning, if one of the fields is not displayed or searchable, it will still be displayed in the `fieldDistribution` object.

##### 3.1.2.4. Errors

- 🔴 If the requested `index_uid` does not exist, the API returns an [index_not_found](0061-error-format-and-definitions.md#index_not_found) error.


#### 3.1.3. General Errors

These errors apply to all endpoints described here.

##### 3.1.3.1 Auth Errors

The auth layer can return the following errors if Meilisearch is secured (a master-key is defined).

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 4. Technical Details
N/A

## 5. Future Possibilities

- Rename `lastUpdate` to `updatedAt`
- Reconsider the existence of `isIndexing`