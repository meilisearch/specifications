- Title: Dumps
- Start Date: 2022-01-12
- UpdatedAt: 2022-01-12

# Dumps

## 1. Functional Specification

### 1.1 Summary

A dump is a compressed file containing an export of a MeiliSearch instance. It contains all indexes, documents, settings, tasks, and api keys but in a raw unprocessed form. A dump isn't an exact copy of a database—it is closer to a blueprint that allows creating of an identical dataset.

### 1.2 Motivation

The dumps exist to upgrade from a previous version to a more recent version. It can also be a helpful tool for loading a production state on a staging server to make changes and test them before propagating them to production.

### 1.3 Explanations

#### 1.3.1 Summary Key Points

- A dump creation can be scheduled from the MeiliSearch API using the `POST - /dumps` endpoint.
- A dump creation status can be tracked using the `GET - /dumps/{uid}/status` endpoint.
- MeiliSearch can only create one dump at a time.
- While a dump is in creation, the task queue is paused and no write operations can occur on the database.
- By default, dumps are created in a folder named `dumps`, and can be found in the same directory as the MeiliSearch binary.
- The `dumps` directory can be customized using the `--dumps-dir` configuration option. If the dump directory does not already exist when the dump creation process is called, MeiliSearch will create it.
- If MeiliSearch is restarted after a dump creation, the dump's status will not appear on the `GET - /dumps/:uid/status` endpoint.
- A `.dump` file can be imported using the `--import-dump` command-line flag.
- The MeiliSearch server starts when the dump is fully imported and indexed.
- By default, importing a dump when a database already exists (a non-empty data.ms folder in the same directory as the MeiliSearch binary) will stop the process and throw an error.
- When using the command-line flag `--ignore-dump-if-db-exists=true`, MeiliSearch will use the existing database to start an instance instead of throwing an error. The dump will be ignored.
- By default, trying to import a dump that do not exists, will stop the process and throw an error.
- When using the command-line flag `--ignore-missing-dump`, MeiliSearch will continue its process and not throw an error.

---

#### 1.3.2 API Definition

##### **As a user, I want to create a dump**

##### Request Definition

`POST` - `/dumps`

##### Headers

```
"Authorization: Bearer :apiKey"
"Content-Type: application/json"
```

##### Body Payload
N/A

##### Response

`202 Accepted`

```json
{
    "uid": "20220112-151751438",
    "status": "in_progress",
    "startedAt": "2022-01-12T15:17:51.438881Z"
}
```

##### Errors

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.
- 🔴 Attempting to create a dump while a dump is already being created return an [dump_already_in_progress](0061-error-format-and-definitions.md#dump_already_in_progress) error.

---

##### **As a user, I want to check a dump status**

##### Request Definition

`GET` - `/dumps/:uid/status`

##### Headers

```
"Authorization: Bearer :masterKey"
```

##### Response

`200 Success`

```json
{
    "uid": "20220112-151751438",
    "status": "in_progress",
    "startedAt": "2022-01-12T15:17:51.438881Z"
}
```

##### Errors

- 🔴 Accessing this route without the `Authorization` header returns a [missing_authorization_header](0061-error-format-and-definitions.md#missing_authorization_header) error.
- 🔴 Accessing this route with a key that does not have permissions (i.e. other than the master-key) returns an [invalid_api_key](0061-error-format-and-definitions.md#invalid_api_key) error.
- 🔴 Attempting to access a dump details that does not exist returns a [dump_not_found](0061-error-format-and-definitions.md#dump_not_found) error.

---

#### 1.3.4 CLI Definition

##### 1.3.5 `--dumps-dir`

By default, MeiliSearch creates dumps in a directory called `dumps` at the root of your MeiliSearch.

The destination can be modified with the `--dumps-dir` flag. e.g. `--dumps-dir myDumps`

##### 1.3.6 `--import-dump`

Using the CLI flag `--import-dump`, MeiliSearch will replace the data.ms with the dump data and start the server using the provided dump. e.g. `--import-dump dumps/20220117-144855452.dump`.

If the `--import-dump` flag is specified when a database exists, an error occurs in the CLI.

```
Error: database already exists at ":pathToDataMs/data.ms", try to delete it or rename it
```

##### 1.3.7 `--ignore-dump-if-db-exists`

To avoid MeiliSearch to throw an error when finding that a database already exists, the following flag: `--ignore-dump-if-db-exists` can be used. When using this flag, MeiliSearch will use the existing database to start an instance instead of throwing an error. The dump will be ignored.

##### 1.3.8 `--ignore-missing-dump`

To avoid MeiliSearch to throw an error when there is no dump at the given path, the following flag: `--ignore-missing-dump` can be used. MeiliSearch will then continue its process and not import any dump.

If the `--ignore-missing-dump` flag is not specified and the file cannot be found, an error occurs in the CLI.

```
Error: dump doesn't exist at ":pathToDumps/:missingFile"
```

---

## 2. Technical Aspects
n/a

## 3. Future Possibilities
n/a