- Title: Anonymous Analytics Policy
- Started At: 2021-04-16
- Updated At: 2021-10-13

# Anonymous Analytics Policy

## 1. Functional Specification

### I. Summary

This specification describes an exhaustive list of anonymous metrics collected by the MeiliSearch binary. It also describes the tools we use for this collection and how we identify a Meilisearch instance.

### II. Motivation

At MeiliSearch, our vision is to provide an easy-to-use search solution that meets the essential needs of our users. At all times, we strive to understand our users better and meet their expectations in the best possible way.

Although we can gather needs and understand our users through several channels such as Github, Slack, surveys, interviews or roadmap votes, we realize that this is not enough to have a complete view of MeiliSearch usage and features adoption. By cross-referencing our product discovery phases with aggregated quantitative data, we want to make the product much better than what it is today. Our decision-making will be taken a step further to make a product that users love.

### III. Explanation

#### General Data Protection Regulation (GDPR)

The metrics collected are non-sensitive, non-personal and do not identify an individual or a group of individuals using MeiliSearch. The data collected is secured and anonymized. We do not collect any data from the values stored in the documents.

We, the MeiliSearch team, provide an email address so that users can request the removal of their data: privacy@meilisearch.com.<br>
Thanks to the unique identifier generated for their MeiliSearch installation (`Instance UID` when launching MeiliSearch), we can remove the corresponding data from all the tools we describe below. Any questions regarding the management of the data collected can be sent to the email address as well.

#### Tools

##### Segment

