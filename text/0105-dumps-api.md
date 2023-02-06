# Dumps API and CLI

## 1. Summary

A dump is a compressed file containing an export of a MeiliSearch instance. It contains all indexes, documents, settings, tasks, and API keys but in a raw unprocessed form. A dump isn't an exact copy of a database—it is closer to a blueprint that allows creating of an identical dataset.

## 2. Motivation

The dumps exist to upgrade a MeiliSearch instance from a previous version to a more recent version. It can also be a helpful tool for loading a production state on a staging server to make changes and test them before propagating them to production.

## 3. Functional Specification

### 3.1. Summary Key Points

- A dump creation can be scheduled from the MeiliSearch API using the `POST - /dumps` endpoint.
- A dump creation status can be tracked using the `GET - /tasks/:task_uid` endpoint.
- MeiliSearch can only create one dump at a time.
- By default, dumps are created in a folder named `dumps`, and can be found in the same directory as the MeiliSearch binary.
- The `dumps` directory can be customized using the `--dump-dir` configuration option. If the dump directory does not already exist when the dump creation process is called, MeiliSearch will create it.
- A `.dump` file can be imported using the `--import-dump` command-line flag.
- The MeiliSearch server starts when the dump is fully imported and indexed.
- By default, importing a dump when a database already exists (a non-empty data.ms folder in the same directory as the MeiliSearch binary) will stop the process and throw an error.
- When using the command-line flag `--ignore-dump-if-db-exists=true`, MeiliSearch will use the existing database to start an instance instead of throwing an error. The dump will be ignored.
- By default, trying to import a dump that does not exist, will stop the process and throw an error.
- When using the command-line flag `--ignore-missing-dump`, MeiliSearch will continue its process and not throw an error.
- When a dump is being imported, the http API is not available. Meilisearch can't receive read or write requests.
- `dumpCreation` task takes priority over enqueued `tasks`. This means that if a `dumpCreation` task is created, it will be directly processed when the current processing task finishes even if other tasks have been enqueued before.

---

### 3.2. Dumps API Definition

#### 3.2.1. POST `/dumps`

Create a dump

##### 3.2.1.1. Body Payload
N/A

##### 3.2.1.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

The uid of the generated dump can be found in the task details.

##### 3.2.1.3. Errors

- 🔴 If Meilisearch is secured, accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 If Meilisearch is secured, accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

---

### 3.3. CLI Definition

#### 3.3.1. `--dump-dir`

By default, MeiliSearch creates dumps in a directory called `dumps` at the root of your MeiliSearch.

The destination can be modified with the `--dump-dir` flag. e.g. `--dump-dir myDumps`

#### 3.3.2. `--import-dump`

Using the CLI flag `--import-dump`, MeiliSearch will replace the data.ms with the dump data and start the server using the provided dump. e.g. `--import-dump dumps/20220117-144855452.dump`.

If the `--import-dump` flag is specified when a database exists, an error occurs in the CLI.

```
Error: database already exists at ":pathToDataMs/data.ms", try to delete it or rename it
```

#### 3.3.3. `--ignore-dump-if-db-exists`

To avoid MeiliSearch to throw an error when finding that a database already exists, the following flag: `--ignore-dump-if-db-exists` can be used. When using this flag, MeiliSearch will use the existing database to start an instance instead of throwing an error. The dump will be ignored.

#### 3.3.4. `--ignore-missing-dump`

To avoid MeiliSearch to throw an error when there is no dump at the given path, the following flag: `--ignore-missing-dump` can be used. MeiliSearch will then continue its process and not import any dump.

If the `--ignore-missing-dump` flag is not specified and the file cannot be found, an error occurs in the CLI.

```
Error: dump doesn't exist at ":pathToDumps/:missingFile"
```

---

### 3.4. Dump version support

To handle dump and Meilisearch version compatibility, it is necessary to also versionize the dumps feature.

The following table describes which version of the dump correspond to which version of Meilisearch

| Dump version | Meilisearch version                  |
|--------------|--------------------------------------|
| `v1`         | `v0.20.0` and below                  |
| `v2`         | `v0.21.0` and `v0.21.1`              |
| `v3`         | From `v0.22.0` to `v0.24.0` included |
| `v4`         | From `v0.25.0` to `v0.27.2` included |
| `v5`         | From `v0.28.0` to `v0.29.1` included |
| `v6`         | `v0.30.0` and later                  |

All dumps created before and up to Meilisearch v1.0 can be imported into Meilisearch v1.X.Y.

## 4. Technical Aspects

### 4.1. Dump Creation

When a dump is being created, the task queue can receive other future operations to perform.

### 4.2. Importing a dump

When a dump is being imported, the http API is not available. Meilisearch can't receive read or write requests.

### 4.3. Dump Creation Task Priority

Dump creation tasks have priority over other task types. If a `dumpCreation` task is enqueued, it will be directly processed when the current processing task finishes even if other tasks have been enqueued before.

## 5. Future Possibilities

- Give information about the import of a dump within the tasks.
