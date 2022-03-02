- Title: Instance options
- Start Date: 2022-03-01
- Specification PR:
- MeiliSearch Tracking-issues:

# Instance options

## 1. Summary

The instance options let the users configure Meilisearch when launching the search engine using
- environment variables
- command-line options

An example when setting an environment variable to change the folder path where the Meilisearch data are stored:

```bash
export MEILI_DB_PATH=./meilifiles
./meilisearch
```

Same behavior using the command-line option:

```bash
./meilisearch --db-path ./meilifiles
```

## 2. Motivation

When Meilisearch is launched, the default configuration may not meet the specific needs of users. Meilisearch exposes configurable options to allow users to fine-tune the behavior of the search engine.

## 3. Functional Specification

The users can configure Meilisearch when launching the search engine using
- **environment variables**. Ex: `--db-path`
- **command-line options or CLI options**. Ex: `MEILI_DB_PATH`

There are 2 categories of CLI (command-line interface) options:
- the ones that expect a value. Ex: `--db-path`.
- the ones that does expect any value, called also "flags". Ex: `--no-analytics`. Their implicit values are booleans.

### 3.1 Some specific behaviors

#### Priority between CLI options and environment variables

Command-line options take precedence over environment variables. If the same configuration option is specified both as a command-line option and as an environment variable, Meilisearch will use the command-line option and its respective value.

#### Flags: accepted values for the corresponding environment variable

The options that do not expect any value when using the command-line option accepts the following value when using the corresponding environment variable: `n`, `no`, `f`, `false`, `off`, and `0` as `false`. An absent environment variable will also be considered as `false`. Everything else is considered `true`.

Example with the snapshort creation:
- `export MEIL_SCHEDULE_SNAPSHOT=yes` means the snapshot creation is enabled.
- `export MEIL_SCHEDULE_SNAPSHOT=off` means the snapshot creation is disabled.
- No variable set means the snapshot creation is disabled.

### 3.2 Error behavior

1. Some configuration options must specify a value to be valid. Using such a command-line option or an environment variable without specifying a value will throw an error and interrupt the launch process.

Example:

❌ Wrong

```bash
./meilisearch --db-path

error: The argument '--db-path <DB_PATH>' requires a value but none was supplied
```

✅ Correct

```bash
./meilisearch --db-path ./meilifiles
```

2. Some command-line options take an implicit boolean as a value. In this case, the users should not set any value when using the option.

❌ Wrong

```bash
./meilisearch --schedule-snapshot yes

error: Found argument 'yes' which wasn't expected, or isn't valid in this context
```

✅ Correct

```bash
./meilisearch --schedule-snapshot
```

The expected behavior of each flag is described in the list above.

### 3.3 Exhaustive list of options

