- Title: Logging
- Start Date: 2021-04-21
- Specification PR: [#33](https://github.com/meilisearch/specifications/pull/33)
- MeiliSearch Tracking-issues:

# Logging

## 1. Functional Specification

### I. Summary

When I use MeiliSearch I want to see what's going on under the hood because it is useful at each stage of development to debug, tweak and maintain MeiliSearch in good conditions giving my usages.

### II. Motivation

Giving tracing about the behavior of a system is useful to those who test, develop and use it in production. The goal of this specification is to state on the logging behavior of the new search engine (Milli).

### III. Additional Materials

#### Algolia

Algolia offers an API endpoint dedicated to retrieving the logs of search and indexing operations.

> A call on the `getLogs` endpoint affect the algolia quota but it is not counted as a log.
>
> The logs are kept for a period of 7 days. After this period, they are no longer accessible from the API.

Algolia gives parameters to modify the request according to the user's needs.

> A query performed without parameters will return the last 10 records by default.
>
> The maximum number of logs that can be returned per request is 1000.

It is possible to use `offset` and `length` parameters to search through the log entries.

A `type` parameter is also provided to select the type of log to retrieve.

| Value | Description                      |
|-------|----------------------------------|
| all   | All the logs                     |
| query | Exclusively the queries          |
| build | Exclusively the build operations |
| error | Exclusively the errors           |

Here is the information that Algolia chooses to return:

| Key                | Description                                                                                     |
|--------------------|-------------------------------------------------------------------------------------------------|
| timestamp         | Timestamp in ISO-8601 format                                                                   |
| method             | Rest type of the method                                                                        |
| answer_code        | HTTP response code                                                                              |
| query_body         | Request body. Limited to 1000 characters                                             |
| answer             | Answer body. Limited to 1000 characters                                              |
| url                | Request URL                                                                                    |
| ip                 | Client ip of the call                                                                          |
| query_headers      | Request Headers (API Key is obfuscated)                                                        |
| sha1               | Id of log entry                                                                               |
| nb_api_calls       | Number of API calls                                                                            |
| processing_time_ms | Processing time for the query (Does not include network time)                              |
| query_nb_hits      | Number of hits returned for the query                                                          |
| exhaustive         | Exhaustive flags used during the query                                                         |
| index              | Index name of the log                                                                          |
| inner_queries      | Contains an object for each performed query with the `indexName`, `queryID`, `offset`, and `userToken` |
> Source: ***[Algolia documentation](https://www.algolia.com/doc/api-reference/api-methods/get-logs/)***
#### TypeSense

TypeSense makes no mention of logs in its documentation. However, the tracing policy seems to be in verbose mode and give a lot of more or less relevant information.

```
typesense_1     | I20210403 01:06:33.688689     1 typesense_server_utils.cpp:301] Starting Typesense 0.19.0
typesense_1     | I20210403 01:06:33.690609     1 typesense_server_utils.cpp:304] Typesense is using jemalloc.
typesense_1     | I20210403 01:06:33.730222     1 typesense_server_utils.cpp:405] Starting API service...
typesense_1     | I20210403 01:06:33.730813    82 typesense_server_utils.cpp:210] Since no --nodes argument is provided, starting a single node Typesense cluster.
typesense_1     | I20210403 01:06:33.734120     1 http_server.cpp:189] Typesense has started listening on port 8108
typesense_1     | I20210403 01:06:33.744827    82 server.cpp:1045] Server[braft::RaftStatImpl+braft::FileServiceImpl+braft::RaftServiceImpl+braft::CliServiceImpl] is serving on port=8107.
typesense_1     | I20210403 01:06:33.744860    82 server.cpp:1048] Check out http://1c913ea63cb1:8107 in web browser.
typesense_1     | I20210403 01:06:33.747742    82 log.cpp:674] Use crc32c as the checksum type of appending entries
typesense_1     | I20210403 01:06:33.747857    82 log.cpp:1158] log load_meta /data/state/log/log_meta first_log_index: 1 time: 56
typesense_1     | I20210403 01:06:33.748559    82 log.cpp:1098] load open segment, path: /data/state/log first_index: 1
typesense_1     | I20210403 01:06:33.748966    82 raft_meta.cpp:521] Loaded single stable meta, path /data/state/meta term 3 votedfor 0.0.0.0:8107:8108 time: 193
```

It also gives the stack trace in case of an exception.

#### ElasticSearch

Elasticsearch is arguably the most versatile search engine when it comes to logging. [See the documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/logging.html).

It is possible to set a precise rolling strategy.

A rolling log strategy offers permits to:

- Facilitate the administration of systems that generate a large number of logs.
- It automates swapping, compressing, deleting, and sending logs. It assists in keeping the logging system within the specified file system space limits.
- Defines an interval over which log analysis can be performed
- Gives an efficient way to identify log files that are no longer used so that an automated process can clean up and compress the log directory and run log analysis programs.

[See more information here about log rotation strategy](https://en.wikipedia.org/wiki/Log_rotation)

This is something important to have when systems generate a lot of logs or are under heavy load. It is also a great practice to feed a monitoring dashboard at certain times and, through this, to offer an easy way to analyze and understand what has happened in the system in a human-readable way.

Internally, Elasticsearch uses log4j2 to trace events and configure the log strategy. Logging levels can be configured per package, giving the precision needed to monitor a specific feature or time within the lifecycle of a search engine.

Log level can be: `debug`, `info`, `warn`, `error`, `fatal`, or `unknown`.

#### Elastic Entreprise Search

Elasticsearch's SaaS platform, like Algolia, offers a quite similar endpoint to browse the logs generated by the system.

However, it is possible to deactivate the storage of logs, modify the retention period but also to filter the logs with more details.

![image](https://www.elastic.co/guide/en/app-search/current/images/guides/log-settings-controls.png)
*Elastic App Search's Log Retention settings view.*

> It is also possible to do that from a Log Settings API endpoint.

> API Logs is aimed for trace requests and responses at engine level while Analytics Logs trace query, clicks and counts.

### IV. Explanation

#### Current Logging behaviour of MeiliSearch (0.20)

MeiliSearch uses `env_logger` to allow setting the output level of logs from the `RUST_LOG` environment variable. `env_logger` permits to set a specific log level per module if needed. [See more here](https://docs.rs/env_logger/0.8.3/env_logger/).

#### Logging behaviour for Milli (0.21)

We have decided to keep the use of `env_logger` for Milli/Transplant. However, we are going to make changes to make the logging more consistent and more versatile.

##### Log Levels

| Level | Description                                                                                                       |
|-------|-------------------------------------------------------------------------------------------------------------------|
| ERROR | Everything related to an error. A non blocking error has occured.                                                 |
| WARN  | Used to warn about an exceptional non-blocking event occurring.                                                   |
| INFO  | Default Log Level. It displays high level informations of events occuring in the search engine.                   |
| DEBUG | Used for debugging and development purposes.  More verbose than INFO.                                             |
| TRACE | Display everything happening at engine level. Not used at the moment.                                             |


##### Log Format

```
[2021-03-02T20:33:09Z INFO actix_web::middleware::logger] 172.17.0.1:57220 "POST /indexes/indexUID/documents HTTP/1.1" 202 14 "-" "PostmanRuntime/7.26.10" 0.023529
```

###### Mandatory log format part. E.g [TIME_FORMAT LOG_LEVEL MODULE] part.

- Time when the request was started to process (in rfc3339 format)
- Log levels are  `ERROR`, `WARN`, `INFO`, `DEBUG`, `TRACE`.
- The module part gives information about which module logs the entry.

###### HTTP Call

Transplant uses `actix_web::middleware::logger` to log informations on API endpoints that receive calls.

Given
```172.17.0.1:57220 "POST /indexes/indexUID/documents HTTP/1.1" 202 14 "-" "PostmanRuntime/7.26.10" 0.023529```

- Peer IP address (or IP address of reverse proxy if used)
- First line of request (Example: GET /test HTTP/1.1)
- Response status code
- Size of response in bytes, including HTTP headers
- User-Agent
- Time taken to serve the request, in seconds to 6 decimal places

> In DEBUG level the Search endpoint should log the request body and response body.

### V. Impact on Documentation

The documentation only mention the logging behavior for the `development` env on the `MEILI_ENV` part.

We should explain how to specify the log level using `RUST_LOG` environment variable and display the Log Levels table as information in a dedicated section.

### VI. Impact on SDKs
N/A

## 2. Technical Aspects

MeiliSearch is not consistent about logging methods. It have occurences of `println` and `eprintln` in the codebase and occurences of `error`, `warn`, `info`, `debug`, `trace` methods. It also calls `log::warn` or `log::error`.

Milli and Transplant should be careful by keeping a consistent way to log informations.

## 3. Future Possibilities

- Store logs on filesystem (give us future possibilites of rolling strategy). We will keep an eye on https://roadmap.meilisearch.com/c/81-specify-log-path, Github issues and, Slack Community messages. Keep in mind that it is possible to send logs to files using `syslog` or `systemd` journalctl.
- Develop API endpoint to search the logged events and configure logging policy within the system (SaaS feature in mind).

## 4. Planned Changes

### 0.21

#### Core
- Use a consistent method to log (relative to internal implementation)
- Log output should start with the mandatory log format part.
- If log level is set to DEBUG, /search endpoint should output parameters and response as a log output.

#### Documentation
- Add a dedicated logging section in the documentation.
- Add a link to this dedicated section on the `environment` [section](https://docs.meilisearch.com/reference/features/configuration.html#environment).
- We should add the default level for the production environment on the `environment` section, by default its `INFO`.
