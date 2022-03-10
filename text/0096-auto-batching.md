- Title: Auto-Batching

# Auto-Batching

## 1. Summary

Meilisearch can automatically group consecutive asynchronous `documentAddition` or `documentPartial` tasks for the same index via an automatic batching mechanism.

The user can enable this auto-batching behavior through various command flag options.

## 2. Motivation

We have regularly collected user pain points pointing out the slow indexing over the last year. We explained several times to users to make batches containing a maximum of documents to be updated/added to compress the indexing time of specific data structures.

To make Meilisearch easier to use, we explored the idea of automatically creating these batches within Meilisearch before indexing users’ documents.

## 3. Functional  Specification

### 3.1. Explanations

All `tasks` are linked to a `batchUid` field. The `batchUid` field identifies that several identical, consecutive tasks have been grouped in the same batch.

Only consecutive `documentAddition` and `documentPartial` tasks for the same index can have the same `batchUid`. All `tasks` concerning other operations will also be part of a `batchUid` having only one task.

#### 3.1.1. Grouping tasks to a single batch

The scheduling program that groups tasks within a single batch is triggered when an asynchronous `task` currently processed reaches a terminal state as `succeeded` or `failed`.

In other words, when a scheduled `task` is picked from the FIFO task queue, the scheduler fetches and groups all consecutive `documentAddition` for a similar index in a batch until it encounters another task type or a similar task type but for a different index.

The more similar consecutive tasks the user sends in a row, the more likely the batching mechanism can group these tasks.

##### 3.1.1.1. Schema

![Auto-batching Process](https://user-images.githubusercontent.com/3692335/145787054-4cb07b5e-c80e-498a-8843-d0cc46329e9b.png)

##### 3.1.1.2. `batchUid` generation

The identifier chosen for the `task` `batchUid` field corresponds to the `uid` value of the first task grouped within a batch. The batch identifiers are therefore unique and consecutive.

#### 3.1.2. Impacts on `task` API resource

- The different tasks grouped in a batch are processed within the same transaction. If a task fails within a batch, the whole batch fails.
- A `batchUid` field is only added on fully-qualified `task` API objects. It corresponds to the `task` `uid` value of the first task grouped within a batch. `batchUid` values are therefore unique and consecutive.
- Tasks within the same batch share the same values for the `startedAt`, `finishedAt`, `duration` fields, and the same `error` object if an error occurs for a `task` during the batch processing.
- If a batch contains many `tasks`, the `task` `details` `indexedDocuments` is identical in all `tasks` belonging to the same processed `batch`.

### 3.2. Auto-batching mechanisms options

### 3.2.1. `--enable-auto-batching`

By default, the auto-batching feature is disabled.

The auto-batching feature can be activated by passing the command flag `--enable-auto-batching` to Meilisearch at launch.

### 3.2.2.  `--max-batch-size`

`--max-batch-size <NUM>` allows setting the maximum number `NUM` of tasks that can be processed together within a single batch.

If `0` is set it will be replaced by `1`, since such a value would prevent any task from ever being processed.

If not specified, this is unlimited.

### 3.2.3. `--max-documents-per-batch`

`--max-documents-per-batch <NUM>` allows setting a limit to the maximum number `NUM` of documents that can be indexed together within a single batch.

Since the batch must contain at least one update, this value can be exceeded.

If not specified, this is unlimited.

### 3.2.4. `--debounce-duration-sec`

`--debounce-duration-sec <SECS>` allow to wait at least `SECS` seconds before processing a scheduled batch.

Defaults to `0`secs (process immediately).

## 4. Technical Aspects
N/A

## 5. Future Possibilities

- Extends it for all consecutive payload types.
- Add a filter capability by `batchUid` on the `/tasks` endpoints.
- Schedule non-consecutive tasks.
- Do not fail the entire transaction if a document is not valid. We must find a way to log the documents that could not be indexed.