- [Database path](#database-path)
- [Environment](#environment)
- [HTTP address & port binding](#http-address-port-binding)
- [Master key](#master-key)
- [Disable analytics](#disable-analytics)
- [Dumps](#dumps-destination)
- [Dumps destination](#dumps-destination)
- [Import dump](#import-dump)
- [Ignore missing dump](#ignore-missing-dump)
- [Ignore dump if DB exists](#ignore-dump-if-db-exists)
- [Log level](#log-level)
- [Max index size](#max-index-size)
- [Max TASK_DB size](#max-task-db-size)
- [Payload limit size](#payload-limit-size)
- [Snapshots](#schedule-snapshot-creation)
- [Schedule snapshot creation](#schedule-snapshot-creation)
- [Snapshot destination](#snapshot-destination)
- [Snapshot interval](#snapshot-interval)
- [Import snapshot](#import-snapshot)
- [Ignore missing snapshot](#ignore-missing-snapshot)
- [Ignore snapshot if DB exists](#ignore-snapshot-if-db-exists)
- [Memory usage](#memory-usage)
- [Indexing jobs](#indexing-jobs)
- [SSL configuration](#ssl-authentication-path)
- [SSL authentication path](#ssl-authentication-path)
- [SSL certificates path](#ssl-certificates-path)
- [SSL key path](#ssl-key-path)
- [SSL OCSP path](#ssl-ocsp-path)
- [SSL require auth](#ssl-require-auth)
- [SSL resumption](#ssl-resumption)
- [SSL tickets](#ssl-tickets)

#### Database path

**Environment variable**: `MEILI_DB_PATH`
**CLI option**: `--db-path`
**Default value**: `"data.ms/"`
**Expected value**: a filepath

Designates the location where database files will be created and retrieved.

#### Environment

**Environment variable**: `MEILI_ENV`
**CLI option**: `--env`
**Default value**: `development`
**Expected value**: `production` or `development`

Configures the instance's environment. Value must be either `production` or `development`.

`production`:

- Setting a master key is **mandatory**
- The search preview interface is disabled

`development`:

- Setting a master key is **optional**
- Search preview is enabled

#### HTTP address & port binding

**Environment variable**: `MEILI_HTTP_ADDR`
**CLI option**: `--http-addr`
**Default value**: `"127.0.0.1:7700"`
**Expected value**: an HTTP address and port

Sets the HTTP address and port Meilisearch will use.

#### Master key

**Environment variable**: `MEILI_MASTER_KEY`
**CLI option**: `--master-key`
**Default value**: `None`
**Expected value**: an alphanumeric string

Sets the instance's master key, automatically protecting all routes except [`GET /health`](/reference/api/health.md). This means you will need an API key to access endpoints such as `POST /search` and `GET /documents`.

You must supply an alphanumeric string when using this option.

Providing a master key is mandatory when `--env` is set to `production`; if none is given, then Meilisearch will throw an error and refuse to launch.

If no master key is provided in a `development` environment, all routes will be unprotected and publicly accessible.

#### Disable analytics

**Environment variable**: `MEILI_NO_ANALYTICS`
**CLI option**: `--no-analytics`
**Default**: Enabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Deactivates Meilisearch's built-in telemetry collect when enabled.

#### Dumps destination

**Environment variable**: `MEILI_DUMPS_DIR`
**CLI option**: `--dumps-dir`
**Default value**: `dumps/`
**Expected value**: a filepath pointing to a valid directory

Sets the directory where Meilisearch will create dump files. If the directory does not exist when a dump is generated it will be created.

#### Import dump

**Environment variable**: N/A
**CLI option**: `--import-dump`
**Default value**: `none`
**Expected value**: a filepath pointing to a `.dump` file

Imports the dump file located at the specified path. Path must point to a `.dump` file.

Meilisearch will only launch once the dump data has been fully indexed.

More regarding dump behaviors in this [spec](https://github.com/meilisearch/specifications/blob/develop/text/0105-dumps-api.md).

#### Ignore missing dump

**Environment variable**: N/A
**CLI option**: `--ignore-missing-dump`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Prevents a Meilisearch instance from throwing an error when `--import-dump` does not point to a valid dump file.

This command will throw an error if `--import-dump` is not defined.

More information in this [section of the spec](https://github.com/meilisearch/specifications/blob/develop/text/0105-dumps-api.md#138---ignore-missing-dump).

#### Ignore dump if DB exists

**Environment variable**: N/A
**CLI option**: `--ignore-dump-if-db-exists`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Prevents a Meilisearch instance with an existing database from throwing an error when using `--import-dump`. Instead, the dump will be ignored and Meilisearch will launch using the existing database.

This command will throw an error if `--import-dump` is not defined.

More information in this [section of the spec](https://github.com/meilisearch/specifications/blob/develop/text/0105-dumps-api.md#137---ignore-dump-if-db-exists).

#### Log level

**Environment variable**: `MEILI_LOG_LEVEL`
**CLI option**: `--log-level`
**Default value**: `'INFO'`
**Expected value**: one of `ERROR`, `WARN`, `INFO`, `DEBUG`, OR `TRACE`

Defines how much detail should be present in Meilisearch's logs.

#### Max index size

**Environment variable**: `MEILI_MAX_INDEX_SIZE`
**CLI option**: `--max-index-size`
**Default value**: `107374182400` (100 GiB)
**Expected value**: an integer (`104857600`) or a human readable size (`100Mb`)

Sets the maximum size of the index. Value must be given in bytes or explicitly stating a base unit. For example, the default value can be written as `107374182400`, `'107.7Gb'`, or `'107374 Mb'`.

The `index` stores processed data and is different from the `task` database, which handles pending tasks.

#### Max TASK_DB size

**Environment variable**: `MEILI_MAX_TASK_DB_SIZE`
**CLI option**: `--max-task-db-size`
**Default value**: `107374182400` (100 GiB)
**Expected value**: an integer (`104857600`) or a human readable size (`100Mb`)

Sets the maximum size of the `task` database. Value must be given in bytes or explicitly stating a base unit. For example, the default value can be written as `107374182400`, `'107.7Gb'`, or `'107374 Mb'`.

The `task` database handles pending tasks. This is different from the `index` database, which only stores processed data.

#### Payload limit size

**Environment variable**: `MEILI_HTTP_PAYLOAD_SIZE_LIMIT`
**CLI option**: `--http-payload-size-limit`
**Default value**: `104857600` (~100MB)
**Expected value**: an integer (`104857600`) or a human readable size (`100Mb`)

Sets the maximum size of accepted payloads. Value must be given in bytes or explicitly stating a base unit. For example, the default value can be written as `107374182400`, `'107.7Gb'`, or `'107374 Mb'`.

#### Schedule snapshot creation

**Environment variable**: `MEILI_SCHEDULE_SNAPSHOT`
**CLI option**: `--schedule-snapshot`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Activates scheduled snapshots when enabled. Snapshots are disabled by default.

#### Snapshot destination

**Environment variable**: `MEILI_SNAPSHOT_DIR`
**CLI option**: `--snapshot-dir`
**Default value**: `snapshots/`
**Expected value**: a filepath pointing to a valid directory

Sets the directory where Meilisearch will store snapshots. If the directory does not exist when a snapshot is generated it will be created.

#### Snapshot interval

**Environment variable**: `MEILI_SNAPSHOT_INTERVAL_SEC`
**CLI option**: `--snapshot-interval-sec`
**Default value**: `86400` (1 day)
**Expected value**: an integer

Defines the interval between each snapshot. Value must be given in seconds.

#### Import snapshot

**Environment variable**: N/A
**CLI option**: `--import-snapshot`
**Default value**: `None`
**Expected value**: a filepath pointing to a snapshot file

Launches Meilisearch after importing a previously-generated snapshot at the given filepath.

This command will throw an error if:

- A database already exists
- No valid snapshot can be found in the specified path

This behavior can be modified with the `--ignore-snapshot-if-db-exists` and `--ignore-missing-snapshot` options, respectively.

#### Ignore missing snapshot

**Environment variable**: N/A
**CLI option**: `--ignore-missing-snapshot`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Prevents a Meilisearch instance from throwing an error when `--import-snapshot` does not point to a valid snapshot file.

This command will throw an error if `--import-snapshot` is not defined.

#### Ignore snapshot if DB exists

**Environment variable**: N/A
**CLI option**: `--ignore-snapshot-if-db-exists`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Prevents a Meilisearch instance with an existing database from throwing an error when using `--import-snapshot`. Instead, the snapshot will be ignored and Meilisearch will launch using the existing database.

This command will throw an error if `--import-snapshot` is not defined.

#### Memory usage

**Environment variable**: `MEILI_MAX_MEMORY`
**CLI option**: `--max-memory`
**Default value**: 2/3 of the available RAM of the machine
**Expected value**: an integer (`104857600`) or a human readable size (`100Mb`)

Set the maximum size of the RAM used by Meilisearch. By default, Meilisearch adapts its behavior to make the indexation use a maximum two thirds of available resources.

Value must be given in bytes or explicitly stating a base unit. For example, the default value can be written as `107374182400`, `'107.7Gb'`, or `'107374 Mb'`.

⚠️ WARNINGS
- We do not recommend setting the full RAM size of your machine. For example, when running Meilisearch on a machine with 4GB of RAM, do not set this options to `4Gb`
- This command-line option will not perfectly ensure the RAM usage, but will help you manage multiple Meilisearch engines on the same machine (for example, using Kubernetes). The core team cannot guarantee the exact usage of the RAM.
- If the number set is higher than the real available RAM in the machine, we cannot prevent Meilisearch from crashing.

#### Indexing jobs

**Environment variable**: `MEILI_INDEXING_JOBS`
**CLI option**: `--indexing-jobs`
**Default value**: half of the available core of the machine
**Expected value**: an integer

Sets the number of jobs available during the indexation.

By default, in machines with multi-core processors, the indexer avoids using more than half of the available processing units. For example, if your machine has twelve cores, the indexer will try to use six of them at most. This ensures Meilisearch is always ready to perform searches, even while you are updating an index.

Obviously, multi-threading is not possible in machines with only one processor core.

If the number set is higher than the real number of core available in the machine, Meilisearch will use the maximum number of available cores.

#### SSL authentication path

**Environment variable**: `MEILI_SSL_AUTH_PATH`
**CLI option**: `--ssl-auth-path`
**Default value**: `None`
**Expected value**: a filepath

Enables client authentication in the specified path.

#### SSL certificates path

**Environment variable**: `MEILI_SSL_CERT_PATH`
**CLI option**: `--ssl-cert-path`
**Default value**: `None`
**Expected value**: a filepath pointing to a valid SSL certificate

Sets the server's SSL certificates.

Value must be a path to PEM-formatted certificates. The first certificate should certify the KEYFILE supplied by `--ssl-key-path`. The last certificate should be a root CA.

#### SSL key path

**Environment variable**: `MEILI_SSL_KEY_PATH`
**CLI option**: `--ssl-key-path`
**Default value**: `None`
**Expected value**: a filepath pointing to a valid SSL keyfile

Sets the server's SSL keyfiles.

Value must be a path to an RSA private key or PKCS8-encoded private key, both in PEM format.

#### SSL OCSP path

**Environment variable**: `MEILI_SSL_OCSP_PATH`
**CLI option**: `--ssl-ocsp-path`
**Default value**: `None`
**Expected value**: a filepath pointing to a valid OCSP certificate

Sets the server's OCSP file. *Optional*

Reads DER-encoded OCSP response from OCSPFILE and staple to certificate.

#### SSL require auth

**Environment variable**: `MEILI_SSL_REQUIRE_AUTH`
**CLI option**: `--ssl-require-auth`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Makes SSL authentication mandatory.

Sends a fatal alert if the client does not complete client authentication.

#### SSL resumption

**Environment variable**: `MEILI_SSL_RESUMPTION`
**CLI option**: `--ssl-resumption`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Activates SSL session resumption.

#### SSL tickets

**Environment variable**: `MEILI_SSL_TICKETS`
**CLI option**: `--ssl-tickets`
**Default**: Disabled

⚠️ This command-line option does not take any values. Assigning a value will throw an error.

Activates SSL tickets.

## 4. Technical Aspects

N/A

## 5. Future Possibilities

- Redo the command-line to create a more interactive CLI
