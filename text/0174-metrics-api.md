# Metrics API

This endpoint is currently in beta.

This means that it can break at any time between two minor versions as long as it is not stabilized.

## 1. Summary

This specification describes the metrics API endpoint with the exhaustive list of returned metrics.

The endpoint returns observability data to monitor a Meilisearch instance using [prometheus](https://prometheus.io/).

## 2. Motivation

Improve the capabilities of a Meilisearch instance regarding observability and ease its integration into monitoring stacks.

## 3. Functionnal Specification

### 3.1. Activating the feature

By default, the `/metrics` endpoint is not accessible. To activate it, the `--enable-metrics-route` CLI option must be specified at launch.

### 3.2. `metrics` API resource definition

Prometheus metrics format is text-based and line-oriented. Lines are separated by a line feed character (n).

A metric is composed by several fields:

- `# HELP` metadata
- `# TYPE` metadata
- Metric name
- Current metric value

Meilisearch returns the metrics specified in the table below.

| Metric                                                         |
|----------------------------------------------------------------|
| [`http_requests_total`](#321-http_requests_total)              |
| [`meilisearch_database_size`](#322-meilisearch_database_size)  |
| [`meilisearch_docs_count`](#323-meilisearch_docs_count)        |
| [`meilisearch_total_index`](#324-meilisearch_total_index)      |
| [`http_response_time_seconds`](#325-http_response_time_seconds)|

#### 3.2.1 `http_requests_total`
tbd

#### 3.2.2 `meilisearch_database_size`

Return the size of the database in bytes.

```
# HELP meilisearch_database_size Meilisearch Stats DbSize
# TYPE meilisearch_database_size gauge
meilisearch_database_size :databaseSizeInBytes
```

#### 3.2.3 `meilisearch_docs_count`

Return the number of documents for an index.
Each index produce a `meilisearch_docs_count` line.

```
# HELP meilisearch_docs_count Meilisearch Stats Docs Count
# TYPE meilisearch_docs_count gauge
meilisearch_docs_count{index=":indexUid"} :numberOfDocuments
```

#### 3.2.4. `meilisearch_total_index`

Return the total number of index for the Meilisearch instance.

```
# HELP meilisearch_total_index Meilisearch Stats Index Count
# TYPE meilisearch_total_index gauge
meilisearch_total_index :numberOfIndexes
````

#### 3.2.5. `http_responses_time_seconds`

tbd

### 3.3. API Endpoints Definition

#### 3.3.1. `GET` - `metrics`

Fetch the metrics of the Meilisearch instance.

`200` - Response body

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

- 🔴 If `--enable-metrics-route` is not specified at launch, the API returns a `404 Not Found` HTTP response.