The collected data is sent to [Segment](https://segment.com/). Segment is a platform for data collection and provides data management tools.

##### Amplitude

[Amplitude](https://amplitude.com/) is a tool for graphing and highlighting collected data. Segment feeds Amplitude so that we can build visualizations according to our needs.


----

#### Events table

| Event name | Description |
|------------|-------------|
| Launched   | Occurs when MeiliSearch is launched the first time. |
| Documents Searched POST | Aggregated event on all received requests via the `POST` - `indexes/:indexUid/search` route during one hour or until a batch size reaches `500Kb`. |
| Documents Searched GET | Aggregated event on all received requests via the `GET` - `/indexes/:indexUid/search` route during one hour or until a batch size reaches `500Kb`. |
| Documents Added | Aggregated event on all received requests via the `POST` - `/indexes/:indexUid/documents` route during one hour or until a batch size reaches `500Kb`. |
| Documents Updated | Aggregated event on all received requests via the `PUT` - `/indexes/:indexUid/documents` route during one hour or until a batch size reaches `500Kb`. |
| Index Created | Occurs when an index is created via `POST` - `/indexes`. |
| Index Updated | Occurs when an index is updated via `PUT` - `/indexes/:indexUid`. |
| Settings Updated | Occurs when the settings are updated via `POST` - `/indexes/:indexUid/settings`. |
| SearchableAttributes Updated | Occurs when searchable attributes are updated via `POST` - `/indexes/:indexUid/settings/searchable-attributes`. |
| RankingRules Updated | Occurs when ranking rules are updated via `POST` - `/indexes/:indexUid/settings/ranking-rules`. |
| FilterableAttributes Updated | Occurs when filterable attributes are updated via `POST` - `/indexes/:indexUid/settings/filterable-attributes`. |
| SortableAttributes Updated | Occurs when sortable attributes are updated via `POST` - `/indexes/:indexUid/settings/sortable-attributes`. |
| TypoTolerance Updated | Occurs when typo tolerance settings are updated via `POST` - `/indexes/:indexUid/settings/typo-tolerance`. |
| Dump Created | Occurs when a dump is created via `POST` - `/dumps`. |
| Tasks Seen | Occurs when tasks are fetched globally via `GET` - `/tasks`. |
| Index Tasks Seen | Occurs when tasks are filtered by index via `GET` - `/indexes/:indexUid/tasks`. |
----

#### Summarized Metrics/Events table

| Metric name                             | Description                                             | Example           | Triggered by |
|-----------------------------------------|---------------------------------------------------------|-------------------|--------------|
| `context.app.version`                   | MeiliSearch version number                              | 0.23.0            | Every hour   |
| `infos.env`                             | MeiliSearch env                                         | production        | Every hour   |
| `infos.db_path`                         | `true` if `--db-path`/`MEILI_DB_PATH` is specified, otherwise `false` | true | Every Hour |
| `infos.import_dump`                     | `true` if `--import-dump` is specified, otherwise `false` | true | Every Hour |
| `infos.dumps_dir`                       | `true` if `--dumps-dir`/`MEILI_DUMPS_DIR` is specified, otherwise `false` | true | Every Hour |
| `infos.ignore_missing_dump`             | `true` if `--ignore-missing-dump` is specified to true, otherwise `false` | true | Every Hour |
| `infos.ignore_dump_if_db_exists`        | `true` if `--ignore-dump-if-db-exists` is specified to true, otherwise `false` | true | Every Hour |
| `infos.import_snapshot`                 | `true` if `--import-snapshot` is specified, otherwise `false` | true | Every Hour |
| `infos.schedule_snapshot`               | `true` if `--schedule_snapshot`/`MEILI_SCHEDULE_SNAPSHOT` is specified to true, otherwise `false` | true | Every hour |
| `infos.snapshot_dir`                    | `true` if `--snapshot-dir`/`MEILI_SNAPSHOT_DIR` is specified to true, otherwise `false` | true | Every Hour |
| `infos.snapshot_interval_sec`           | Value of `--snapshot_interval_sec`/`MEILI_SNAPSHOT_INTERVAL_SEC` in seconds | 86400 | Every Hour |
| `infos.ignore_missing_snapshot`         | `true` if `--ignore_missing_snapshot` is specified to true, otherwise `false` | true | Every Hour |
| `infos.ignore_snapshot_if_db_exists`    | `true` if `--ignore_snapshot_if_db_exists` is specified to true, otherwise `false` | true | Every Hour |
| `infos.http_addr`                       | `true` if `--http-addr`/`MEILI_HTTP_ADDR` is specified, otherwise `false` | true| Every Hour |
| `infos.max_index_size`                  | Value of `--max-index-size`/`MEILI_INDEX_SIZE` in bytes | 336042103 | Every Hour |
| `infos.max_task_db_size`                | Value of `--max-task-db-size`/`MEILI_MAX_TASK_DB_SIZE` in bytes | 336042103 | Every Hour |
| `infos.http_payload_size_limit`         | Value of `--http-payload-size-limit`/`MEILI_HTTP_PAYLOAD_SIZE_LIMIT` in bytes | 336042103 | Every Hour |
| `infos.enable_auto_batching`            | `true` if `--enable-auto-batching` is specified to true, otherwise `false` | `true` | Every Hour |
| `infos.max_batch_size`                  | Value of `--max-batch-size` in integer, otherwise `null` | 1000 | Every Hour |
| `infos.max_documents_per_batch`         | Value of `--max-documents-per-batch` in integer, otherwise `null` | 1000 | Every Hour |
| `infos.debounce_duration_sec`           | Value of `--debounce-duration-sec` in seconds, otherwise `0` | 3600 | Every Hour |
| `infos.log_level`                       | Value of `--log-level`/`MEILI_LOG_LEVEL`                | debug             | Every Hour |
| `infos.max_indexing_memory`                      | Value of `--max-indexing-memory`/`MEILI_MAX_INDEXING_MEMORY` in bytes     | 336042103         | Every Hour |
| `infos.max_indexing_threads`                   | Value of `--max-indexing-threads`/`MEILI_MAX_INDEXING_THREADS` in integer | 4             | Every Hour |
| `system.distribution`                   | Distribution on which MeiliSearch is launched           | Arch Linux        | Every hour |
| `system.kernel_version`                 | Kernel version on which MeiliSearch is launched         | 5.14.10           | Every hour |
| `system.cores`                          | Number of cores                                         | 24                | Every hour |
| `system.ram_size`                       | Total RAM capacity. Expressed in `KB`                   | 16777216          | Every hour |
| `system.disk_size`                      | Total capacity of the largest disk. Expressed in `Bytes` | 1048576000        | Every hour |
| `system.server_prodiver`                | Users can tell us on which provider MeiliSearch is hosted by filling the `MEILI_SERVER_PROVIDER` env var. This is also filled by our cloud deploy scripts, e.g. [GCP cloud-config.yaml](https://github.com/meilisearch/cloud-scripts/blob/56a7c2630c1a508e5ad0c0ba1d8cfeb8d2fa9ae0/scripts/providers/gcp/cloud-config.yaml#L33)                                                                              | gcp               | Every hour |
| `stats.database_size`                   | Database size. Expressed in `Bytes`                     | 2621440           | Every hour |
| `stats.indexes_number`                  | Number of indexes                                       | 2                 | Every hour |
| `start_since_days`                      | Number of days since instance was launched              | 365               | Every hour |
| `user_agent`                            | User-agent header encountered during one or more API calls | ["Meilisearch Ruby (v2.1)", "Ruby (3.0)"] | `Documents Searched POST`, `Documents Searched GET`, `Index Created`, `Index Updated`, `Documents Added`, `Documents Updated`, `Settings Updated`, `Ranking Rules Updated`, `SortableAttributes Updated`, `FilterableAttributes Updated`, `SearchableAttributes Updated`, `Dump Created` |
| `requests.99th_response_time`           | Highest latency from among the fastest 99% of successful search requests | 57ms    | `Documents Searched POST`, `Documents Searched GET`|
| `requests.total_succeeded`              | Total number of successful search requests in this batch | 3456 | `Documents Searched POST`, `Documents Searched GET` |
| `requests.total_failed`                 | Total number of failed search requests in this batch    | 24   | `Documents Searched POST`, `Documents Searched GET` |
| `requests.total_received`               | Total number of received search requests in this batch  | 3480 | `Documents Searched POST`, `Documents Searched GET` |
| `sort.with_geoPoint`                    | `true` if the sort rule `_geoPoint` was used in this batch, otherwise `false` | true | `Documents Searched POST`, `Documents Searched GET` |
| `sort.avg_criteria_number`              | Average number of sort criteria among all requests containing the `sort` parameter in this batch | 2 | `Documents Searched POST`, `Documents Searched GET` |
| `filter.with_geoRadius`                 | `true` if the filter rule `_geoRadius` was used in this batch, otherwise `false` | false | `Documents Searched POST`, `Documents Searched GET` |
| `filter.most_used_syntax`               | Most used filter syntax among all requests containing the `filter` parameter in this batch | string | `Documents Searched POST`, `Documents Searched GET` |
| `q.max_terms_number`                    | Highest number of terms given for the `q` parameter in this batch | 5 | `Documents Searched POST`, `Documents Searched GET` |
| `pagination.max_limit`                  | Highest value given for the `limit` parameter in this batch | 60 | `Documents Searched POST`, `Documents Searched GET` |
| `pagination.max_offset`                 | Highest value given for the `offset` parameter in this batch | 1000 | `Documents Searched POST`, `Documents Searched GET` |
| `primary_key`                           | Value given for the `primaryKey` parameter if used, otherwise `null` | id | `Index Created`, `Index Updated`, `Documents Added`, `Documents Updated`|
| `payload_type`                          | All `payload_type` encountered in this batch | ["application/json", "text/plain", "application/x-ndjson"] | `Documents Added`, `Documents Updated` |
| `index_creation`                        | `true` if a document addition or update request triggered index creation in this batch, otherwise `false` | true | `Documents Added`, `Documents Updated` |
| `ranking_rules.sort_position`           | Position of the `sort` ranking rule | 5 | `Settings Updated`, `Ranking Rules Updated` |
| `sortable_attributes.total`             | Number of sortable attributes | 3 | `Settings Updated`, `SortableAttributes Updated`|
| `sortable_attributes.has_geo`           | `true` if `_geo` is set as a sortable attribute, otherwise `false` | true | `Settings Updated`, `SortableAttributes Updated` |
| `filterable_attributes.total`           | Number of filterable attributes | 3 | `Settings Updated`, `FilterableAttributes Updated` |
| `filterable_attributes.has_geo`         | `true` if `_geo` is set as a filterable attribute, otherwise `false` | false | `Settings Updated`, `FilterableAttributes Updated`|
| `searchable_attributes.total`           | Number of searchable attributes | 4 | `Settings Updated`, `SearchableAttributes Updated` |
| `typo_tolerance.enabled`                          | Whether the typo tolerance is enabled | `true` | `Settings Updated`, `TypoTolerance Updated` |
| `typo_tolerance.disable_on_attributes`              | `true` if at least one value is defined | `false` | `Settings Updated`, `TypoTolerance Updated` |
| `typo_tolerance.disable_on_words`                   | `true` if at least one value is defined | `false` | `Settings Updated`,  `TypoTolerance Updated` |
| `typo_tolerance.min_word_size_for_typos.one_typo` | The defined value for `minWordSizeForTypos.oneTypo` property | `5` | `Settings Updated`, `TypoTolerance Updated` |
| `typo_tolerance.min_word_size_for_typos.two_typos`| The defined value for `minWordSizeForTypos.twoTypos` property | `9` | `Settings Updated`, `TypoTolerance Updated` |
| `per_task_uid`                          | `true` if an uid is used to fetch a particular task resource, otherwise `false` | true | `Tasks Seen`, `Index Tasks Seen` |
|

----

#### Detailed list of Instance metrics and, events with their metrics

##### System Configuration `system`

This property allows us to gather essential information to better understand on which type of machine MeiliSearch is used. This allows us to better advise users on the machines to choose according to their data volume and their use-cases.

| Property name          | Description                                            | Example         |
|------------------------|--------------------------------------------------------|-----------------|
| system.distribution    | Distribution on which MeiliSearch is launched          | `Arch Linux`      |
| system.kernel_version  | Kernel version on which MeiliSearch is launched        | `5.14.10-arch1-1` |
| system.cores           | Number of cores                                        | `24`              |
| system.ram_size        | Total RAM capacity. Expressed in `KB`                  | `33604210`        |
| system.disk_size       | Total capacity of the largest disk. Expressed in `Bytes`| `336042103`       |
| system.server_provider | Users can tell us on which provider MeiliSearch is hosted by filling the `MEILI_SERVER_PROVIDER` env var. This is also filled by our providers deploy scripts. e.g. [GCP cloud-config.yaml](https://github.com/meilisearch/cloud-scripts/blob/56a7c2630c1a508e5ad0c0ba1d8cfeb8d2fa9ae0/scripts/providers/gcp/cloud-config.yaml#L33) | `gcp` |

##### MeiliSearch Configuration `context` and `infos`

| Property name | Description | Example |
|---------------|-------------|---------|
| context.app.version  | MeiliSearch version number.  Sent in a `context` object instead of `properties` to match Amplitude requirement | 0.23.0 |
| infos.env | Value of `--env`/`MEILI_ENV` | `production` |
| infos.db_path | `true` if `--db-path`/`MEILI_DB_PATH` is specified, otherwise `false` | `true`|
| infos.import_dump | `true` if `--import-dump` is specified, otherwise `false` | `true` |
| infos.dumps_dir | `true` if `--dumps-dir`/`MEILI_DUMPS_DIR` is specified, otherwise `false` | `true` |
| infos.ignore_missing_dump | `true` if `--ignore-missing-dump` is specified to true, otherwise `false` | `true` |
| infos.ignore_dump_if_db_exists | `true` if `--ignore-dump-if-db-exists` is specified to true, otherwise `false` | `true` |
| infos.import_snapshot | `true` if `--import-snapshot` is specified, otherwise `false` | `true` |
| infos.schedule_snapshot | `true` if `--schedule-snapshot`/`MEILI_SCHEDULE_SNAPSHOT` is specified to true, otherwise `false` | `true` |
| infos.snapshot_dir | `true` if `--snapshot-dir`/`MEILI_SNAPSHOT_DIR` is specified to true, otherwise `false` | `true` |
| infos.snapshot_interval_sec | Value of `--snapshot-interval-sec`/`MEILI_SNAPSHOT_INTERVAL_SEC` in seconds | `86400` |
| infos.ignore_missing_snapshot | `true` if `--ignore-missing-snapshot` is specified to true, otherwise `false` | `true` |
| infos.ignore_snapshot_if_db_exists | `true` if `--ignore-snapshot-if-db-exists` is specified to true, otherwise `false` | `true` |
| infos.http_addr | `true` if `--http-addr`/`MEILI_HTTP_ADDR` is specified, otherwise `false` | `true`|
| infos.max_index_size | Value of `--max-index-size`/`MEILI_INDEX_SIZE` in bytes | `336042103` |
| infos.max_task_db_size | Value of `--max-task-db-size`/`MEILI_MAX_TASK_DB_SIZE` in bytes | `336042103` |
| infos.http_payload_size_limit | Value of `--http-payload-size-limit`/`MEILI_HTTP_PAYLOAD_SIZE_LIMIT` in bytes | `336042103` |
| infos.enable_autobatching | `true` if `--enable-autobatching` is specified to true, otherwise `false` | `true` |
| infos.max_batch_size | Value of `--max-batch-size` in integer, otherwise `null` | `1000` |
| infos.max_documents_per_batch | Value of `--max-documents-per-batch` in integer, otherwise `null` | `1000` |
| infos.debounce_duration_sec | Value of `--debounce-duration-sec`in seconds, otherwise `0` | `3600` |
| infos.log_level | Value of `--log-level`/`MEILI_LOG_LEVEL`  | `debug` |
| infos.max_indexing_memory  | Value of `--max-indexing-memory`/`MEILI_MAX_INDEXING_MEMORY` in bytes     | `336042103` |
| infos.max_indexing_threads  | Value of `--max-indexing-threads`/`MEILI_MAX_INDEXING_THREADS` in integer | `4` |

##### MeiliSearch Statistics `stats`

| Property name | Description | Example |
|---------------|-------------|---------|
| stats.database_size  | Database size. Expressed in `Bytes` | `180230`|
| stats.indexes_number | Number of indexes | `2` |
| stats.documents_number | Number of indexed documents | `165847` |
| start_since_days | Number of days since instance was launched | `328` |

---

#### `Launched`

> This is the first event sent to mark that MeiliSearch is launched a first time.

#### `Documents Searched POST`

> The Documents Searched event is sent once an hour or when a batch reaches the maximum size of `500kb`. The event's properties are averaged over all requests on `POST` - `/indexes/:indexUid/search`.

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents all the user-agents encountered on this endpoint in the aggregated event.| `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| requests.99th_response_time | The maximum latency, in `ms`, for the fastest 99% of succeeded requests in the aggregated event. | `57ms` |
| requests.total_succeeded | The total number of succeeded search requests in the aggregated event. | `3456` |
| requests.total_failed | The total number of failed search requests in the aggregated event. | `24` |
| requests.total_received | The total number of received search requests in the aggregated event. | `3480` |
| sort.with_geoPoint | Does the built-in sort rule _geoPoint rule has been used in the aggregated event? | `true` |
| sort.avg_criteria_number | The average number of sort criteria among all the requests containing the `sort` parameter in the aggregated event. `"sort": []` equals to `0` while not sending `sort` does not influence the average. | `2` |
| filter.with_geoRadius | Does the built-in filter rule _geoRadius has been used in the aggregated event? | `false` |
| filter.avg_criteria_number | The average number of filter criteria among all the requests containing the `filter` parameter in the aggregated event. `"filter": []` equals to `0` while not sending `filter` does not influence the average in the aggregated event. | `4` |
| filter.most_used_syntax | The most used filter syntax among all the requests containing the requests containing the `filter` parameter in the aggregated event. `string` / `array` / `mixed` | `mixed` |
| q.max_terms_number | The maximum number of terms for the `q` parameter among all requests in the aggregated event. | `5` |
| pagination.max_limit | The maximum limit encountered among all requests in the aggregated event. | `20` |
| pagination.max_offset | The maxium offset encountered among all requests in the aggregated event. | `1000` |

---

#### `Documents Searched GET`

> The Documents Searched event is sent once an hour or when a batch reaches the maximum size of `500kb`. The event's properties are averaged over all requests on `GET` - `/indexes/:indexUid/search`.

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents all the user-agents encountered on this endpoint in the aggregated event.| `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| requests.99th_response_time | Highest latency from among the fastest 99% of successful search requests. | `57ms` |
| requests.total_succeeded | The total number of succeeded search requests in the aggregated event. | `3456` |
| requests.total_failed | The total number of failed search requests in the aggregated event. | `24` |
| requests.total_received | The total number of received search requests in the aggregated event. | `3480` |
| sort.with_geoPoint | Does the built-in sort rule _geoPoint rule has been used in the aggregated event? | `true` |
| sort.avg_criteria_number | The average number of sort criteria among all the requests containing the `sort` parameter in the aggregated event. `"sort": []` equals to `0` while not sending `sort` does not influence the average. | `2` |
| filter.with_geoRadius | Does the built-in filter rule _geoRadius has been used in the aggregated event? | `false` |
| filter.avg_criteria_number | The average number of filter criteria among all the requests containing the `filter` parameter in the aggregated event. `"filter": []` equals to `0` while not sending `filter` does not influence the average in the aggregated event. | `4` |
| filter.most_used_syntax | The most used filter syntax among all the requests containing the requests containing the `filter` parameter in the aggregated event. `string` / `array` / `mixed` | `mixed` |
| q.max_terms_number | The maximum number of terms for the `q` parameter among all requests in the aggregated event. | `5` |
| pagination.max_limit | The maximum limit encountered among all requests in the aggregated event. | `20` |
| pagination.max_offset | The maxium offset encountered among all requests in the aggregated event. | `1000` |

---

## `Index Created`

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered for this API call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| primary_key   | The field's name used as a primary key if set, otherwise `null`. | `id` |

---

## `Index Updated`

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered for this API call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| primary_key   | The field's name used as a primary key if set, otherwise `null`. | `id` |

---

## `Documents Added`

> The Documents Added event is sent once an hour or when a batch reaches the maximum size of `500kb`. The event's properties are averaged over all requests on `POST` - `/indexes/:indexUid/documents`.

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents all the user-agents encountered on this endpoint in the aggregated event. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| payload_type | Represents all the payload_type encountered on this endpoint in the aggregated event as a set. `application/json`/ `application/x-ndjson`/ `text/plain` or any non-supported content-type. | [`text/plain`, `application/json`] |
| primary_key   | Represents all the `primaryKey` query parameters encountered in the aggregated event as a set, otherwise `null`. | `["id"]` |
| index_creation | Does an index creation happened among all requests in the aggregated event? | `false`|

---

## `Documents Updated`

> The Documents Updated event is sent once an hour or when a batch reaches the maximum size of `500kb`. The event's properties are averaged over all requests on `PUT` - `/indexes/:indexUid/documents`.

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents all the user-agents encountered on this endpoint in the aggregated event. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| payload_type | Represents all the payload_type encountered on this endpoint in the aggregated event as a set. `application/json`/ `application/x-ndjson`/ `text/plain` or any non-supported content-type. | [`text/plain`, `application/json`] |
| primary_key   | Represents all the `primaryKey` query parameters encountered in the aggregated event as a set, otherwise `null`. | `["id"]` |
| index_creation | Does an index creation happened among all requests in the aggregated event? | `false`|

---

## `Settings Updated`


| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered on this call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| searchable_attributes.total | Number of searchable attributes. | `3`|
| ranking_rules.sort_position | Position of the `sort` ranking rule if any, otherwise `null`. | `5` |
| sortable_attributes.total   | Number of sortable attributes. | `3` |
| sortable_attributes.has_geo | Indicate if `_geo` is set as a sortable attribute. | `false`|
| filterable_attributes.total   | Number of filterable attributes. | `3` |
| filterable_attributes.has_geo | Indicate if `_geo` is set as a filterable attribute. | `false`|
| typo_tolerance.enabled        | Whether the typo tolerance is enabled. | `true` |
| typo_tolerance.disable_on_attributes | `true` if at least one value is defined for `disableOnAttributes` property. | `false` |
| typo_tolerance.disable_on_words    | `true` if at least one value is defined for `disableOnWords` property. | `false` |
| typo_tolerance.min_word_size_for_typos.one_typo | The defined value for `minWordSizeForTypos.oneTypo` property. | `5` |
| typo_tolerance.min_word_size_for_typos.two_typos | The defined value for `minWordSizeForTypos.twoTypos` property. | `9` |

---

## `RankingRules Updated`

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered on this call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| ranking_rules.sort_position | Position of the `sort` ranking rule if any, otherwise `null`. | `5` |

---

## `SortableAttributes Updated`

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered on this call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| sortable_attributes.total   | Number of sortable attributes. | `3` |
| sortable_attributes.has_geo | Indicate if `_geo` is set as a sortable attribute. | `false`|

---

## `FilterableAttributes Updated`

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered on this call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| filterable_attributes.total   | Number of filterable attributes. | `3` |
| filterable_attributes.has_geo | Indicate if `_geo` is set as a filterable attribute. | `false`|

## `SearchableAttributes Updated`

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered on this call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| searchable_attributes.total   | Number of searchable attributes. | `3` |


## `TypoTolerance Updated`

| Property name | Description | Example |
|---------------|-------------|---------|
| typo_tolerance.enabled        | Whether the typo tolerance is enable.d | `true` |
| typo_tolerance.disable_on_attributes | `true` if at least one value is defined for `disableOnAttributes` property. | `false` |
| typo_tolerance.disable_on_words    | `true` if at least one value is defined for `disableOnWords` property. | `false` |
| typo_tolerance.min_word_size_for_typos.one_typo | The defined value for `minWordSizeForTypos.oneTypo` property. | `5` |
| typo_tolerance.min_word_size_for_typos.two_typos | The defined value for `minWordSizeForTypos.twoTypos` property. | `9` |

## `Dump Created`

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered on this call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |

## `Tasks Seen`

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered on this call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| per_task_uid  | `true` if an uid is used to fetch a particular task resource, otherwise `false` | `true` |

## `Index Tasks Seen`

| Property name | Description | Example |
|---------------|-------------|---------|
| user_agent    | Represents the user-agent encountered on this call. | `["Meilisearch Ruby (v2.1)", "Ruby (3.0)"]` |
| per_task_uid  | `true` if an uid is used to fetch a particular task resource, otherwise `false` | `true` |

---

#### User-interface

Analytics are enabled by default while leaving the option for users to disable it with the `--no-analytics` option.

**Message displayed on the CLI at launch if analytics are enabled**

> Thank you for using MeiliSearch!
>
> We collect anonymized analytics to improve our product and your experience. To learn more, including how to turn off analytics, visit our dedicated documentation page: https://docs.meilisearch.com/learn/what_is_meilisearch/telemetry.html.
>
> Anonymous telemetry:	"Enabled"
>
> Instance UID: ":uidGeneratedAtFirstLaunch"

**Message displayed on the CLI at launch if analytics are disabled after being activated**

The unique identifier of the instance remains displayed even if analytics are disabled so that it does not reactivate the analytics to obtain it after having stopped it. The user can still ask us to remove the data previously collected by giving us his `Instance UID`.

> Thank you for using MeiliSearch!
>
> Anonymous telemetry:	"Disabled"
>
> Instance UID: ":uidGeneratedAtFirstLaunch"

**Message displayed on the CLI at launch if analytics are disabled at first launch**

> Thank you for using MeiliSearch!
>
> Anonymous telemetry:	"Disabled"

## 2. Technical Aspects

### I. Technical Details

#### User-Agent case

The `User-Agent` header is tracked on the events listed below. Our official SDKs/integrations should always contain `MeiliSearch` in their names.

Each endpoint API tracked sends the `User-Agent` as a `user_agent` event property as an array. If several values are contained in the `User-Agent` header, they are split by the `;` character.

#### Identifying MeiliSearch installation

To identify instances, we generate a unique identifier at first launch if analytics are not disabled.

- This unique identifier is inserted in the data.ms folder to be kept in case of version upgrades.
- A file named following the given pattern `:MeiliSearchData.msPath-:instanceUid` is generated in a `MeiliSearch` folder to recover an identifier in case of corruption of the data.ms folder, causing it to be deleted and recreated. This `MeiliSearch` folder is created in the `config_dir` of each platform.

| Directory	| Windows |	Linux/*BSD | macOS |
|-----------|---------|------------|-------|
|config_dir	| %APPDATA% (C:\Users\%USERNAME%\AppData\Roaming) |	$XDG_CONFIG_HOME (~/.config) |	~/Library/Application Support |

#### Segment Identify Call

The `identify` method of Segment permits identifying an instance by sending a unique identifier. It groups the information of a MeiliSearch binary such as `system`, `stats`, and general properties related below in this specification.

The segment identify call is only sent after the first hour if the instance is still running. At the first launch, MeiliSearch sends a `Launched` event on an instance ID equal to `total_launch` in order to avoid tracking instances usage for nothing when they could be shut down and never restarted.

#### Segment Track Call

The `track` calls of Segment allow tracking the events passed on the instance.

#### Batching

A batch is sent every hour or when it reaches the maximum size of `500Kb` to avoid sending analytics in real-time and preserve network exchanges.

This batch contains an identify payload and all tracked events that occurred during this hour.

#### Logging

Errors occurring when sending metrics to Segment should be silent. In general, the impact of data collection should be minimized as much as possible concerning performance and be entirely transparent for the user during its use.

#### Debug build

In debug build, no analytics are collected.

## 3. Future possibilities
n/a
