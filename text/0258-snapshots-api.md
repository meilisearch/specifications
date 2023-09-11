# Snapshots API and CLI

## 1. Summary

A snapshot is a compressed file containing an export of a MeiliSearch instance. It contains all indexes, documents, settings, tasks, and API keys.

## 2. Motivation

The snapshots exist to start a MeiliSearch instance from a database as fast as possible. It can be a helpful tool for loading a production state on a staging server to make changes and test them before propagating them to production.

## 3. Functional Specification

### 3.1. Summary Key Points

- A snapshot creation can be scheduled from the MeiliSearch API using the `POST - /snapshots` endpoint.
- A snapshot creation status can be tracked using the `GET - /tasks/:task_uid` endpoint.
- MeiliSearch will autobatch all your snapshots requests into one.
- By default, snapshots are created in a folder named `snapshots`, and can be found in the same directory as the MeiliSearch binary.
- The `snapshots` directory can be customized using the `--snapshot-dir` configuration option. If the snapshot directory does not already exist when the snapshot creation process is called, MeiliSearch will create it.
- A `.snapshot` file can be imported using the `--import-snapshot` command-line flag.
- The MeiliSearch server starts when the snapshot is fully imported.
- By default, importing a snapshot when a database already exists (a non-empty `data.ms` folder in the same directory as the MeiliSearch binary) will stop the process and throw an error.
- When using the command-line flag `--ignore-snapshot-if-db-exists=true`, MeiliSearch will use the existing database to start an instance instead of throwing an error. The snapshot will be ignored.
- By default, trying to import a snapshot that does not exist, will stop the process and throw an error.
- When using the command-line flag `--ignore-missing-snapshot`, MeiliSearch will continue its process and not throw an error.
- When a snapshot is being imported, the http API is not available. Meilisearch can't receive read or write requests.
- `snapshotCreation` task takes priority over enqueued `tasks`. This means that if a `snapshotCreation` task is created, it will be directly processed when the current processing task finishes even if other tasks have been enqueued before.

---

### 3.2. Snapshots API Definition

#### 3.2.1. POST `/snapshot`

Create a snapshot

##### 3.2.1.1. Body Payload
N/A

##### 3.2.1.2. Response Definition

When the request is successful, Meilisearch returns the HTTP code `202 Accepted`. The response's content is the summarized representation of the received asynchronous task.

See [Summarized `task` Object for `202 Accepted`](0060-tasks-api.md#summarized-task-object-for-202-accepted).

The name of the generated snapshot is the database path (`data.ms` by default), joined by the `.snapshot` extension. By default, it's `data.ms.snapshot`.

##### 3.2.1.3. Errors

- 🔴 If Meilisearch is secured, accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 If Meilisearch is secured, accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.

---

### 3.3. CLI Definition

#### 3.3.1. `--snapshot-dir`

By default, MeiliSearch creates snapshots in a directory called `snapshots` at the root of your MeiliSearch.

The destination can be modified with the `--snapshot-dir` flag. e.g. `--snapshot-dir mySnapshots`

#### 3.3.2. `--import-snapshot`

Using the CLI flag `--import-snapshot`, MeiliSearch will replace the `data.ms` with the snapshot data and start the server using the provided snapshot. e.g. `--import-snapshot snapshot/data.ms.snapshot`.

If the `--import-snapshot` flag is specified when a database exists, an error occurs in the CLI.

```
Error: database already exists at ":pathToDataMs/data.ms", try to delete it or rename it
```

#### 3.3.3. `--ignore-snapshot-if-db-exists`

To avoid MeiliSearch to throw an error when finding that a database already exists, the following flag: `--ignore-snapshot-if-db-exists` can be used. When using this flag, MeiliSearch will use the existing database to start an instance instead of throwing an error. The snapshot will be ignored.

#### 3.3.4. `--ignore-missing-snapshot`

To avoid MeiliSearch to throw an error when there is no snapshot at the given path, the following flag: `--ignore-missing-snapshot` can be used. MeiliSearch will then continue its process and not import any snapshot.

If the `--ignore-missing-snapshot` flag is not specified and the file cannot be found, an error occurs in the CLI.

```
Error: snapshot doesn't exist at ":pathToSnapshots/:missingFile"
```

#### 3.3.5 Other snapshots related errors

When starting a new Meilisearch version, if Meilisearch tries to read an old `data.ms` but cannot read it, the following message should appear:

```
Error: Your database version (`:old_version`) is incompatible with your current engine (`:new_version`). To migrate data between Meilisearch versions, follow our guide on https://docs.meilisearch.com/learn/advanced/updating.html
```

## 4. Technical Aspects

### 4.1. Snapshot Creation

When a snapshot is being created, the task queue can receive other future operations to perform.

### 4.2. Importing a snapshot

When a snapshot is being imported, the http API is not available. Meilisearch can't receive read or write requests.

### 4.3. Snapshot Creation Task Priority

Snapshot creation tasks have priority over other task types. If a `snapshotCreation` task is enqueued, it will be directly processed when the current processing task finishes even if other tasks have been enqueued before.

## 5. Future Possibilities

- Give information about who created the task (is it scheduled or created on a user demand) in the details.
