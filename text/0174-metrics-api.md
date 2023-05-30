# Metrics API

This endpoint is currently experimental.

This means that it can break at any time between two minor versions as long as it is not stabilized.

## 1. Summary

This specification describes the metrics API endpoint with the exhaustive list of returned metrics.

The endpoint returns observability data to monitor a Meilisearch instance using [prometheus](https://prometheus.io/).

## 2. Motivation

Improve the capabilities of a Meilisearch instance regarding observability and ease its integration into monitoring stacks.

## 3. Functionnal Specification

### 3.1. Activating the feature

By default, the `/metrics` endpoint is not accessible. To activate it, the `--experimental-enable-metrics` CLI option or `MEILI_EXPERIMENTAL_ENABLE_METRICS` env var must be specified at launch. See [Instance Options](0119-instance-options.md)

### 3.2. `metrics` API resource definition

Prometheus metrics format is text-based and line-oriented. Lines are separated by a line feed character (n).

A metric is composed by several fields:

- `# HELP` metadata
- `# TYPE` metadata
- Metric name
- Current metric value

Meilisearch returns the metrics specified in the table below.

| Name                                                                                   | Type      |
|----------------------------------------------------------------------------------------|-----------|
| [`meilisearch_http_requests_total`](#321-meilisearch_http_requests_total)              | counter   |
| [`meilisearch_http_response_time_seconds`](#322-meilisearch_http_response_time_seconds)| histogram |
| [`meilisearch_db_size_bytes`](#323-meilisearch_db_size_bytes)                          | gauge     |
| [`meilisearch_used_db_size_bytes`](#324-meilisearch_used_db_size_bytes)                | gauge     |
| [`meilisearch_index_docs_count`](#325-meilisearch_index_docs_count)                    | gauge     |
| [`meilisearch_index_count`](#326-meilisearch_index_count)                              | gauge     |
| [`meilisearch_nb_tasks`](#327-meilisearch_nb_tasks)                                    | counter   |

#### 3.2.1 `meilisearch_http_requests_total`

Returns the number of times an API resource is accessed.

```
# HELP http_requests_total HTTP requests total
# TYPE http_requests_total counter
http_requests_total{method=":httpMethod",path=":resourcePath"} :numberOfRequest
```

#### 3.2.2. `meilisearch_http_response_time_seconds`

Returns a time histogram showing the number of times an API resource call goes into a time bucket (expressed in second).

```
# HELP http_response_time_seconds HTTP response times
# TYPE http_response_time_seconds histogram
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.0005"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.0008"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.00085"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.0009"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.00095"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.001"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.00105"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.0011"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.00115"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.0012"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.0015"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.002"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="0.003"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="1"} :numberOfRequest
http_response_time_seconds_bucket{method=":httpMethod",path=":resourcePath",le="+Inf"} :numberOfRequest
http_response_time_seconds_sum{method=":httpMethod",path=":resourcePath"} :numberOfRequest
http_response_time_seconds_count{method=":httpMethod",path=":resourcePath"} :numberOfRequest
```

#### 3.2.3. `meilisearch_db_size_bytes`

Returns the size of the database in bytes.

```
# HELP meilisearch_db_size_bytes Meilisearch Db Size In Bytes
# TYPE meilisearch_db_size_bytes gauge
meilisearch_db_size_bytes :databaseSizeInBytes
```

#### 3.2.4. `meilisearch_used_db_size_bytes`

Returns the size of the database actually used by meilisearch in bytes.

```
# HELP meilisearch_used_db_size_bytes Meilisearch Used DB Size In Bytes
# TYPE meilisearch_used_db_size_bytes gauge
meilisearch_used_db_size_bytes :databaseSizeInBytes
```

#### 3.2.5. `meilisearch_index_docs_count`

Returns the number of documents for an index.

```
# HELP meilisearch_index_docs_count Meilisearch Index Docs Count
# TYPE meilisearch_index_docs_count gauge
meilisearch_index_docs_count{index=":indexUid"} :numberOfDocuments
```

#### 3.2.6. `meilisearch_index_count`

Returns the total number of index for the Meilisearch instance.

```
# HELP meilisearch_index_count Meilisearch Index Count
# TYPE meilisearch_index_count gauge
meilisearch_index_count :numberOfIndex
````

#### 3.2.7. `meilisearch_nb_tasks`

Returns the total number of index for the Meilisearch instance.

```
# HELP meilisearch_nb_tasks Meilisearch Number of tasks
# TYPE meilisearch_nb_tasks gauge
meilisearch_nb_tasks{kind=":kind",value=":value"} :number
````

Here is a list of available kind and associated values:

| Kind      | values                                                                 |
|-----------|------------------------------------------------------------------------|
| indexes   | Any created indexes                                                    |
| statuses  | Any task statuses (i.e.: succeeded, failed, enqueued, etc...)          |
| types     | Any task types (i.e.: documentAdditionOrUpdate, settingsUpdate, etc...)|

### 3.3. API Endpoints Definition

#### 3.3.1. `GET` - `/metrics`

Fetch the metrics of the Meilisearch instance.

`200` - Response body example

```
# HELP meilisearch_database_size MeiliSearch Stats DbSize
# TYPE meilisearch_database_size gauge
meilisearch_database_size 1097728
# HELP meilisearch_docs_count MeiliSearch Stats Docs Count
# TYPE meilisearch_docs_count gauge
meilisearch_docs_count{index="movies"} 807
meilisearch_docs_count{index="movies_2"} 0
# HELP meilisearch_total_index MeiliSearch Stats Index Count
# TYPE meilisearch_total_index gauge
meilisearch_total_index 2
```

#### 3.3.2. Errors

- 🔴 If `--experimental-enable-metrics` CLI option / `MEILI_EXPERIMENTAL_ENABLE_METRICS` env var is not specified at launch, the API returns a `404 Not Found` HTTP response.

##### 3.3.2.1 Auth Errors

If a master key is used to secure a Meilisearch instance, the auth layer returns the following errors:

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have the permission `metrics.get` (i.e. other than the master key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.
- 🔴 Accessing this route with a key that have a restriction on the indexes returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

## 4. Technical Details
N/A

## 5. Future Possibilities

- Merge `/stats` with `/metrics`. A header could specify the prefered format. e.g `application/json` (similar to actual `stats` resource) or `text/plain` (prometheus)