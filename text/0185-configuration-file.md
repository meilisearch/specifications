# Configuration File

## 1. Summary

Meilisearch options can be defined in a `.toml` configuration file being loaded at launch.

## 2. Motivation

Teams can easily version, track, and share configuration files. It makes it easier to learn about the different options available and to define at a glance how Meilisearch is configured.

## 3. Functional Specification

### 3.1. Supported Formats

| Format    |
|-----------|
| `.toml`   |

### 3.2. Priority Order

`Configuration file < Env variables < Command-line options`

It means the options in the configuration file are overwritten by the environment variables set (if they exist) and that can be themself overwritten by the command-line options (if they exist).

### 3.3. Default Configuration File

By default, Meilisearch will try to read the `./config.toml` in the current working directory.

We provide a default configuration file as an example [here](https://github.com/meilisearch/meilisearch/blob/main/config.toml).

### 3.4. `--config-file-path`/`MEILI_CONFIG_FILE_PATH`

It is possible to override the default configuration file by setting the `MEILI_CONFIG_FILE_PATH` environment variable or by specifying the `--config-file-path` option.

### 3.5. Configuration File Content Definition

The current consistency rule is that all environment variables and launch options are configurable by a dedicated key in the configuration file.

Configuration keys are named following the `snake_case` convention from the name of a CLI option. That is, `--import-dump` must be named `import_dump` within the configuration file.

There is an exception rule. The `config_file_path` key is not accepted in the configuration file and stops Meilisearch from being launched if specified.

See [Instance Options](0119-instance-options.md) for more details.

### 3.6. Indication of the loaded configuration file

When Meilisearch is launched, the loaded configuration file is indicated by the `Config file path` key on stdout.

```bash
888b     d888          d8b 888 d8b                                            888
8888b   d8888          Y8P 888 Y8P                                            888
88888b.d88888              888                                                888
888Y88888P888  .d88b.  888 888 888 .d8888b   .d88b.   8888b.  888d888 .d8888b 88888b.
888 Y888P 888 d8P  Y8b 888 888 888 88K      d8P  Y8b     "88b 888P"  d88P"    888 "88b
888  Y8P  888 88888888 888 888 888 "Y8888b. 88888888 .d888888 888    888      888  888
888   "   888 Y8b.     888 888 888      X88 Y8b.     888  888 888    Y88b.    888  888
888       888  "Y8888  888 888 888  88888P'  "Y8888  "Y888888 888     "Y8888P 888  888

Config file path:       "./config.toml"
```

If Meilisearch can't find the default `config.toml` or if no other config file is specified with `MEILI_CONFIG_FILE_PATH` env var or `--config-file-path` options, `none` is indicated.

### 3.7. Errors

#### 3.7.1. Defining `config_file_path` in the configuration file

Defining `config_file_path` in the configuration file is not valid and Meilisearch returns an error.

```bash
Error: `config_file_path` is not supported in configuration file.
```

#### 3.7.2. Defining a non-existent config file.

Defining a configuration file other than the default file that does not exist returns an error when launching Meilisearch.

```bash
Error: unable to open or read the "XXX" configuration file: No such file or directory (os error 2).
```

#### 3.7.3. Defining a config file with a syntax error.

Defining a configuration file wrongly formatted or containing an invalid configuration key returns an error when launching Meilisearch.

```bash
Error: unknown field `XXX` at line Y column Z
```

## 4. Technical Details
N/A

## 5. Future Possibilities

- Better indicate a syntax error
- List all loaded configuration at launch on the stdout
- Introduce toml sections to group options under a common name. e.g. A dump section could contain a `schedule`, `dir`, `interval-sec` and `enable`. That could help navigate the toml.
- Introduce other formats