- Title: Rename MAX_MDB_SIZE env var and --max-mdb-size option flag
- Start Date: 2021-04-14
- Specification PR: [#48](https://github.com/meilisearch/specifications/pull/48)
- MeiliSearch Tracking-issues: [transplant/#206](https://github.com/meilisearch/transplant/issues/206)


# Rename MAX_MDB_SIZE env var and --max-mdb-size option flag

## 1. Functional Specification

### I. Summary

This specification is written to rename the `MAX_MDB_SIZE` environnement variable and `--max-mdb-size` option flag to `MAX_INDEX_SIZE` and `--max-index-size`.

### II. Motivation

Since version 0.21 no longer has a main LMDB database for all indexes but one LMBD database per index, the concept of a `main` database no longer exists internally.


### III. Additional Materials
N/A

### IV. Explanation

- Rename `--max-mdb-size` to `--max-index-size`
- Rename `MAX_MDB_SIZE` to `MAX_INDEX_SIZE`

### V. Impact on Documentation

- Update the [Max MDB Size documentation part](https://docs.meilisearch.com/reference/features/configuration.html#max-mdb-size).

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities
- Provide a way to set a custom size per index.
