- Title: Auto-Batching
- Start Date: 2021-10-06
- Specification PR: [#96](https://github.com/meilisearch/specifications/pull/96)
- Discovery Issue: [#69](https://github.com/meilisearch/product/issues/69)

# Auto-Batching

## 1. Functional Specification

### 1.1 Summary

MeiliSearch can automatically group consecutive asynchronous documentsAddition or documentsPartial tasks for the same index into a single batch via an automatic batching mechanism.

This auto-batching behavior is enabled by the user through various command flag options.

### 1.2 Motivation

We have been collecting user pain points regularly over the last year, pointing out the slow indexing. We have explained several times to users to make batches containing a maximum of documents to be updated/added to compress the indexing time of specific data structures.

To make MeiliSearch easier to use, we explored the idea of automatically creating these batches within MeiliSearch before indexing users’ documents.

### 1.3 Explanations

#### 1.3.1 Overall

All `tasks` are linked to a specific `batchUid` field. This `batchUid` field permits to identify that several identical, consecutive tasks have been grouped in the same batch.

Only consecutive `documentAddition` and `documentPartial` `tasks` for the same index can have a shared `batchUid` since the scheduler will be able to group them together. This means that all `tasks` concerning other operations will also be part of a `batchUid` having only one task. This could be extended in the future by enhancing the scheduler capabilities.

This is to avoid having multiple incompressible computation time by grouping them together to increase performance when applied to a maximum number of documents.

#### 1.3.2 Grouping tasks to a single batch

The scheduling program that groups tasks within a single batch is triggered when an asynchronous `task` currently processed reaches a terminal state as `succeeded` or `failed`.

In other words, when the next `task` should be picked from the FIFO task queue. The scheduler fetches and groups all consecutive `documentAddition` for a similar index in a batch until it encounters another task type or a similar task type but for a different index.

The more similar consecutive tasks the user sends in a row, the more likely the batching mechanism is able to group these tasks in a batch. It can be seen as an automatic back-pressure mechanism.

##### 1.3.2.1 Schema

![Auto-batching Process](https://user-images.githubusercontent.com/3692335/145787054-4cb07b5e-c80e-498a-8843-d0cc46329e9b.png)

##### 1.3.2.2 `batchUid` generation

The identifier chosen for the `task` batchUid` field corresponds to the `uid` value of the first task grouped within a batch. The batch identifiers are therefore unique and consecutive.

#### 1.3.3 Impacts on `task` API object format

- The different tasks grouped in a batch are processed within the same transaction. If a task fails within a batch, all the tasks fail or succeed.
- A `batchUid` field is added on fully-qualified `task` API objects. It corresponds to the `task` `uid` value of the first task grouped within a batch. `batchUid` values are therefore unique and consecutive.
- Tasks within the same batch share the same values for the `startedAt`, `finishedAt`, `duration` fields, and the same `error` object if an error occurs for a `task` during the batch processing.
- If a batch contains many `tasks`, the `task` `details` `indexedDocuments` is identical in all `tasks` belonging to the same processed `batch`.

#### 1.3.4 Activate auto-batching feature with `--enable-auto-batching`

The auto-batching feature is activated by passing the command flag `--enable-auto-batching` to MeiliSearch when it is launched.

#### 1.3.5 Auto-batching mechanisms options

##### 1.3.5.1  `--max-batch-size`

`--max-batch-size <NUM>` allows to set the maximum number `NUM` of tasks that can be processed together within a single batch.

If `0` is set it will replaced by `1`, since such a value would prevent any task from ever being processed.

If not specified, this is unlimited.

##### 1.3.5.2 `--max-documents-per-batch`

`--max-documents-per-batch <NUM>` allows to set a limit to the maximum number `NUM` of documents  that can be indexed together within a single batch.

Since the batch must contain at least one update, this value can be exceeded. If not specified, this is unlimited.

##### 1.3.5.3 `--debounce-duration-secs`

`--debounce-duration-secs <SECS>` allow to wait at least `SECS` seconds before processing a scheduled batch.

Defaults to `0`secs (process immediately).

## 2. Technical Aspects
n/a

## 3. Future Possibilities

- Extends it for all consecutive payload types.
- Add a filter capability by `batchUid` on the `/tasks` endpoints.
- Schedule non-consecutive tasks.
- Do not fail the entire transaction if a document is not valid. We must find a way to log the documents that could not be indexed.