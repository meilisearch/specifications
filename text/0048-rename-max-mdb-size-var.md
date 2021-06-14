- Title: Rename MAX_MDB_SIZE  configuration variable
- Start Date: 2021-
- Specification PR: [#48](https://github.com/meilisearch/specifications/pull/48)
- MeiliSearch Tracking-issues: [transplant/#206](https://github.com/meilisearch/transplant/issues/206)


# Rename MAX_MDB_SIZE configuration variable

## 1. Functional Specification

### I. Summary

This specification is written to rename the `MAX_MDB_SIZE` configuration variable.

### II. Motivation

Since version 0.21 no longer has a main LMDB database for all indexes but one LMBD database per index, the concept of a `main` database no longer exists internally.


### III. Additional Materials
N/A

### IV. Explanation

- Rename `--max-mdb-size` to `--max-index-size`
- Rename `MAX_MDB_SIZE` to `MAX_INDEX_SIZE`
- Error handling at index creation should be iso with version 0.20.

When I test the current version of meilisearch in 0.21, the http call on `POST /indexes/:index_uid/documents` is successful and an empty index is created whereas 0.20 returns an error on the API side and the index is not created.

```
{
    "message": "Impossible to create index; heed error; MDB_MAP_FULL: Environment mapsize limit reached",
    "errorCode": "index_creation_failed",
    "errorType": "internal_error",
    "errorLink": "https://docs.meilisearch.com/errors#index_creation_failed"
}
```

### V. Impact on Documentation

- Update the [Max MDB Size documentation part](https://docs.meilisearch.com/reference/features/configuration.html#max-mdb-size).

### VI. Impact on SDKs
N/A

## 2. Technical Aspects
N/A

## 3. Future Possibilities
- Provide a way to set a custom size per index.
